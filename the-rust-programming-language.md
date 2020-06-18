# Rust程序设计语言

## 入门指南

### Hello, World!

rustfmt进行格式化

使用四个空格缩进，`{`与函数声明同一行

带有符号`!`，就是调用宏而不是函数



### Hello, Cargo!

Cargo是Rust的构建系统和包管理器
```shell
cargo new {name} # 创建新项目
cargo build (--release)
cargo run # 只检查是否可以编译
```


包含toml配置文件和src目录（[main.rs](http://main.rs)），并初始化git仓库

Cargo.lock：实际依赖的版本



## 常见编程概念

### 变量和可变性

变量（let）默认不可变（immutable），加上`mut`则为可变

常量（const）

let可以Shadow变量，新的变量和老的类型可以不同



### 数据类型

每个值都属于某个数据类型，Scalar或Compound

静态类型，存在多种类型可能性时必须要添加类型注解



Scalar

- 整型：i/u 8/16/32/64/128/size，size依赖于架构，允许类型后缀，和`_`作为分隔符。溢出debug下会panic，release下会wrap，如果确实需要wrap，标准库显式提供了功能。默认i32
- 浮点型：f32和f64，默认f64
- 布尔类型：bool，取值true / false
- 字符类型：char，四字节，代表Unicode标量值



Compound

- tuple：长度固定，支持destructuring，点号.访问
- array：元素类型相同，固定长度，栈上分配，越界访问在运行时会panic
```rust
let tup = (500, 6.4, 1);
let (x, y, z) = tup;
let first = tup.0;
let a: [i32; 5] = [1, 2, 3, 4, 5];
let a = [3; 5]; // 长度5，每个值为3
```



### 函数如何工作

函数定义顺序不重要

函数签名必须声明参数类型

形参（parameter），实参（argument）



语句（Statement）执行操作但不返回值

表达式（Expression）计算产生值，没有分号

`{}`可以是表达式，如果内部语句不返回值则是空元组`()`

->声明返回类型

返回值是最后一个表达式的值



### 控制流

条件必须是bool值（不会自动转换）

if是表达式，可以赋值（每个分支的类型要相同）

loop相当于while (true)

break可以加上值，作为loop的值

for .. in …



## 认识所有权

### 什么是所有权

- 每个值都有一个称为所有者的变量
- 值有且只有一个所有者
- 当所有者变量离开作用域，值会被丢弃

（RAII）离开作用域时，Rust自动调用drop



移动

```rust
let s1 = String::from("x");
let s2 = s1;
// s1会失效，再使用会报错
```



克隆

自由定义复制函数

实现了Copy trait的，会执行复制，包括Scalar和元组

如果自身或者任何部分实现了Drop trait则不允许使用Copy trait

函数调用也是实现了移动

```rust
fn main() {
    let s = String::from("x");
    take(s);
    // s不再有效
}
fn take(s: String) {
    println!("{}", s);
}
```



返回值会转移所有权

```rust
fn main() {
    let s1 = give();
    let s2 = String::from("y");
    let s3 = give_and_back(s2);
    // s2失效，s3有效
}

fn give() -> String {
    let s = String::from("x");
    x // 转移所有权给调用方
}

fn give_and_back(s: String) -> String {
    s
}
```



### 引用与借用

引用：使用值但不获取所有权

借用（borrowing）：以引用作为函数参数

```rust
fn calculate_length(s: &String) -> usize {
    s.len()
    // s不会被释放
} 
```



引用默认和变量一样也是不可变，除非加上`mut`

任意时间，要么只能有一个可变引用，要么只能有多个不可变引用

引用的作用域时从声明开始，持续到最有一次使用

```rust
let mut s = String::from("hello");
let r1 = &s;
let r2 = &s;
println!("{} {}", r1, r2); // 多个不可变引用，允许

let r3 = &mut s; // 若r1，r2最后使用在此之前，允许
println!("{}", r3);
```



引用总是有效，不会悬垂（Dangling Reference）

```rust
fn dangle() -> &String {
    let s = String::from("x");
    &s // 报错，生命周期，返回String可以，转移所有权
}
```



### Slices

部分引用
```rust
let s = String::from("abcde");
let slice = &s[0..2];
let slice = &s[..2];
let slice = &s[3..];
let slice = &s[..]
// 可以省略默认0..len
```

```rust
fn main() {
    let mut s = String::from("xyz");
    let word = find(&s); // word是&str
    s.clear(); // 报错，word是不可变引用，仍生效，clear需要获取可变引用
    println!("{}", word);
}
```

String literal就是slice，不可变引用

String在传参时，可以自动转为&str



## 使用结构体组织相关联的数据

### 定义并实例化结构体

不允许只将某个字段标记为可变
```rust
let mut u = User {
    email: String::from("@"),
    name: String::from("n"),
};
// 整个实例都是可变
```



同名变量初始化可以简写

```rust
fn new_user(email: String, name String) -> User {
    User {
        email,
        name,
    }
}
```



结构体更新，从已有实例构建新实例

```rust
let u2 = User {
    email: String::from("e"),
    ..u1
};
// 未显式设置的字段从已有实例获取
```



元组结构体，成员没有命名

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
// 两者是不同类型
```



Unit-like Struct，没有任何字段，类似空元组

只为实现trait，但不占用存储空间



结构体引用数据的有效性要与结构体本身保持一致



### 一个使用结构体的示例程序

`pintln!`，`{}`默认使用Display格式

struct加上`#[derive(Debug)]`注解，可使用`{:?}`

`{:#?}`则进行格式化



### 方法语法

方法定义在impl上下文内
```rust
struct Rect {
    width: u32,
    height: u32,
};

impl Rect {
    // self可以借用，可以mut，也可以转移所有权
    fn area(&self) -> u32 {
        self.width * self.height
    }
    // 关联方法，没有self，使用Rect::square调用
    fn square(size: 32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
};
```

自动引用和解引用，在方法的接受者明确时，会自动加上`&`，`&mut`或者`*`，以匹配方法签名

impl可以多个



## 枚举和模式匹配

### 定义枚举

枚举值可以是不同类型
```rust
enum IpAddrKind {
    V4,
    V6,
}

enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```



没有空值，只有使用Option时才可能会没有值，其他必定都是有效值

```rust
// 标准库中
enum Option<T> {
    Some(T),
    None,
}
```



### match控制流运算符

match的模式可以是任意类型，顺序匹配

绑定值的模式

```rust
enum Coin {
    Penny,
    Quarter(UsState),
}
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Quarter(state) => {
            25
        },
    }
}
```



`Option<T>`匹配

匹配要求是exhaustive，否则报错

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None, // 不匹配None会报错
        Some(i) => Some(i + 1),
    }
}
```

`_`匹配所有值



### if let简单控制流

相当于只match一个值做处理
```rust
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}

