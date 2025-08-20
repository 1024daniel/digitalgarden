---
{"dg-publish":true,"permalink":"/AI/ascend/work/vllm/","noteIcon":"3"}
---




下面给你一套**不再 monkey-patch forward**、而是用 **direct_register_custom_op + fake_impl** 来“按输入长度阈值（>1000）动态走 FlashComm 分支”的**通用实现**。思路是：

- 把「要不要走 FlashComm（RS/AG）」这个**运行期判断**放进**自定义 OP**里（Dynamo 只看到一个稳定的 OP）。
    
- 在 **fake_impl** 里只返回**形状/ dtype 正确**的占位结果，保证编译期能顺利构图。
    
- 用一个**轻薄的 LinearMethod 包装器**替换 RowParallelLinear/ColumnParallelLinear 里的 quant_method.apply，在里面调用我们注册的自定义 OP。这样**不用动 vLLM 的 forward**，也不用 patch 你之前那两个文件。
    

  

> 下面代码按两类数据路径分别给出：**浮点 (FP)** 和 **W8A8**；同时覆盖 **Row (Reduce-Scatter)** 与 **Column (All-Gather)** 两种并行方向。

---

# **1) 自定义 OP 注册（含** 

# **fake_impl**

# **）**

  

保存为 ascend_flashcomm_ops.py（或任何你喜欢的模块名，确保能被 import 到）。

