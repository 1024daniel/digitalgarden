---
{"dg-publish":true,"permalink":"/AI/torch/tensor/contiguous/","noteIcon":"3"}
---

#contiguous

在 PyTorch 中，一个张量（tensor）的内存布局（memory layout）可以是连续的（contiguous）或非连续的（non-contiguous）。这主要影响张量在内存中的存储方式以及某些操作的性能。

### 连续内存布局（Contiguous）

- 当一个张量是连续的，这意味着它的所有元素在内存中是顺序存储的，就像一个一维数组一样。
- 许多 PyTorch 操作都假设输入张量是连续的，因为这样可以提高性能。如果一个张量不是连续的，这些操作可能会先复制张量，使其连续，然后再执行操作，这会增加额外的计算和内存开销。
- 可以通过调用 `.contiguous()` 方法来确保张量是连续的。

### 非连续内存布局（Non-contiguous）

- 非连续的张量意味着它的元素在内存中不是顺序存储的。这种情况通常在对张量进行切片、索引、转置等操作后发生。
- 非连续的张量可能会导致某些操作的性能下降，因为 PyTorch 需要额外处理非连续的内存布局。
- 非连续的张量可以通过 `.view()`, `.expand()`, `.index_select()` 等操作创建。

### 区别和影响

1. **性能**：连续的张量通常在性能上更优，因为它们在内存中是顺序存储的，这使得数据加载和处理更加高效。
    
2. **操作兼容性**：某些 PyTorch 操作要求张量必须是连续的。如果张量不是连续的，这些操作可能会自动调用 `.contiguous()` 方法，这会增加额外的内存和计算开销。
    
3. **内存使用**：非连续的张量可能会占用更多的内存，因为它们可能包含指向原始数据的指针，而不是数据的实际副本。
    
4. **代码清晰度**：在某些情况下，非连续的张量可以减少内存使用，但可能会使代码更难理解和维护，因为需要额外注意内存布局。



以下一个代码示例中y为通过转置x获得，其实可以从后面的内存地址可以看待y和x的内存地址是一样的，都是共享同一块内存，但是y视图(view)不一样了, 读取维度数据和x不一样，也就是不是顺序读取的, z是通过调用contiguous来重新申请了一块内存，内存是连续的
对于数据内存排布可以通过stride函数来获取，`stride`是一个整数列表，其中每个整数表示在对应维度上移动到相邻元素需要跳跃的内存单位数。对于转置操作，`transpose()`函数会交换两个指定的维度，但不会改变张量的物理内存排布
```py
import torch
x = torch.randn(2, 3)  # 创建一个形状为 (2, 3) 的张量
print(x.is_contiguous())
y = x.t()  # 转置张量
print(y.is_contiguous())  # 通常返回 False
z = y.contiguous()
print(z.is_contiguous())

print(x.data_ptr())
print(y.data_ptr())
print(z.data_ptr())

print(x.stride())
print(y.stride())
print(z.stride())
```