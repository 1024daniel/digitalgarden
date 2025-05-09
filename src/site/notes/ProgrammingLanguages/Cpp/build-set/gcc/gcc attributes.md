---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Cpp/build-set/gcc/gcc attributes/","noteIcon":"3"}
---

https://stackoverflow.com/questions/11770451/what-is-the-meaning-of-attribute-packed-aligned4

#gcc #attributes #cling
### 1. alignment

```cpp
// 注意cling一行一行解析，这里对于结构体定义的大括号需要在同一行
struct sSampleStruct1 {
     char Data1;
     //3-Bytes Added here.
     int Data2;
     unsigned short Data3;
     char Data4;
     //1-byte Added here.
};

struct sSampleStruct2 {
     char Data1;
     int Data2;
     unsigned short Data3;
     char Data4;

}__attribute__((packed, aligned(1)));

cout<<sizeof(sSampleStruct1)<<endl; // output: 12
cout<<sizeof(sSampleStruct2)<<endl; // output: 8
```