```python
# ascend_flashcomm_ops.py
from __future__ import annotations
import torch
import torch.nn.functional as F
import torch_npu
from vllm.utils import direct_register_custom_op
from vllm.distributed import (
    tensor_model_parallel_reduce_scatter,
    tensor_model_parallel_all_gather,
)
from typing import Optional


# -----------------------------
# 工具：推导通信域（示例实现，按你环境调整）
# -----------------------------
def _infer_comm_domain(tp_size: int) -> str:
    cur = torch.npu.current_device()
    # 这里的做法与你的 patch 一致；如需更严格可从 HCCL backend 查询
    return str(cur // tp_size)


# =========================================================
# RowParallel: Maybe FlashComm (FP)
#   - 若启用且 num_tokens > threshold → 走 _npu_matmul_reduce_scatter
#   - 否则：普通 matmul + reduce_scatter（形状对齐 RS 分支）
# 输出形状（两边对齐）：
#   out: ((num_tokens + pad) // tp_size, out_features)
# =========================================================
def _row_fp_impl(input_, weight, bias, *,
                 tp_rank:int, tp_size:int, reduce_results:bool,
                 threshold_tokens:int, enabled:bool,
                 out_dtype:torch.dtype) -> torch.Tensor:
    num_tokens = int(input_.size(0))
    out_features = int(weight.shape[0])
    commDomain = _infer_comm_domain(tp_size)

    def _rs_shape_len(n:int) -> int:
        pad = (tp_size - (n % tp_size)) % tp_size
        return (n + pad) // tp_size

    if enabled and (num_tokens > threshold_tokens) and reduce_results and (tp_size > 1):
        x = input_
        pad = (tp_size - (num_tokens % tp_size)) % tp_size
        if pad > 0:
            x = F.pad(x, (0, 0, 0, pad))
        out = torch.empty(_rs_shape_len(num_tokens), out_features,
                          dtype=out_dtype, device=x.device)
        # outdata_type: bf16=27, fp16=1（按需调整）
        torch_npu.atb._npu_matmul_reduce_scatter(
            x, weight, out, None, None,
            rank=tp_rank, rankSize=tp_size, commDomain=commDomain, outdata_type=27)
        # 注意：bias 只在 rank0 融合的话，这里可不加；外部与 vLLM 逻辑一致即可
        return out
    else:
        # 常规路径：保持与 RS 分支相同的输出形状（避免再编译）
        mm = torch.matmul(input_, weight.t())
        if bias is not None:
            mm = mm + bias
        if reduce_results and (tp_size > 1):
            pad = (tp_size - (num_tokens % tp_size)) % tp_size
            if pad > 0:
                mm = F.pad(mm, (0, 0, 0, pad))
            return tensor_model_parallel_reduce_scatter(mm, dim=0)
        else:
            # 无 RS 的情况：直返（单卡/无 reduce）
            return mm.to(out_dtype)

def _row_fp_fake(input_, weight, bias, *,
                 tp_rank:int, tp_size:int, reduce_results:bool,
                 threshold_tokens:int, enabled:bool,
                 out_dtype:torch.dtype) -> torch.Tensor:
    n = input_.shape[0]
    out_features = int(weight.shape[0])
    if reduce_results and (tp_size > 1):
        pad = (tp_size - (n % tp_size)) % tp_size
        chunk = (n + pad) // tp_size
        shape = (chunk, out_features)
    else:
        shape = (n, out_features)
    return torch.empty(shape, dtype=out_dtype, device=input_.device)

direct_register_custom_op(
    op_name="ascend_row_linear_maybe_flashcomm_fp",
    op_func=_row_fp_impl,
    fake_impl=_row_fp_fake,
    mutates_args=[],
    dispatch_key="PrivateUse1",
)


# =========================================================
# RowParallel: Maybe FlashComm (W8A8)
#   - 输入若非 int8：先量化（依赖你已有的 scale/offset）
#   - 走 _npu_matmul_reduce_scatter，带 deqScale
#   - 否则 fallback 与 FP 类似（但这里仍按 int8 输入处理）
# 需要的额外参数：
#   input_scale_reciprocal, input_offset, deq_scale（形状对齐你的 patch）
# =========================================================
def _row_w8a8_impl(input_, weight, bias, *,
                   tp_rank:int, tp_size:int, reduce_results:bool,
                   threshold_tokens:int, enabled:bool,
                   input_scale_reciprocal: torch.Tensor,
                   input_offset: torch.Tensor,
                   deq_scale: torch.Tensor,
                   out_dtype: torch.dtype) -> torch.Tensor:
    num_tokens = int(input_.size(0))
    out_features = int(weight.shape[1])  # 注意 W8A8 权重布局
    commDomain = _infer_comm_domain(tp_size)

    # 量化
    if input_.dtype != torch.int8:
        # 你项目里已有的 per-tensor 量化函数
        # 这里直接用线性量化的公式演示：round(x*scale) + offset
        xq = torch.clamp((input_ * input_scale_reciprocal).round() + input_offset, -128, 127).to(torch.int8)
    else:
        xq = input_

    def _rs_shape_len(n:int) -> int:
        pad = (tp_size - (n % tp_size)) % tp_size
        return (n + pad) // tp_size

    if enabled and (num_tokens > threshold_tokens) and reduce_results and (tp_size > 1):
        pad = (tp_size - (num_tokens % tp_size)) % tp_size
        if pad > 0:
            xq = F.pad(xq, (0, 0, 0, pad))
        out = torch.empty(_rs_shape_len(num_tokens), out_features,
                          dtype=out_dtype, device=xq.device)
        bias_i32 = torch.zeros(out_features, dtype=torch.int).unsqueeze(0)
        torch_npu.atb._npu_matmul_reduce_scatter(
            xq, weight, out, bias_i32, deqScale=deq_scale.unsqueeze(0),
            rank=tp_rank, rankSize=tp_size, commDomain=commDomain, outdata_type=27)
        return out
    else:
        # fallback：简单 matmul + RS（注意：真实 W8A8 的 fallback 通常需要 int8 GEMM + deq，
        # 这里为了通用示例，直接转回 fp 做 mm；你也可接入项目里的 int8 kernel）
        x = input_.to(out_dtype)
        mm = torch.matmul(x, weight.transpose(0, 1).to(out_dtype))
        if bias is not None and bias.dtype == out_dtype:
            mm = mm + bias
        if reduce_results and (tp_size > 1):
            pad = (tp_size - (num_tokens % tp_size)) % tp_size
            if pad > 0:
                mm = F.pad(mm, (0, 0, 0, pad))
            return tensor_model_parallel_reduce_scatter(mm, dim=0)
        else:
            return mm

def _row_w8a8_fake(input_, weight, bias, *,
                   tp_rank:int, tp_size:int, reduce_results:bool,
                   threshold_tokens:int, enabled:bool,
                   input_scale_reciprocal: torch.Tensor,
                   input_offset: torch.Tensor,
                   deq_scale: torch.Tensor,
                   out_dtype: torch.dtype) -> torch.Tensor:
    n = input_.shape[0]
    out_features = int(weight.shape[1])
    if reduce_results and (tp_size > 1):
        pad = (tp_size - (n % tp_size)) % tp_size
        chunk = (n + pad) // tp_size
        shape = (chunk, out_features)
    else:
        shape = (n, out_features)
    return torch.empty(shape, dtype=out_dtype, device=input_.device)

direct_register_custom_op(
    op_name="ascend_row_linear_maybe_flashcomm_w8a8",
    op_func=_row_w8a8_impl,
    fake_impl=_row_w8a8_fake,
    mutates_args=[],
    dispatch_key="PrivateUse1",
)


# =========================================================
# ColumnParallel: Maybe FlashComm (FP)
#   - 若启用且 num_tokens > threshold → _npu_all_gather_matmul
#   - 否则：gather 输入再常规 matmul
# 输出形状：两边都对齐为 (num_tokens_total_after_AG, out_features)：
#   = (num_tokens * tp_size - pad)；我们内部处理 pad 去除
# =========================================================
def _col_fp_impl(input_, weight, bias, *,
                 tp_rank:int, tp_size:int, gather_output:bool,
                 threshold_tokens:int, enabled:bool,
                 out_dtype:torch.dtype) -> torch.Tensor:
    num_tokens_local = int(input_.size(0))
    out_features = int(weight.shape[1])
    commDomain = _infer_comm_domain(tp_size)

    if enabled and (num_tokens_local * tp_size > threshold_tokens) and (tp_size > 1):
        # FlashComm AG GEMM（ATB 内部做 all_gather + matmul）
        total_tokens = num_tokens_local * tp_size
        out = torch.empty(total_tokens, out_features, dtype=out_dtype, device=input_.device)
        bias0 = torch.zeros(out_features, dtype=torch.int).unsqueeze(0)  # 无效，占位
        torch_npu.atb._npu_all_gather_matmul(
            input_, weight, out, bias0,
            deqScale=None, rank=tp_rank, rankSize=tp_size, commDomain=commDomain, outdata_type=27)
        # 去 pad：这里按行数正好整除，无显式 pad；若你在别处做了 pad，这里裁掉
        if bias is not None and bias.dtype == out_dtype:
            out = out + bias.reshape(1, -1)
        return out if gather_output else torch.chunk(out, tp_size, dim=0)[tp_rank]
    else:
        # fallback：先 gather 输入，再常规 matmul
        x = input_
        if tp_size > 1:
            x = tensor_model_parallel_all_gather(x, dim=0)
        mm = torch.matmul(x, weight)
        if bias is not None and bias.dtype == out_dtype:
            mm = mm + bias.reshape(1, -1)
        return mm if gather_output else torch.chunk(mm, tp_size, dim=0)[tp_rank]

def _col_fp_fake(input_, weight, bias, *,
                 tp_rank:int, tp_size:int, gather_output:bool,
                 threshold_tokens:int, enabled:bool,
                 out_dtype:torch.dtype) -> torch.Tensor:
    n = int(input_.shape[0])
    out_features = int(weight.shape[1])
    total = n * tp_size
    shape = (total, out_features) if gather_output else (n, out_features)
    return torch.empty(shape, dtype=out_dtype, device=input_.device)

direct_register_custom_op(
    op_name="ascend_col_linear_maybe_flashcomm_fp",
    op_func=_col_fp_impl,
    fake_impl=_col_fp_fake,
    mutates_args=[],
    dispatch_key="PrivateUse1",
)


# =========================================================
# ColumnParallel: Maybe FlashComm (W8A8)
# =========================================================
def _col_w8a8_impl(input_, weight, bias, *,
                   tp_rank:int, tp_size:int, gather_output:bool,
                   threshold_tokens:int, enabled:bool,
                   input_scale_reciprocal: torch.Tensor,
                   input_offset: torch.Tensor,
                   deq_scale: torch.Tensor,
                   out_dtype: torch.dtype) -> torch.Tensor:
    num_tokens_local = int(input_.size(0))
    out_features = int(weight.shape[1])
    commDomain = _infer_comm_domain(tp_size)

    if input_.dtype != torch.int8:
        xq = torch.clamp((input_ * input_scale_reciprocal).round() + input_offset, -128, 127).to(torch.int8)
    else:
        xq = input_

    if enabled and (num_tokens_local * tp_size > threshold_tokens) and (tp_size > 1):
        total_tokens = num_tokens_local * tp_size
        out = torch.empty(total_tokens, out_features, dtype=out_dtype, device=input_.device)
        bias_i32 = torch.zeros(out_features, dtype=torch.int).unsqueeze(0)
        torch_npu.atb._npu_all_gather_matmul(
            xq, weight, out, bias_i32,
            deqScale=deq_scale.unsqueeze(0), rank=tp_rank, rankSize=tp_size,
            commDomain=commDomain, outdata_type=27)
        if bias is not None and bias.dtype == out_dtype:
            out = out + bias.reshape(1, -1)
        return out if gather_output else torch.chunk(out, tp_size, dim=0)[tp_rank]
    else:
        # fallback（演示用：回到 fp 做 mm）；实际可接入你的 int8 fallback
        x = input_.to(out_dtype)
        if tp_size > 1:
            x = tensor_model_parallel_all_gather(x, dim=0)
        mm = torch.matmul(x, weight.to(out_dtype))
        if bias is not None and bias.dtype == out_dtype:
            mm = mm + bias.reshape(1, -1)
        return mm if gather_output else torch.chunk(mm, tp_size, dim=0)[tp_rank]

def _col_w8a8_fake(input_, weight, bias, *,
                   tp_rank:int, tp_size:int, gather_output:bool,
                   threshold_tokens:int, enabled:bool,
                   input_scale_reciprocal: torch.Tensor,
                   input_offset: torch.Tensor,
                   deq_scale: torch.Tensor,
                   out_dtype: torch.dtype) -> torch.Tensor:
    n = int(input_.shape[0])
    out_features = int(weight.shape[1])
    total = n * tp_size
    shape = (total, out_features) if gather_output else (n, out_features)
    return torch.empty(shape, dtype=out_dtype, device=input_.device)

direct_register_custom_op(
    op_name="ascend_col_linear_maybe_flashcomm_w8a8",
    op_func=_col_w8a8_impl,
    fake_impl=_col_w8a8_fake,
    mutates_args=[],
    dispatch_key="PrivateUse1",
)
```

