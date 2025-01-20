---
{"dg-publish":true,"permalink":"/AI/torch/onnx/replace node/","noteIcon":"3"}
---

```py
import onnx
from onnx import helper

def replace_node_op_type(model, old_op_type, new_op_type):
    # 遍历模型的节点
    for node in model.graph.node:
        if node.op_type == old_op_type:
            # 将 old_op_type 替换为 new_op_type
            node.op_type = new_op_type
    return model

# 加载 ONNX 模型
onnx_model_path = "model.onnx"  # 替换为你的模型路径
model = onnx.load(onnx_model_path)

# 将 'MMCVRoiAlignRotated' 替换为 'RoiAlignRotated'
model = replace_node_op_type(model, "MMCVRoiAlignRotated", "RoiAlignRotated")

# 保存修改后的模型
output_model_path = "modified_model.onnx"  # 替换为保存模型的路径
onnx.save(model, output_model_path)

print("替换完成并保存新的模型。")

```