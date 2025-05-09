---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Cpp/visit/","noteIcon":"3"}
---

references:https://levelup.gitconnected.com/understanding-std-visit-in-c-a-type-safe-way-to-traverse-variant-objects-dbeff9b47003

```cpp
#include <iostream>
#include <variant>
#include <vector>

void func(int i) {
  std::cout << "Called func(int): " << i << std::endl;
}

void func(double d) {
  std::cout << "Called func(double): " << d << std::endl;
}

void func(const std::string& s) {
  std::cout << "Called func(string): " << s << std::endl;
}

int main() {
  std::vector<std::variant<int, double, std::string>> myVector = {1, 3.14, "Hello"};

  for (auto& element : myVector) {
    std::visit([](auto&& arg){ func(arg); }, element);
  }

  return 0;
}


```

1. auto && indicate compiler自动deduce arg type
2. 
> std::visit is a runtime polymorphic function that determines the type of the variant object at runtime and then calls the appropriate function overload.
> When we call std::visit, we pass it a callable object that defines a set of overloaded function objects for each type in the variant. At runtime, std::visit uses the type of the variant object to determine which function object to call, and then passes the variant object to that function object.