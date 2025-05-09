---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Python/modules/argparse/","noteIcon":"3"}
---

#argparse

https://blog.csdn.net/tsinghuahui/article/details/89279152

```py hl:5
import argparse

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument("--test_action", action='store_true')

    args = parser.parse_args()
    action_val = args.test_action

    print(action_val)

```

对于非选项，通过环境变量的方式开启某种特性的时候可以通过:
```py
import os
enable_a = int(os.getenv("ENABLE_A", 0))
if enable_a:
	pass

```