**要点**

- fake_impl 与 op_func **签名完全一致**，并返回**相同形状/ dtype**的张量（对同一次调用），防止 Dynamo 反复再编译。
    
- 阈值逻辑（>1000）放在 OP 内部，**trace 阶段不会被折叠**。
    
- Row（RS）与 Column（AG）两个方向都各自有 FP 与 W8A8 两个 OP。
    
- 量化/反量化在演示里做了**最小可用**实现；你可以替换成你项目里的 quant_per_tensor 与实际的 int8 kernel。
    

---

# **2) 轻量 LinearMethod 包装器（不改 forward）**

  

保存为 ascend_flashcomm_linear_method.py。用它来替换各线性层的 quant_method，从而在 apply(...) 里**调用上面的自定义 OP**。

```python
# ascend_flashcomm_linear_method.py
from __future__ import annotations
import torch
from vllm.model_executor.layers.linear import (
    UnquantizedLinearMethod,
)
from typing import Optional


class UnquantizedLinearMethodWithFlashAuto(UnquantizedLinearMethod):
    """
    浮点版本：在 apply 内部调用 ascend_*_maybe_flashcomm_fp 自定义 OP
    """
    def apply(self, module, x: torch.Tensor, bias: Optional[torch.Tensor] = None,
              **kwargs):
        # 从 module 获取必要参数
        tp_rank = module.tp_rank
        tp_size = module.tp_size
        out_dtype = module.params_dtype
        reduce_results = getattr(module, "reduce_results", True)
        gather_output = getattr(module, "gather_output", False)

        # 动态门限与开关（可改为 env 或 vllm_config 注入）
        threshold_tokens = kwargs.pop("flash_threshold_tokens", 1000)
        enabled = kwargs.pop("flash_enabled", True)

        # Row / Column 的判断：看 weight 维度 & module 类型
        # RowParallelLinear: weight.shape = [out_per_tp, in_dim]
        # ColumnParallelLinear: weight.shape = [in_dim, out_per_tp]
        is_row = module.weight.shape[1] == x.shape[-1]  # 粗略判断

        if is_row:
            # RowParallel → Reduce-Scatter 形状
            return torch.ops.vllm.ascend_row_linear_maybe_flashcomm_fp(
                x, module.weight, bias,
                tp_rank=tp_rank, tp_size=tp_size,
                reduce_results=reduce_results,
                threshold_tokens=threshold_tokens, enabled=enabled,
                out_dtype=out_dtype,
            )
        else:
            # ColumnParallel → All-Gather 形状
            return torch.ops.vllm.ascend_col_linear_maybe_flashcomm_fp(
                x, module.weight, bias,
                tp_rank=tp_rank, tp_size=tp_size,
                gather_output=gather_output,
                threshold_tokens=threshold_tokens, enabled=enabled,
                out_dtype=out_dtype,
            )


class W8A8LinearMethodWithFlashAuto(UnquantizedLinearMethod):
    """
    W8A8 版本：这里继承 UnquantizedLinearMethod 只是为了少改动 vLLM 外层；
    实际里你可继承 AscendW8A8LinearMethod，并根据你现有字段拿到量化参数
    """
    def apply(self, module, x: torch.Tensor, bias: Optional[torch.Tensor] = None,
              **kwargs):
        tp_rank = module.tp_rank
        tp_size = module.tp_size
        out_dtype = module.params_dtype
        reduce_results = getattr(module, "reduce_results", True)
        gather_output = getattr(module, "gather_output", False)

        threshold_tokens = kwargs.pop("flash_threshold_tokens", 1000)
        enabled = kwargs.pop("flash_enabled", True)

        # 需要从 module 中取到量化参数（命名与项目保持一致）
        input_scale_reciprocal = getattr(module, "aclnn_input_scale_reciprocal")
        input_offset = getattr(module, "aclnn_input_offset")
        deq_scale = getattr(module, "deq_scale")

        is_row = module.weight.shape[1] == x.shape[-1]

        if is_row:
            return torch.ops.vllm.ascend_row_linear_maybe_flashcomm_w8a8(
                x, module.weight, bias,
                tp_rank=tp_rank, tp_size=tp_size,
                reduce_results=reduce_results,
                threshold_tokens=threshold_tokens, enabled=enabled,
                input_scale_reciprocal=input_scale_reciprocal,
                input_offset=input_offset,
                deq_scale=deq_scale,
                out_dtype=out_dtype,
            )
        else:
            return torch.ops.vllm.ascend_col_linear_maybe_flashcomm_w8a8(
                x, module.weight, bias,
                tp_rank=tp_rank, tp_size=tp_size,
                gather_output=gather_output,
                threshold_tokens=threshold_tokens, enabled=enabled,
                input_scale_reciprocal=input_scale_reciprocal,
                input_offset=input_offset,
                deq_scale=deq_scale,
                out_dtype=out_dtype,
            )
```

