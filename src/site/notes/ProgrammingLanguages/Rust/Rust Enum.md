---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Rust/Rust Enum/","noteIcon":"3"}
---

rust enum支持将多个类型内置到rust enum variants
string,numeric types,structs甚至enum，同时enum类似struct支持impl block来定义method
与struct不同的事，impl block的方法可以为enum variants共用
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // method body would be defined here
    }
}

let m = Message::Write(String::from("hello"));
m.call();


```


### Option Enum


```cardlink
url: https://doc.rust-lang.org/std/option/enum.Option.html
title: "Option in std::option - Rust"
description: "The `Option` type. See the module level documentation for more."
host: doc.rust-lang.org
favicon: ../../static.files/favicon-044be391.svg
```

Option Enum是一个特殊的enum，主要是用于类似其他语言的null语法，但是相对于null有rust安全的特性
```rust
enum Option<T> {
    None,
    Some(T),
}
let some_number = Some(5);
let some_char = Some('e');

let absent_number: Option<i32> = None;

    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y; // will failed
    
```
上面可以看到`Option<T>`和T是不同的type，所以不能直接进行互操作，在操作前你需要主动先获取`Option<T>`的value在和其他的T进行操作

-  针对可能存在null的value可以设置类型为`Option<T>`, 在获取对应的值之前我们可以自信安全得认为程序中不会出现空指针之类的错误
-  对于要访问可能是null的数值，我们需要手动去获取对应的值，否则编译器会报错，这从语法上让我们对于相关操作进行检验


```rust
let x = Some("air");
assert_eq!(x.unwrap(), "air");

```