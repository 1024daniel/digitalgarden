---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/cpp/cast between string and number/","noteIcon":"3"}
---

#cast #strinstream
https://blog.csdn.net/MOU_IT/article/details/89060249
### 1. from integer to string

#### 1.1 二进制转字符串
```run-cpp

bitset<10> bit("010101");
string str = bit.to_string();
cout << str << endl;
```
#### 1.2 十进制转字符串
```cpp
int a = 25
string s = to_string(a)

```

### 2. 进制间转换

#### 2.1 10进制和16进制转换


```cpp
#include <sstream>
using namespace std;

// 16进制转10进制
 
int x;
stringstream ss;
ss << std::hex << "1A";  //std::oct（八进制）、std::dec（十进制）
ss >> x;
x

string out = "1A";
x = stoi(out, nullptr, 16);
x


// 10进制转16进制
x = 26 ;
out = "";
ss << std::hex <<x;
ss.str()
ss >> out ;
transform(out.begin(), out.end(), out.begin(), ::toupper);
out

```

#### 2.2 2进制和10进制转换

```cpp
#include<bitset>
using namespace std;

// 10进制转2进制
int a = 4
bitset<8> bit(a)
bit.to_string()

```