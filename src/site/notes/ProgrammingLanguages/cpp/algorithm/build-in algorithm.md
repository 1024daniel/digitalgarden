---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/cpp/algorithm/build-in algorithm/","noteIcon":"3"}
---

### 1. all_of, any_of, none_of

第三个参数是`unary predicate`
```cpp
// if all elements is '0', then result is "0"
#include<vector>

auto a = vector<int>(5,0)
all_of(a.cbegin(), a.cend(), [](int e) { return e == 0; })

auto b = vector<int>{0,0,1,0}
any_of(b.cbegin(), b.cend(), [](int e) { return e == 1; })

none_of(a.cbegin(), a.cend(), [](int e) { return e != 0; })

```


### 2. max max_element

```cpp
#include <iostream>
#include <algorithm>
using namespace std;
int main(void)
{
	int a[6] = {5, 3, 2, 6, 1, 4};
	int b = a[0];
	int c = a[1];
	cout<<max(b, c)<<" "<<min(b,c)<<endl; //输出为5 3
	cout<<max_element(a, a+6) - a<<endl;// 输出为3 下标
	cout<<*max_element(a, a+6)<<endl;//输出为 6 
	cout<<min_element(a, a+6) - a<<endl;// 输出为4 下标
	cout<<*min_element(a, a+6)<<endl;	 //输出为1 
	return 0; 
}



```
3. 