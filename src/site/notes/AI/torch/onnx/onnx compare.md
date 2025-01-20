---
{"dg-publish":true,"permalink":"/AI/torch/onnx/onnx compare/","noteIcon":"3"}
---

#onnx
https://blog.csdn.net/tailor997/article/details/129104881

```sh
import onnx
from onnx_diff.diff import OnnxDiff


model_a = onnx.load("yolov5s_v6_0.onnx")
model_b = onnx.load("yolov5s_rknn.onnx")
diff = OnnxDiff(model_a, model_b,)
results = diff.summary(output=True)
print(results)


```