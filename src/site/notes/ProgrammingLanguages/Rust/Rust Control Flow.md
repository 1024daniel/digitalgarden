---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Rust/Rust Control Flow/","noteIcon":"3"}
---

#rust
### 1. if expression
```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}

```
Rust的if else等expression像python一样不需要小括号， 但是需要大括号而不是冒号，且if接的expression的evaluate的值必须是bool类型，不会像cpp那种非0的值都会被当做true这种
因为if是一个expression，所以可以使用if expression赋值给变量
```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {number}");
}

```

### 2. Loop expression
[Fetching Data#xkfi](https://stackoverflow.com/questions/73262372/return-value-from-loop-expression-with-break)

[Fetching Data#mips](https://stackoverflow.com/questions/73262372/return-value-from-loop-expression-with-break)

```rust
fn main() {
    loop {
        println!("again!");
    }
}

```

loop 是一个expression，对于下面的代码你可能会产生疑问，**按照之前的理解在expression结尾加上semicolon之后是一个statement，不会返回值，但是break结尾有semicolon之后还是会返回值**
其实这个可以这么理解， break是一个特殊的expression，和return类似，他会跳出当前的域来返回值
```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}");
}

```

下面一个可以展示出正常let就是一个statement, {}就是一个expression，默认最后会返回()
![Pasted image 20250620152033.png](/img/user/ProgrammingLanguages/Rust/attachments/Pasted%20image%2020250620152033.png)

对于嵌套的loop expression，我们可以显示得指定我们需要break那个loop
```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}

```

### 3. while expression


```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index += 1;
    }
}

```
### 4. for expression
```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
}

```

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{number}!");
    }
    println!("LIFTOFF!!!");
}

```

### 5. if let and let else

if let是match statement的语法糖(Rust1.65前就支持)，在只关注一个pattern，忽略其他pattern的时候可以简化代码，使得代码逻辑更加清晰, 注意下面的例子，else的return是提前返回
```rust
fn describe_state_quarter(coin: Coin) -> Option<String> {
    let state = if let Coin::Quarter(state) = coin {
        state         // return to state after let
    } else {
        return None;  // return the function
    };

    if state.existed_in(1900) {
        Some(format!("{state:?} is pretty old, for America!"))
    } else {
        Some(format!("{state:?} is relatively new."))
    }
}


```

let else是Rust 1.65(2022)开始支持，更加简洁

```rust
fn describe_state_quarter(coin: Coin) -> Option<String> {
    let Coin::Quarter(state) = coin else {
        return None;
    };

    if state.existed_in(1900) {
        Some(format!("{state:?} is pretty old, for America!"))
    } else {
        Some(format!("{state:?} is relatively new."))
    }
}


```


let else能替代 if let的场景: ## **模式匹配 + 失败时立即 return、break、panic 等**

对于以下场景还是得使用if let
 **❶ 多分支逻辑：**
```rust
if let Some(x) = maybe {
    // ...
} else if let Some(y) = other {
    // ...
} else {
    // fallback
}

```

 **❷ 非绑定语句：**

```rust
if let Some(x) = maybe {
    println!("x is {x}");
}

```

 **❸ 条件嵌套表达式中的匹配：**

```rust
let result = if let Some(x) = opt { x + 1 } else { 0 };

```