> 如果你项目里已经有 AscendW8A8LinearMethod，把上面这个类改成继承它更合适：这样能直接访问你实现里已有的量化字段，并与权重布局严格对齐。

---

# **3) 将 Linear 的** 

# **quant_method**

#  **替换为“自动决策”版本**

  

可在**模型构造后**统一替换。**无需修改** RowParallelLinear.forward / ColumnParallelLinear.forward：

```python
# hook_quant_method.py
from __future__ import annotations
import torch
from vllm.model_executor.layers.linear import RowParallelLinear, ColumnParallelLinear
from vllm.model_executor.layers.linear import UnquantizedLinearMethod
from ascend_flashcomm_linear_method import (
    UnquantizedLinearMethodWithFlashAuto,
    W8A8LinearMethodWithFlashAuto,
)

def enable_flashcomm_auto_on_model(model: torch.nn.Module,
                                   use_w8a8: bool | None = None,
                                   threshold_tokens: int = 1000,
                                   enabled: bool = True) -> None:
    """
    遍历模型，把线性层的 quant_method 换成自动 FlashComm 版本。
    threshold_tokens / enabled 可作为默认 kwargs 传进 apply（按需扩展）。
    """
    for m in model.modules():
        if isinstance(m, (RowParallelLinear, ColumnParallelLinear)):
            qm = getattr(m, "quant_method", None)
            if qm is None:
                continue
            # 根据实际类型选择替换器（这里用启发式或外部参数 use_w8a8）
            new_qm = None
            if use_w8a8 is True:
                new_qm = W8A8LinearMethodWithFlashAuto()
            elif use_w8a8 is False or isinstance(qm, UnquantizedLinearMethod):
                new_qm = UnquantizedLinearMethodWithFlashAuto()
            else:
                # 你的项目里若能识别 W8A8 方法类名，则在此分支处理
                new_qm = UnquantizedLinearMethodWithFlashAuto()

            m.quant_method = new_qm  # 替换

            # 可选：把默认门限写进模块，apply 时作为 kwargs（也可通过外部 sampling/meta 传）
            m.flash_threshold_tokens = threshold_tokens
            m.flash_enabled = enabled
```