// 等同于
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```



## 使用包、Crate和模块管理不断增长的项目

### 包和crate

crate是一个二进制项或者库

package至少包含一个crate，包含一个Cargo.toml，描述如何构建crate

- 只能包含一个library crate
- 可以包含任意个binary crate

src/[main.rs](http://main.rs)是与包同名的binary crate的根

若目录包含src/[lib.rs](http://lib.rs)，则包带有与其同名的library crate，且src/[lb.rs](http://lb.rs)是crate根

src/bin下每个文件会被编译为独立的binary crate



### 定义模块来控制作用域与私有性

模块mod：crate的代码分组，可以控制可见性

mod可以嵌套

模块的根模块就是crate



### 路径用于引用模块树中的项

- 绝对路径：从crate根开始，以crate名或者字面量crate开头
- 相对路径：从当前模块开始，以self、super（从父模块开头）或当前模块标识符开头

默认项都是私有的，父模块不能使用子模块的私有项，但子模块可以使用父模块中的项（上下文）

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();
    // Relative path，front_of_house是同一模块所以可以访问
    front_of_house::hosting::add_to_waitlist();
}

fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }
    fn cook_order() {}
}
```



pub struct的成员仍为私有

pub enum的成员为公有



### 使用use关键字将名称引入作用域

use相当于软链接

