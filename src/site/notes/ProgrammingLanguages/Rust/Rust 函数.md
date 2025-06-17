---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Rust/Rust 函数/","noteIcon":"3"}
---

Rust中函数和变量通常使用snake case命名法来命名，也就是all letters are lowercase and using underscores to separate words

Rust函数相当于其他语言如cpp一个重要的区别是Rust is expression-based language，对于statement和expression需要进行区分理解

- **Statements** are instructions that perform some action and do not return a value.
- **Expressions** evaluate to a resultant value. Let’s look at some examples.

cpp代码可以通过`x=y=10;`来同时将x和y进行赋值，但是Rust将y=10这个视作Statement，并不会返回值
还有一个值得关注的点是在Rust中，一个expression如何结尾加上semicolon的话会转变成statement。
比如如下是一个代码示例，其中函数plus_one函数体里面`x+1`没有分号，是一个expression，返回值，这个和plus_one的函数签名能对的上，如果在`x+1`后面加上semicolon的话会导致编译器报错
```rust
fn main() {
    let x = plus_one(5);
    println!("The value of x is: {x}");
}
fn plus_one(x: i32) -> i32 {
    x + 1
}

```