> 上面留了 use_w8a8 开关：

- > 如果你的模块上能可靠判断是否 W8A8（例如 isinstance(qm, AscendW8A8LinearMethod)），就用这个条件来选择 W8A8LinearMethodWithFlashAuto。
    
- > 否则先用 UnquantizedLinearMethodWithFlashAuto 跑通 FP，W8A8 再按你项目类名/字段对接。
    

---

# **4) 在你的** 

# **qwen3.py**

#  **里对接（最小改动）**

  

因为「是否走 FlashComm」已经完全封装在**自定义 OP**里了，CustomQwen3Model / CustomQwen3DecoderLayer 里的那些 flashcomm_v1_enabled / matmul_rs_enabled / ag_matmal_enabled / pad_size 参数**可以保留但不再强依赖**（不会破坏兼容性）。你只需要在**模型构建完**之后调用一次替换函数：

```python
# 在你初始化完模型后（例如 CustomQwen3ForCausalLM.__init__ 末尾）：
from hook_quant_method import enable_flashcomm_auto_on_model
# FP 先跑通：use_w8a8=False；若要 W8A8，传 True 并保证量化字段可用
enable_flashcomm_auto_on_model(self.model, use_w8a8=False, threshold_tokens=1000, enabled=True)
```

这样：

- **prefill 阶段**当 num_tokens（或 num_tokens_local * tp_size 对于列并行）超过 1000，就会**自动走 FlashComm**（RS 或 AG）；
    