习惯上，引入函数时定向到作用域，而引入结构体、枚举时则定向到完整路径

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use front_of_house::hosting;

pub fn eat_at_restaurant() {
    // 保留作用域
    hosting::add_to_waitlist();
}

use std::collections::HashMap;

fn main() {
    // 引入了完整路径
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```



as解决重名问题

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {}
fn function2() -> IoResult<()> {}
```



pub use重导出，原本私有的可以导出访问

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    // 外部也可以用hosting，即使front_of_house非公有
    hosting::add_to_waitlist();
}
```



相同前缀的可以化简

```rust
use std::io::{self, Write};
```



使用`*`把所有公有项引入

```rust
use std::collections::*;
```



### 将模块分割进不同文件

src/lib.rs

```rust
mod front_of_house;  // 只是声明

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```



src/front_of_house.rs

```rust
pub mod hosting;
```



src/front_of_house/hosting.rs

```rust
pub fn add_to_waitlist() {}
```



## 常见集合

集合指向的数据是存储在堆上的

### vector

如果需要存入不同类型的对象，可以用enum或者trait对象
```rust
// 无法推断，需要类型注解
let v: Vec<i32> = Vec::new();
// 用宏初始化
let v = vec![1, 2, 3];
```



[]不存在时会panic，get会放回None

```rust
let mut v = vec![1, 2, 3, 4, 5];
let first = &v[0]; // v不可变引用
v.push(6); // 可变引用，报错，因为v可能扩容导致first失效
println!("The first element is: {}", first);
```



遍历

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```



### 字符串

语言中只有一种字符串类型，str（&str，字符串slice）

String是标准库提供的，都是UTF-8编码

其他字符串类型，对应不同的所有权和字符串形式（编码 / 存储）

```rust
// 只是风格不同
let s = "initial contents".to_string();
let s = String::from("initial contents");
```



String不支持索引，len返回的是字节数而不是字符串长度

转换为字符串slice，如果没有对齐也会panic

```rust
let hello = "Здравствуйте";
let s = &hello[0..4];
// &hello[0..1] panic
```



- chars()按字符返回
- bytes()按字节



### 哈希map

默认使用密码学安全的哈希函数，可以改用其他追求速度

构造（不在prelude，没有宏）

```rust
use std::collections::HashMap;
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

// 用vec构造
let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];
// HashMap注解必需，因为collect有泛型参数，可以返回不同类型，所以需要显式指定
let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```



不存在时插入，使用entry

```rust
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.entry(String::from("Yellow")).or_insert(50);

let text = "hello world wonderful world";
let mut map = HashMap::new();
for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}
```



## 错误处理

### panic!与不可恢复的错误

`panic!`宏，默认行为展开（ortunwinding），打印错误，展开并清理栈，最后退出

可以设置为直接终止（abort），不清理数据直接退出。在Cargo.toml的`[profile]`增加panic = ‘abort’

环境变量RUST_BACKTRACE设置为非0，可以查看backtrace



### Result与可恢复的错误

