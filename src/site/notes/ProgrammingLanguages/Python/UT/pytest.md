---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Python/UT/pytest/","noteIcon":"3"}
---

> **单元测试只测试你写的逻辑，所有外部依赖（IO、网络、数据库、时间、随机、其他模块）都建议 mock。**


capture
```py
# test_example.py
def test_printing():
    print("Hello from test")
    assert True

```
`pytest`默认会捕获print内容，只会输出assert之类的pytest之类的内容，防止函数内部的print内容对于test的内容干扰



monkeypatch为pytest的内置fixture，不需要显式进行import，只需要在待测试的函数内参数传入就可以
```python
import torch

def select_device():
    if hasattr(torch, "npu") and torch.npu.is_available():
        return torch.device("npu")
    return torch.device("cpu")
    
    
def test_select_device_npu(monkeypatch):
    monkeypatch.setattr("torch.npu.is_available", lambda: True)
    device = select_device()
    assert str(device) == "npu"

def test_select_device_cpu(monkeypatch):
    monkeypatch.setattr("torch.npu.is_available", lambda: False)
    device = select_device()
    assert str(device) == "cpu"


```


### fixture

```python
@pytest.fixture(scope="function")  # 默认，每个测试函数都会重新调用
@pytest.fixture(scope="module")    # 模块级共享一次
@pytest.fixture(scope="session")   # 整个测试会话共享

```

pytest 会在运行前：

- 扫描所有 @pytest.fixture 定义的函数（这些会注册为 fixture）
    
- 解析每个测试函数的参数名
    
- 如果某个参数名正好是 fixture，就会：
    
    - 自动调用 fixture
        
    - 把返回值注入到对应参数位置


### patch

> [!NOTE] 注意
> > **Patch 的路径应该是你在测试的模块中导入该对象的“使用位置”**，而不是它在源码中“被定义的位置”

```python
@patch("module.path.A")  # mock_A
@patch("module.path.B")  # mock_B
def test_something(mock_B, mock_A):  # 注意：参数顺序反了！
    ...

```


### MagicMock
```python
class A:
	property_in_class: int = 0
	def __init__(
		property_in_class: int = 0,
		property_in_instance: int = 0
	):
		self.property_in_class = property_in_class
		self.property_in_instance = property_in_instance
 

```
spec指定类名会通过dir只会获取到类属性和函数，实例属性没法获取到，想要获取实例属性，需要先实例化之后spec指定实例名

```python

dummy_A = MagicMock(spec=A)
dummy_A.property_in_class = 1        ### fine
dummy_A.property_in_instance =1      ### not ok

```

### @pytest.mark.parametrize
测试多组参数
```python
@pytest.mark.parametrize("a, b, expected", [
    (1, 2, 3),
    (2, 3, 5),
])
def test_add(a, b, expected):
    assert a + b == expected

```

### side_effect
在 unittest.mock 里，side_effect 是一个非常灵活的机制，赋值给一个 Mock 对象的属性后：

- 如果 side_effect 是一个**函数**，当 Mock 被调用时，调用会“委托”给这个函数执行；
    
- 换句话说，Mock 本身不执行默认的返回逻辑，而是执行 side_effect 指定的函数体；
    
- 这个函数的参数，就是调用 Mock 时传入的参数；
    
- 返回值是 side_effect 函数的返回值。

```cardlink
url: https://github.com/vllm-project/vllm-ascend/pull/2037
title: "add ut for device allocator/camem and mutistream/layers by 1024daniel · Pull Request #2037 · vllm-project/vllm-ascend"
description: "What this PR does / why we need it?test device allocator/camem and mutistream/layers contains resource allocation and stream opsDoes this PR introduce any user-facing change?N/AHow was this pat..."
host: github.com
favicon: https://github.githubassets.com/favicons/favicon.svg
image: https://opengraph.githubassets.com/454a18dbbb94c695f8e59625140427fef1ff9b55988c5eb4de5e9bb889cb811b/vllm-project/vllm-ascend/pull/2037
```

对于side_effect的用法可以举一个简单的例子:
这里我写了一个ut，最后提交的时候显示ut代码存在两行未执行(ut代码必须得100%执行)
![Pasted image 20250728224643.png](/img/user/ProgrammingLanguages/Python/UT/attachments/Pasted%20image%2020250728224643.png)
![Pasted image 20250728224629.png](/img/user/ProgrammingLanguages/Python/UT/attachments/Pasted%20image%2020250728224629.png)
![Pasted image 20250728224950.png](/img/user/ProgrammingLanguages/Python/UT/attachments/Pasted%20image%2020250728224950.png)
我这里只是简单想要给两个dump func当做参数传递，来完成执行init_module,其中init_module是进行patch了的，也就是说最后实际dump func只是为了满足函数签名要求，并不需要执行函数体，我们本意也是不需要执行，最后导致这两个函数体miss
通过以下操作，将init_module的mock的side_effect设置为一个局部函数，局部函数内进行调用我们miss的函数，从而可以完成全部调用
![Pasted image 20250728225143.png](/img/user/ProgrammingLanguages/Python/UT/attachments/Pasted%20image%2020250728225143.png)