- **compile/trace 阶段**只会看到我们的**自定义 OP**，其 fake_impl 会产出**稳定的形状**，不会触发真正的 NPU 通信；
    
- **运行期**在 op_func 里根据实际长度**动态切换**分支，不需要禁用 compile，也不需要 monkey-patch forward。
    

---

## **额外提醒 / 常见坑**

1. **形状一致性**
    
    两条分支（FlashComm 与 fallback）**必须返回一致的外部可见形状**（我已在示例中统一为 RS/AG 的输出形状），否则极易引发再编译。
    
2. **bias 的处理**
    
    你的原始 patch 里有“只在 rank0 融合 bias”之类策略。上面示例为了通用性做了最小化处理：
    
    - Row/RS 分支：不在 NPU 内部加 bias（或按你策略添加），保持与 vLLM 外部一致即可。
        
    - Column/AG 分支：AG 后在 Python 里加 bias。
        
        如果你有严格策略，直接把 bias 的融合移动到 op_func 内即可，但保证两边一致。
        
    
3. **W8A8 的量化参数来源**
    
    我在示例里通过 module.aclnn_input_scale_reciprocal/input_offset/deq_scale 取参数，名称和你 patch 一致；如你项目字段不同，替换即可。真实场景建议把类改为继承你项目的 AscendW8A8LinearMethod，直接读取内部状态。
    
4. **commDomain 的获取**
    
    我保留了你 rank // tp_size 的写法；如果你们有 HCCL 正规接口，建议改为 backend 提供的通信域 ID。
    
5. **门限与开关来源**
    
    目前示例用固定阈值 1000 和布尔开关 enabled。可以：
    
    - 做成环境变量；
        
    - 挂到 vllm_config；
        
    - 或按 batch/场景动态传入（在 quant_method.apply 的 kwargs 里透传）。
        
    

---

如果你需要，我可以再给一版把 **W8A8LinearMethodWithFlashAuto 改为继承你们的 AscendW8A8LinearMethod** 的实现（对接你们现有的量化字段/权重布局），或者把 **Row/Col 的 OP 合并**成一个带 mode="row"/"col" 的通用 OP。