标准库
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```



与Option一样，Result及其成员也被倒入到prelude中（std和std::prelude默认导入）

```rust
use std::fs::File;
fn main() {
    let f = File::open("hello.txt");
    let f = match f {
        Ok(file) => file,
        // 不同错误匹配
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

match太长，`Result<T, E>`定义了辅助方法进行处理。

unwarp：如果Result的值是Ok则返回Ok中的值，如果是Err则调用panic!

except：类似，但可以自定义错误信息



错误传播：?运算符

如果Ok则继续执行；如果Err，则作为函数返回值

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");
    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    let mut s = String::new();
    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```



`?`（定义于标准库From trait）与match的另一个区别是，会调用from函数把错误类型转换为当前函数返回的错误类型

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```



## 泛型、trait和生命周期

### 泛型数据类型

没有任何性能损失，编译时进行泛型代码的单态化（monomorphization）（编译时填充具体使用的类型，把通用代码转为特定代码的过程）
```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

struct Point<T, U> {
    x: T,
    y: U,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```


### trait: 定义共享的行为

类似于接口
```rust
pub trait Summary {
    // 可以加上默认实现
    fn summarize(&self) -> String;
}
```



实现

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    // 用默认实现则为空
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
```



只有当trait或者要实现trait的类型位于crate的本地作用域时，才能为该类型实现trait

不能为外部类型实现外部trait

trait可以作为函数参数和返回值

```rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
pub fn notify(item: impl Summary + Display) { }
```



trait bound可以实现比impl更复杂的约束

```rust
pub fn notify<T: Summary>(item1: T, item2: T) { }
pub fn notify<T: Summary + Display>(item: T) { }
```



用where简化trait bound

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 { }
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug { }
```



在实现了特定trait的类型有条件地实现trait（blanket implementations）

```rust
impl<T: Display> ToString for T { }
```



### 生命周期与引用有效性

```rust
// 无法确定返回的引用是x还是y，借用检查器无法确认引用是有效的
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```



生命周期注解，不会改变任何引用的生命周期，描述了多个引用生命周期的相互关系

生命周期参数必须以`'`开头，注解在`&`之后

注解只存在于签名，不在实现中

```rust
// ‘a是x和y作用域的重叠部分
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```



结构体内的引用生命周期注解

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```



Rust内置了一些模式，某些情况下可以推断出生命周期，而不需要显式增加注解

生命周期省略规则（检查完这三条规则仍存在没有计算出生命周期的引用，则报错）

- 每个参数都有各自的生命周期参数
- 如果只有一个输入生命周期参数，则它被赋予所有输出生命周期参数
- 如果方法有多个输入生命周期参数（其中之一为&self或&mut self），则self的生命周期赋给所有输出生命周期参数

静态生命周期`'static`

存活于整个程序期间，所有字符串字面量都是`'static`



## 测试

### 编写测试

cargo test会执行所有带test属性的函数

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle { width: 8, height: 7 };
        let smaller = Rectangle { width: 5, height: 1 };

        assert!(larger.can_hold(&smaller));
    }
}
```

assert的两个参数为left和right，不区分actual和excepted

`#[should_panic]`检查代码必定panic



通过返回`Result<T, E>`进行测试

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```



### 运行测试

默认并行执行

- --test-threads控制线程数

- --nocapture，查看测试中打印的值
- 可以只运行名字匹配的测试

- `#[ignore]`忽略测试，cargo test -- --ignored，只运行ignore的测试



### 测试的组织结构

单元测试与要测试代码存放在相同的文件，因为在同一个模块，私有函数也可以测试

`#[cfg(test)]`声明模块只有在test才编译和运行



集成测试位于tests目录

cargo test --test integration_test，只运行某个文件

