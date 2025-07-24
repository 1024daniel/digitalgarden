---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Rust/Rust Struct/","noteIcon":"3"}
---

### Field Init Shorthand
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

// 
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
```

### Struct Update Syntax
未显式指定的field使用指定实例的值，注意Struct Update Synatx使用的是move，旧实例的指定值会invalid
```rust

//
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}

```

### Tuple Struct
tuple struct对于结构内的filed不需要进行命名，下面的Color和Point计算内部都是三个i32，但是属于不同的类型，一个接受Color的函数不能传入Point
对于Tuple Struct，由于属性没有名字，需要通过index来访问指定的field
和Tuple不一样的是如果需要进行内容展开的话需要指定Tuple Struct的类型来进行展开
```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let color = Color(1, 2, 3);
    let origin = Point(4, 5, 6);
    println!("{}",origin.0);
    let Color(r,g,b) = color;
    println!("{}",r)
}

```



Struct默认没有实现Debug trait，无法直接使用println!来打印struct的值
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {rect1:?}");
    println!("rect1 is {rect1:#?}");
}


```

dbg!宏可以打印当前表达式所在的文件和line number
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}

```

### Method in struct
struct的成员函数可以都放在一个impl block中或者是多个impl block中
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}

```

### Associated Functions
关联函数通常是用作构造struct实例，也就是构造函数的概念

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
let sq = Rectangle::square(3);

```