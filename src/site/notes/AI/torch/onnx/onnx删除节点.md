---
{"dg-publish":true,"permalink":"/AI/torch/onnx/onnx删除节点/","noteIcon":"3"}
---

#onnx 
```py
import onnx
from onnx import helper

def remove_node_and_connect(model, target_node_name):
    # 获取模型的图
    graph = model.graph

    # 查找目标节点、前驱和后继节点
    target_node = None
    predecessor_nodes = []
    successor_nodes = []
    
    # 遍历图中的所有节点，找到目标节点及其前后节点
    for node in graph.node:
        if node.name == target_node_name:
            target_node = node
            # 记录目标节点的输入（前驱节点的输出）
            input_name = node.input[0]
            # 记录目标节点的输出（后继节点的输入）
            output_name = node.output[0]
        else:
            if target_node_name in node.input:
                successor_nodes.append(node)
            if target_node_name in node.output:
                predecessor_nodes.append(node)

    if not target_node:
        print(f"节点 {target_node_name} 未找到")
        return model

    # 更新前后节点以跳过目标节点
    for successor in successor_nodes:
        for i, input_name in enumerate(successor.input):
            if input_name == target_node.output[0]:  # 检查是否为目标节点的输出
                successor.input[i] = target_node.input[0]  # 将前驱节点的输出连接到后继节点

    # 从图中移除目标节点
    graph.node.remove(target_node)

    return model

# 加载 ONNX 模型
model_path = "path/to/your_model.onnx"
model = onnx.load(model_path)

# 删除节点并连接前后节点
target_node_name = "name_of_target_node"  # 替换为实际节点名称
modified_model = remove_node_and_connect(model, target_node_name)

# 保存修改后的模型
onnx.save(modified_model, "path/to/modified_model.onnx")
print("模型已保存")

```