tests/common/*下可放通用逻辑，Ruct不会把common看成是测试文件



## 迭代器和闭包

### 闭包

闭包的参数和返回值类型自动推导（也允许添加注解）

只有一行可以忽略大括号

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```



只推断一次

```rust
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);  // 报错，类型不匹配
```



所有闭包都实现了Fn / FnMut / FnOnce中的一个（函数也实现了）

- FnOnce：从环境捕获变量，Once表示不会多次获取相同变量的所有权
- FnMut：获取可变的借用值
- Fn：获取不可变的借用值



move，强制获取环境值的所有权

```rust
fn main() {
    let x = vec![1, 2, 3];
    let equal_to_x = move |z| z == x;
    // 报错，x已经被move
    println!("can't use x here: {:?}", x);
}
```



### 使用迭代器处理元素序列

lazy（类比xrange）

迭代器实现了Iterator trait，next是要求实现的唯一方法，每次返回Some，结束返回None

Consuming adaptors，调用他们会消耗迭代器

```rust
fn iterator_sum() {
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();

    // sum之后无法使用v1_iter
    let total: i32 = v1_iter.sum();
    assert_eq!(total, 6);
}
```



Iterator adaptors，把当前迭代器变为不同类型

```rust
let v1: Vec<i32> = vec![1, 2, 3];
// 报错，迭代器是惰性的，map实际上什么都没做
v1.iter().map(|x| x + 1);

// 使用collect消费
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
```



迭代器是Rustzero-cost abstractions之一

循环可能会被展开，性能和手写相当



## 智能指针

相比引用，智能指针可通过引用计数，允许数据有多个所有者

引用只借用数据，而智能指针拥有指向的数据

智能指针一般使用结构体实现，并实现了Deref和Drop trait



### 使用`Box<T>`指向堆上的数据

值在堆上，指针在栈。用法上与直接访问栈上的值没有区别。
```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```



可以创建递归类型

```rust
// 无限大小，无法创建
enum List {
    Cons(i32, List),
    Nil,
}
enum List {
    Cons(i32, Box<List>),
    Nil,
}
```



### 通过Deref trait将智能指针当作常规引用处理

Deref trait允许使用解引用运算符`*`

常规引用是指针类型

```rust
use std::ops::Deref;

// 自定义智能指针，元组结构体
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;
    fn deref(&self) -> &T {
        &self.0
    }
}
```



`*`会展开为，`*(x.deref())`，其中展开后的`*`是普通解引用，不会递归

deref返回引用，是为了避免转移所有权



解引用强制多态

就是在传参过程中，Rust会尝试调任意多次Deref::deref以匹配参数类型

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m); // &m解引用获取&String，&String解引用获取&str
    // hello(&(*m)[..]); 若无解引用强制多态
}
```



DerefMut trait用于重载可变引用的`*`运算符，满足以下情况会进行解引用强制多态

- T: Deref<Target=U>，从&T到&U
- T: DerefMut<Target=U>，从&mut T到&mut U
- T: Deref<Target=U>，从&mut T到&U（可变转为不可变，反之不可）



### 使用Drop Trait运行清理代码

离开作用域时执行，Drop trait包含在prelude中

按创建顺序反向调用



不能禁用drop，但可以提早释放（不能显式调用drop，Rust会插入drop导致double free）

调用(std::mem::)drop可以显式清理（prelude中）



### `Rc<T>`引用计数智能指针

引用计数，只能用于单线程

Rc::clone区别于clone，只增加引用计数，不会做深拷贝

Rc::strong_count，强引用计数

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```



### `RefCell<T>`和内部可变性模式

内部可变性，Rust的一个设计模式，允许在有不可变引用时改变数据

只用于单线程，`RefCell<T>`代表数据的唯一所有权

RefCell相当于把可变借用的检查从编译期移到了运行期，若违反了则panic

```rust
struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }
    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            // self是不可变，但依然可以从sent_messages取出可变引用
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }
}
```



`Rc<T>`和`RefCell<T>`结合，拥有多个可变数据的所有者

```rust
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));
    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));
    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));
    *value.borrow_mut() += 10;
    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```



### 引用循环与内存泄漏

Rust编译期拒绝数据竞争，但不保证完全避免内存泄漏，引用循环的可能性是存在的

Rc::downgrade用Rc实例创建弱引用`Weak<T>`

`Weak<T>` upgrade会获取`Option<Rc<T>>`



## 进一步认识Cargo和Crates.io

### 采用发布配置自定义构建

`[profile.dev / release]`可以覆盖`[profile]`配置

opt-level，优化等级（0到3），dev默认0，release默认3



### 将Crate发布到Crates.io

文档注释`///`（常用部分：Panices，Errors，Safety）

`cargo doc --open`构建当前文档（HTML）

文档注释也可以作为测试

`//!`是作为整个模块的文档，一般在src/lib.rs文件开头



发布时license必需

Rust自身使用的是双许可MIT OR Apache-2.0



发布是永久的，不可以覆盖也不能删除

cargo yank撤回指定版本，阻止新项目依赖改版本（但已有的Cargo.lock不影响）



### Cargo 工作空间

多个crate，共享Cargo.lock和输出目录

顶级Cargo.toml指定所有的空间（目录）

```toml
[workspace]

members = [
    "adder",
    "add-one",
]
```



可以指定另一个空间为依赖（cargo不会假定互相依赖）

```toml
[dependencies]

add-one = { path = "../add-one" }
```



指定运行 / 测试目录

```shell
cargo run -p adder
cargo test -p add-one
```



cargo publish不提供all选项，需要每个crate独立发布



### 使用cargo install从Crates.io安装二进制文件

安装二进制，默认目录为`$HOME/.cargo/bin`

```shell
cargo install ripgrep
```



### Cargo 自定义扩展命令

如果`$PATH`下有cargo-something的二进制文件，就可以通过`cargo something`来执行。

`cargo --list`可显示所有命令



## 无畏并发

### 线程

thread::spawn接收闭包创建线程
```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];
    // 转移所有权，不使用move会报错
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });
    handle.join().unwrap();
}
```

### 消息传递

channel，由发送者和接收者组成，任一被丢弃时可以认为通道被关闭

mpsc::channel，多生产者，单消费者

mpsc::Sender::clone，可以复制生产者

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap(); // 若接收者被丢弃则返回错误，val的所有权也被转移
    });
    // rx也可以作为迭代器，不显式调用recv
    let received = rx.recv().unwrap(); // recv会阻塞，对应try_recv
    println!("Got: {}", received);
}
```

### 共享状态

`Mutex<T>`，lock返回MutexGuard的智能指针

与`RefCell<T>`类似，`Mutex<T>`同样提供了内部可变性

也会存在死锁的可能



`Arc<T>`与`Rc<T>`类似，但使用原子引用计数

```rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    for handle in handles {
        handle.join().unwrap();
    }
    println!("Result: {}", *counter.lock().unwrap());
}
```



### 可扩展的并发：Sync与Send

std::marker的Sync和Send trait

Send表明类型的所有权可以在线程间传递，几乎所有Rust类型都是Send（裸指针、`Rc<T>`不是），完全由

Send类型组成的类型也会自动标记为Send

Sync表明可以安全地在多个线程中拥有其值的引用

基本类型是Sync，完全由Sync类型组成的类型也是Sync



`Rc<T>` / `RefCell<T>` / `Cell<T>`不是Sync，`Mutex<T>`是Sync

一般来说，手动实现Send和Sync都是不安全的，因为有自动标记

语言层面除此之外基本没有处理并发，而是由crate实现



## Rust的面向对象编程特性

### 面向对象语言的特点

Rust没有继承，而是基于trait解决方案（trait只抽象了行为，没有数据）

继承的两个主要目的：

- 代码重用：trait可以提供默认实现，实现trait的类型也可以重写方法

- 多态：通过泛型加上trait bounds约束，bounded parametric polymorphism



### 为使用不同类型的值而设计的trait对象

```rust
pub trait Draw {
    fn draw(&self);
}

pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {}
}

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {}
}
```



如果使用泛型（静态分发），则在编译期就进行单态化，则`Vec`只能放`Button`或者`SelectBox`

```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
    where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```



trait对象执行动态分发，就是相当于虚表实现（有性能影响）

```rust
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```



trait对象要求是对象安全。

- 返回值不为Self
- 方法没有任何泛型类型参数

简单来说就是使用trait对象时，已经丢失了具体类型的信息，使用这部分已丢失的信息会报错

```rust
pub trait Clone {
    fn clone(&self) -> Self;
}

pub struct Screen {
    // 报错，Clone非对象安全
    pub components: Vec<Box<dyn Clone>>,
}
```



### 面向对象设计模式的实现

传统的状态机实现，缺点

- 状态之间存在耦合，因为要处理跳转逻辑
- 存在重复实现，但无法提供默认实现，return self不是对象安全的
- 对外的接口基本上都是转发到状态实现，存在代码重复

```rust
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    // --snip--
    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }
  
    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    fn approve(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
    fn approve(self: Box<Self>) -> Box<dyn State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```



把状态映射为类型编码

```rust
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

impl Post {
    pub fn new() -> DraftPost {
        DraftPost {
            content: String::new(),
        }
    }

    pub fn content(&self) -> &str {
        &self.content
    }
}

impl DraftPost {
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }

    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost {
            content: self.content,
        }
    }
}

pub struct PendingReviewPost {
    content: String,
}

impl PendingReviewPost {
    pub fn approve(self) -> Post {
        Post {
            content: self.content,
        }
    }
}

fn main() {
    // 类型的变化就是状态的切换
    let mut post = Post::new();
    post.add_text("I ate a salad for lunch today");
    let post = post.request_review();
    let post = post.approve();

    assert_eq!("I ate a salad for lunch today", post.content());
}
```



## 模式用来匹配值的结构

### 所有可能会用到模式的位置

- match分支：要求穷尽
- if let条件表达式：可以不穷尽，与match一样也支持覆盖变量

```rust
if let Ok(age) = age

// 报错，因为age在body才生效
if let Ok(age) = age && age > 30
```

- while let条件循环

- for循环

```rust
let v = vec!['a', 'b', 'c'];
for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

- let语句

```rust
let (x, y, z) = (1, 2, 3);
```

- 函数参数

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```



### Refutability：何时模式可能会匹配失败

irrefutable：能匹配任何传递的可能值模式（函数参数、let、for）

refutable：某些可能的值匹配会失败（if let、while let）

```rust
let Some(x) = some_option_value;  // 报错，因为匹配不了None
if let Some(x) = some_option_value {
    println!("{}", x);
}

// 报错，因为必定匹配
if let x = 5 {
    println!("{}", x);
};
```



### 模式的全部语法

匹配多个模式`|`

```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```



匹配值的范围`..=`，只支持char和数字值

```rust
let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}
```



解构结构体，同名可以简写

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    // let Point { x: x, y: y } = p;
    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
  
    // 部分创建变量
    let p = Point { x: 0, y: 7 };
    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```



解构枚举

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }
}
```



解构嵌套的结构体和枚举

```rust
enum Color {
   Rgb(i32, i32, i32),
   Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!(
                "Change the color to hue {}, saturation {}, and value {}",
                h,
                s,
                v
            )
        }
        _ => ()
    }

    let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
}
```



`_`忽略整个值或部分值，变量名前加上`_`可以忽略变量未使用的警告

`..`可以忽略剩余值，有歧义的时候报错

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        },
    }
}
```



match guard：match分支模式之后的额外if（`|`的优先级比`if`高）

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {}", n),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {}", x, y);
}
```



`@`可以在一个模式中同时测试和保存变量值

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```



## 高级特征

### 不安全的Rust

unsafe块中执行操作
- 解引用裸指针
- 调用不安全函数或方法
- 访问或修改可变静态变量
- 实现不安全trait（例如Send / Sync）
- 访问union字段

unsafe中依然会执行其他安全检查，包括引用



裸指针：`*const T` / `*mut T`

- 允许忽略借用规则，可以同时拥有多个可变和不可变指针
- 不保证指向有效的内存
- 允许为空
- 没有自动清理功能
```rust
let mut num = 5;
// 创建裸指针不是unsafe，解引用才是unsafe
let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
let address = 0x012345usize;
let r = address as *const i32;
```



unsafe标识的函数，unsafe的函数内不需要再加unsefe

```rust
unsafe fn dangerous() {}
unsafe {
    dangerous();
}
```



用unsafe创建不安全代码的安全抽象（就是逻辑上是安全的，但编译器无法识别）

```rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();
    assert!(mid <= len);
    unsafe {
        (slice::from_raw_parts_mut(ptr, mid),
         slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid))
    }
    // 创建了两个可变引用，虽然不重叠，但依然会报错
    // (&mut slice[..mid],
    //    &mut slice[mid..])
}
```



调用外部extern的代码需要unsafe

外部调用Rust需要增加`#[no_mangle]`注解

```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}

extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```



静态变量，访问和修改都是unsafe

```rust
static HELLO_WORLD: &str = "Hello, world!"; // 必定是’static，省略
```



### 高级trait

关联类型

```rust
impl Iterator for Counter {
    type Item = u32;
    fn next(&mut self) -> Option<Self::Item> { }
}

// 假如使用泛型，则使用的时候就必需显式指定类型
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```



Rust不允许创建自定义运算符或重载任意运算符，但可以通过std::ops列出的运算符和实现运算符相关trait实现重载

```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
}
```



默认类型参数

```rust
trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
```



同名方法消除歧义

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}

fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```



若没有self参数，则需要使用完全限定语法

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());

    // 报错，无法确定使用哪个实现
    println!("A baby dog is called a {}", Animal::baby_name());
}
```



trait之间的依赖

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
    }
}
```



使用newtype模式可以为外部类型实现外部trait

```rust
use std::fmt;

// Wrapper是新类型，如果要实现Vec<T>的所有方法，可以实现Deref trait代理到self.0
struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```



### 高级类型

newtype模式也可以用于区分静态类型，隐藏内部泛型实现。



类型别名

```rust
// 与newtype不同，只是别名，依然是i32
type Kilometers = i32;

type Result<T> = std::result::Result<T, std::io::Error>;
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: Arguments) -> Result<()>;
}
```



`!`特殊类型，empty type / never type，可以转为任意类型

发散函数，从不返回的函数

```rust
fn bar() -> ! {
    // --snip--
}

// continue的值是!，事实上Err匹配时guess没有赋值
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};

// panic!同样是!类型，所以能编译通过
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}

// 死循环表达式的值是!
loop {
    print!("and ever ");
}
```



动态大小类型（DST，dynamically sized types）

str是一个DST，所以不能创建str类型变量，只能用&str

DST的值必需置于某种指针（`Box<str>` / `Rc<str>`）

Sized trait，在编译期已知类型大小，每一个泛型都隐式添加了Sized bound

```rust
fn generic<T>(t: T) {}
// 实际处理
fn generic<T: Sized>(t: T) {}

// ?可能是Sized，所以返回值要改为引用
fn generic<T: ?Sized>(t: &T) {}
```



### 高级函数与闭包

函数指针，类型为`fn`（区别于`Fn` trait）

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

// 不能接收闭包
fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);
    println!("The answer is: {}", answer);
}
```



初始化构造枚举

```rust
enum Status {
    Value(u32),
    Stop,
}

let list_of_statuses: Vec<Status> =
    (0u32..20)
    .map(Status::Value)
    .collect();
```



闭包表现为trait，所以不能直接返回闭包（非Sized， 要使用dyn）

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```



### 宏

宏（元编程）只接受一个可变参数，在编译器翻译代码前展开，因此可以给指定类型实现trait，但函数是运行时被调用。调用宏之前必须先定义并引入作用域。



声明宏，macro_rules!（deprecated）

```rust
#[macro_export]
macro_rules! vec {
    // 单边模式
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

let v: Vec<u32> = vec![1, 2, 3];
```



过程宏

- 自定义derive
- 类属性
- 类函数