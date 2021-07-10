# 0 RustStudy

**学习目的：**

- 学习rust语言以进一步学习操作系统

**考核标准：**

- 每周学习记录情况 (25%)
- 在issues上的提问和回答问题情况，Pull Request提交情况 (25%)
- step 0 要求的编程代码的完成情况 (25%)
- step 2 rcore tutorial的通过要求完成情况 (25%)

**运行环境：**

- wsl2+vscode

**参考资料：**

- 《Rust 程序设计语言》
- 《Rust 编程之道》

<img src="https://kaisery.github.io/trpl-zh-cn/img/ferris/does_not_compile.svg" alt="img" style="zoom:10%;" /><img src="https://kaisery.github.io/trpl-zh-cn/img/ferris/panics.svg" alt="img" style="zoom:10%;" /><img src="https://kaisery.github.io/trpl-zh-cn/img/ferris/unsafe.svg" alt="img" style="zoom:10%;" /><img src="https://kaisery.github.io/trpl-zh-cn/img/ferris/not_desired_behavior.svg" alt="img" style="zoom:10%;" />

# Day01 Rust与Cargo的安装与使用

> 2021.07.10
>
> 《Rust 程序设计语言》第一、二章

## 1.1 安装rust

1. 在wsl中安装`rustup`

   ```shell
   curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
   ```

2. 查看安装是否成功

   ```shell
   rustc --version
   ```

   ![](https://raw.githubusercontent.com/SiO-2/PictureBed/main/img/20210710233422.png)

3. 在vscode中安装插件`rust-analyzer` 和 `Rust Syntax`

## 1.2 cargo的使用

> Cargo 是 Rust 的构建系统和包管理器。大多数 Rustacean 们使用 Cargo 来管理他们的 Rust 项目，因为它可以为你处理很多任务，比如构建代码、下载依赖库并编译这些库。——《Rust 程序设计语言》

1. 刷新当前的shell环境

   ```shell
   source $HOME/.cargo/env
   ```

2. 使用Cargo创建项目

   ```shell
   cargo new hello
   ```

3. Cargo项目的目录结构与文件

   ![](https://raw.githubusercontent.com/SiO-2/PictureBed/main/img/20210710233825.png)

   - `Cargo.toml`

     ```toml
     [package]
     name = "hello"
     version = "0.1.0"
     edition = "2018"
     
     [dependencies]
     ```

     - `[package]`，是一个片段（section）标题，表明下面的语句用来配置一个包
     - `[dependencies]`，是罗列项目依赖的片段的开始，在 Rust 中，代码包被称为 crates

   - `src`

     - 用于存放源文件
     - 项目根目录只存放 README、license 信息、配置文件和其他跟代码无关的文件

4. Cargo的基本指令

   - `cargo build`：为了开发，需要经常快速重新构建项目
   - `cargo run`
   - `cargo check`：编写代码时持续的进行检查
   - `cargo release`：为用户构建最终程序，它们不会经常重新构建，并且希望程序运行得越快越好，但构建时间相对`cargo build`而言比较长

## 1.3 通过项目了解Rust

通过编写一个猜数字的程序进一步了解`rust`语言编程

### 1.3.1 项目创建

1. 首先创建一个新的项目

   ```shell
   cargo new guessing_game
   ```

2. 修改`Cargo.toml`文件引入`rand`依赖

   ```toml
   [dependencies]
   
   rand = "0.8.3"
   ```

   - `[dependencies]` 片段告诉 Cargo 本项目依赖了哪些外部 crate 及其版本
   - 采用语义化版本`0.8.3` 来指定 `rand` crate，`0.8.3`事实上是`^0.8.3`的简写

3. 通过Cargo.lock文件确保构建可重现

   - 当第一次构建项目时，Cargo 计算出所有符合要求的依赖版本并写入 *Cargo.lock* 文件

   - 当将来构建项目时，Cargo 会发现 *Cargo.lock* 已存在并使用其中指定的版本，而不是再次计算所有的版本

   - 这意味着项目会持续使用 `0.8.3` ，直到显式升级

     - 可以使用`create update`升级，但不能升级到`0.9.x`的版本

     - 想要升级到`0.9.x`的版本必须要更新`Cargo.coml`文件

       ```toml
       [dependencies]
       
       rand = "0.9.0"
       ```

### 1.3.2 编程实现

```rust
use rand::Rng;
use std::io;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);
        match guess.cmp(&secret_number) {
            std::cmp::Ordering::Less => println!("Too small!"),
            std::cmp::Ordering::Greater => println!("Too big!"),
            std::cmp::Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

1. `use rand::Rng;` `use std::io;`

   - 使用 `use` 语句显式地将需要使用的类型引入作用域

2. `fn main(){}` 程序入口

3. `let secret_number = rand::thread_rng().gen_range(1..101);`

   - 定义了一个不可变的变量并通过随机数发生器赋值

4. `let mut guess = String::new();`

   - 定义了一个可变的字符串变量`guess`

5. 读入字符串变量并处理expect

   ```rust
   io::stdin()
   	.read_line(&mut guess)
   	.expect("Failed to read line");
   ```

   - 注意不能将`&mut guess`写为`&guess`
   - `read_line()`返回`io:Result`类型的值，用于`expect`的处理

6. 将输入的字符串转化为数字

   ```rust
   let guess: u32 = match guess.trim().parse() {
       Ok(num) => num,
       Err(_) => continue,
   };
   ```

   - 虽然已经定义了一个`guess`变量，但Rust允许用一个新值来 **隐藏**`guess` 之前的值,常用在需要转换值类型之类的场景
   - 将 `guess` 绑定到 `guess.trim().parse()` 表达式上
   - `: u32` 表示指定变量类型为无符号的32位整型
   - 通过`match`来匹配转化成功与失败时的下一步操作
     - 疑问：这里为什么需要写成`Ok(num)`与`Err(_)`，传入的参数有什么必要性?

7. `println!("You guessed: {}", guess);`

   - 打印输入，采用占位符`{}`

8. 通过`match`来匹配不同输入下程序运行的结果

   ```rust
   match guess.cmp(&secret_number) {
       std::cmp::Ordering::Less => println!("Too small!"),
       std::cmp::Ordering::Greater => println!("Too big!"),
       std::cmp::Ordering::Equal => {
           println!("You win!");
           break;
       }
   }
   ```

## 1.4 总结

今天安装了`rust`，配置了相对应的环境，了解使用了`cargo`来管理构建项目，并通过编写一个简易的`rust`程序大致了解了`rust`的基本语法与使用习惯，为进一步的学习做好准备。

