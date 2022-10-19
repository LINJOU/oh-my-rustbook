# Rust

### Gargo

#### 认识Gargo

一个Rust构建工具和包管理器。安装完成Rust环境之后，Cargo也附带安装。在实际的项目开发过程中，建议用Cargo来管理项目，方便维护。

##### Gargo常用命令



### 变量绑定与解构

#### 命名规范

[rust命名规范]: https://course.rs/practice/naming.html

+ 对于 **type-level** 的构造，用**驼峰命名法**（eg：UpperCamelCase）
  + 类型 Types
  + 特征 Traits（特征的名称应该使用动词，而不是形容词或者名词。eg：`Print`好于`Printable`）
  + 枚举 Emumerations
  + 结构体 Structs
  + 类型参数 Type parameters (通常使用一个大写字母: `T`)
+ 对于 **value-level** 的构造，用**蛇形命名法**（eg：snake_case）
  + 模块 Modules
  + 函数 Functions
  + 方法 Methods
  + 通用构造器 General constructors
  + 转换构造器 Conversion constructors
  + 宏 Macros
  + 局部变量 Local variables
  + 静态类型 Statics (所有字母需大写，eg：SCREAMING_SNAKE_CASE)
  + 常量 Constants (所有字母需大写，eg：SCREAMING_SNAKE_CASE)

+ 其他
  + 包 Crates | Features ： [unclear](https://github.com/rust-lang/api-guidelines/issues/29)
  + 生命周期 Lifetimes：通常使用小写字母: `'a`，`'de`，`'src`

+ **关键字**

  https://course.rs/appendix/keywords

#### 变量名绑定

在rust中使用 **let** 来对变量进行绑定。对于`let a = "hello"`，在其他语言中表示将字符串`"hello"`赋值给`a`，在rust中这个过程称为**变量绑定**，即将字符串`"hello"`绑定给`a`。

由于rust最核心的原则(**所有权**)，使用绑定的含义更加清晰准确。

#### 变量可变性

##### 手动设置变量可变性的意义

其他语言中，有只支持可变的变量提供灵活性，有只支持不可变的变量(如：函数式语言)确保安全性。

rust通过手动设置的方式做到两者兼顾，既提供灵活性(虽然会提高语言底层代码的实现复杂度)，又确保安全性。除了这两个优点外，不可变变量在运行期会避免一些多余的`runtime`的检查，提升程序运行的性能。

##### rust的变量默认是不可变的

rust中的变量默认不可变是rust的特性之一，确保编写的代码的更安全，性能更好。

eg：变量`a`不可变，一旦为它绑定值，就不能修改`a`

```rust
fn main() {
    let a = 5;
    println!("{}", a);
    a = 6;
    println!("{}", a);
}
```

```shell
error[E0384]: cannot assign twice to immutable variable `a`
// 无法对不可变的变量 `a` 进行重复赋值
   --> src\main.rs:118:5
    |
116 |     let a = 5;
    |         -
    |         |
    |         first assignment to `a`
    |         help: consider making this binding mutable: `mut a`
117 |     println!("{}", a);
118 |     a = 6;
    |     ^^^^^ cannot assign twice to immutable variable
```

这样是为了避免出现无法预期的错误，当一个变量被多处代码使用，其中一部风代码假定该变量的值是不会改变的，另一部分代码却需要改变这个值，这样的错误在编程中(特别是多线程)很难被发现。同时也为代码的阅读带来便利。

##### 通过 mut 关键字让变量变为可变（只能修改值，而不能修改类型）

为了编程的灵活性，可变性也是非常重要的，不然每次要改变变量值，就要重新生成一个对象(涉及到内存对象的再分配)，这样会导致性能低下，内存拷贝成本异常高。

eg：在使用大型数据结构或热点代码路径(被大量频繁调用)的情形下，在同一内存位置更新实例可能比复制并返回新分配的实例要更快。使用较小的数据结构时，通常创建新的实例并以更具函数式的风格来编写程序，可能会更容易理解，所以值得以较低的性能开销来确保代码清晰。

在rust中可通过在变量名前添加 `mut` 显性声明的方式来说明该变量为可变的。

```rust
fn main() {
    let mut a = 5;
    println!("{}", a);
    a = 6;
    println!("{}", a);
}
```

```shell
5
6
```

**注意：mut只能修改值，而不能修改类型**

```rust
let mut a: i32 = 1;
a = "str";
```

编译器报错：

```shell
error[E0308]: mismatched types
   --> src\main.rs:159:9
    |
158 |     let mut a: i32 = 1;
    |                --- expected due to this type
159 |     a = "str";
    |         ^^^^^ expected `i32`, found `&str`
```

##### 变量解构

`let`表达式不仅用于变量的绑定，还能进行复杂变量的解构：从一个相对复杂的变量中，匹配出该变量的一部分内容

```rust
fn main() {
    let (a, mut b): (bool, bool) = (true, false);
    println!("a = {:?}, b = {:?}", a, b);

    b = true;
    assert_eq!(a, b);
}
```

```shell
a = true, b = false
```

##### 解构式赋值

在`rust 1.59`之后，支持赋值语句的左式中使用元组、切片和结构体模式

```rust
struct Struct {
    e: i32
}

fn main() {
    let (a, b, c, d, e);

    (a, b) = (1, 2);
    [c, .., d, _] = [1, 2, 3, 4, 5];
    Struct {e, ..} = Struct {e: 5};

    println!("a: {}, b: {}, c: {}, d: {}, e: {}", a, b, c, d, e);
}
```

```shell
a: 1, b: 2, c: 1, d: 4, e: 5
```

这种使用方式与之前`let`保持一致性，但`let`会进行重新绑定，这里仅仅只是对之前绑定的变量进行再赋值。

注意使用`+=`的赋值语句还不支持解构式赋值。

##### 变量与常量之间的差异

+ 常量不允许使用`mut`。常量不只默认不可变，是自始至终不可变。变量在编译完成后，已经确定它的值。

+ 常量使用`const`关键字而不是`let`关键字来声明，并且值的类型**必须**标注。

+ 常量命名时约定全部字母都使用**大写**，并使用下划线分隔单词，另外对数字字面量可插入下划线以提高可读性。

  ```rust
  const MAX_POINTS: u32 = 100_000; 
  ```

+ 常量可以在任意作用域内声明，包括全局作用域，在声明作用域内，常量在程序运行的整个过程都有效。对于需要多处代码共享一个不可变的值是非常有用的。

##### 变量遮蔽(shadowing)

rust允许声明相同的变量名，但后面声明的变量会覆盖掉前面声明的变量。

```rust
fn main() {
    let x = 5;
    // 在main函数的作用域内对之前的x进行遮蔽
    let x = x + 1;

    {
        // 在当前的花括号作用域内，对之前的x进行遮蔽
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}
```

```shell
The value of x in the inner scope is: 12
The value of x is: 6
```

这个程序中先给`x`绑定数值5，然后通过重复声明`x`来遮蔽之前的x，并取原来的值 加1，最终`x`的值为6。

###### 变量遮蔽与 mut 声明变量的区别

变量遮蔽与`mut`变量的使用是不同的。

+ 是否涉及一次内存对象再分配
  + 变量遮蔽：第二个`let`生成的是不同的新变量，两个变量只是拥有相同名称，但涉及一次内存对象再分配。
  + `mut`声明的变量：修改同一内存地址上的值，没有发生内存对象再分配，性能要更好。

+ 是否能变更后续值的类型

  + 变量遮蔽可以变更

    ```rust
    // 字符串类型
    let spaces = "";
    // usize数值类型
    let spaces = spaces.len();
    ```

  + `mut`声明的变量不可变更

    ```rust
    let mut spaces = "";
    
    spaces = spaces.len();
    ```

    编译器报错：不允许将整数类型 `usize` 赋值给字符串类型

    ```shell
    error[E0308]: mismatched types
     --> src/main.rs:3:14
      |
    3 |     spaces = spaces.len();
      |              ^^^^^^^^^^^^ expected `&str`, found `usize`
    
    error: aborting due to previous error
    ```

###### 变量遮蔽的作用

在某个作用域内无需再使用之前的变量（在被遮蔽后，无法再访问到之前的同名变量），就可以重复的使用变量名字。

### 数据类型

rust是一门静态类型语言，每个值必须要有确切的数据类型。

#### 字面量(Literal)

字面量是用于表达源代码中一个固定值的表示法。对于字面量只能作为等号右边的值出现。几乎所有编程语言都具有对基本值的字面量表示，如：整数，浮点数以及字符串。

#### 类型推导与标注

rust编译器必须在编译期知道变量类型，但不意味着在编写代码时必须为每个变量指定类型，编译器会根据变量的值和上下文的使用方式来自动推导出变量的类型，但在某些情况下编译器无法推断出来就需要手动给予一个类型标注。

```rust
let guess = "40".parse().expect("Not a number!");
```

```shell
error[E0282]: type annotations needed
   --> src\main.rs:165:9
    |
165 |     let guess = "40".parse().expect("Not a number!");
    |         ^^^^^ consider giving `guess` a type
```

这段代码目的是对字符串“40”进行解析，而编译器在这里无法推导出我们想要的类型，因此报错。需手动给予一个类型标注。

```rust
let guess: i32 = "40".parse().expect("Not a number!");
```

#### 基本类型

基本类型意味着它是一个**最小化**原子类型，无法解构为其它类型。

##### 数值类型

###### 整数类型

**类型定义的形式：** `有|无符号 + 类型位数`

```rust
let a: i32 = 1;
let b: u32 = -1;
```

**rust内置的整数类型**

|                长度(位数)                 |  有符号类型   | 无符号类型 |
| :---------------------------------------: | :-----------: | :--------: |
|                     8                     |      i8       |     u8     |
|                    16                     |      i16      |    u16     |
|                    32                     | **i32(默认)** |    u32     |
|                    64                     |      i64      |    u64     |
|                    128                    |     i128      |    u128    |
| 由CPU位数而定，若为32\|64位，则为32\|64位 |     isize     |   usize    |

**rust 整数类型默认使用 `i32`**

**整数各类型所代表数值范围**

| 类型       | 表示数值范围(n：长度\|位数)                  |
| ---------- | -------------------------------------------- |
| 有符号类型 | -(2<sup>n - 1</sup>) ~ 2<sup>n - 1</sup> - 1 |
| 无符号类型 | 0 ~ 2<sup>n - 1</sup>                        |

**整数类型字面量书写形式**

|    数字字面量    |    示例     |
| :--------------: | :---------: |
|      十进制      |   98_222    |
|     十六进制     |    0xff     |
|      八进制      |    0o77     |
|      二进制      | 0b1111_0000 |
| 字节(仅限于`u8`) |    b'A'     |

**整型溢出(integer overflow)**

假设一个数值类型为`u8`，则它的表示范围为0 ~ 255，当该数值超出255则会发生整型溢出。

rust处理整型溢出规则：

```rust
let a: u8 = 255 + 1;
println!("a: {}", a);
```

+ 默认情况下，在debug模式编译时，rust会检查溢出，一旦发生溢出，则会引发`pacic`(崩溃)

  ```shell
  error: this arithmetic operation will overflow
     --> src\main.rs:165:17
      |
  165 |     let a: u8 = 255 + 1;
      |                 ^^^^^^^ attempt to compute `u8::MAX + 1_u8`, which would overflow
      |
      = note: `#[deny(arithmetic_overflow)]` on by default
  ```

+ 使用 `--release`参数进行`release`模式构建时，rust出于性能原因运行时**不**检查溢出，但当发生整型溢出时，rust会按照补码循环溢出（*two’s complement wrapping*）的规则处理。

  ```shell
  cargo run --release
  ```

  ```shell
  a: 0
  ```

显式处理可能溢出，可以使用标准库针对原始数字类型提供的这些方法：

+ 使用 `wrapping_*` 方法在所有模式下都按照补码运算溢出规则处理(直接抛弃已经溢出的最高位，将剩下的部分返回)，eg：wrapping_add

  ```rust
  let a: u8 = 255;
  let b: u8 = 254;
  println!("a: {}", a.wrapping_add(1));
  println!("b: {}", b.wrapping_add(1));
  ```

  ```shell
  a: 0
  b: 255
  ```

+ 如果使用 `checked_*` 返回的类型是`Option<_>`，当出现溢出的时候，返回值是`None`；

  ```rust
  let a: u8 = 255;
  let b: u8 = 254;
  println!("a: {:?}", a.checked_add(1));
  println!("b: {:?}", b.checked_add(1));
  ```

  ```shell
  a: None
  b: Some(255)
  ```

+ 使用 `overflowing_*` 方法返回该值和一个指示是否存在溢出的布尔值

  ```rust
  let a: u8 = 255;
  let b: u8 = 254;
  println!("a: {:?}", a.overflowing_add(1));
  println!("b: {:?}", b.overflowing_add(1));
  ```

  ```shell
  a: (0, true)
  b: (255, false)
  ```

+ 使用 `saturating_*` 方法返回类型是整数，如果溢出，则给出该类型可表示范围的“最大/最小”值

  ```rust
  let a: u8 = 255;
  let b: u8 = 254;
  println!("a: {}", a.saturating_add(1));
  println!("b: {}", b.saturating_add(1));
  ```

  ```shell
  a: 255
  b: 255
  ```

###### 浮点类型

浮点类型数字是带有小数点的数字，rust中有两种基本类型：`f32` 和 `64`，分别为32位(单精度)和64位(双精度)。**默认为`f64`**，精度更高。

```rust
let a = 2.0; // f64
let b: f32 = 3.0; // f32
```

**浮点数陷阱**

浮点数由于底层格式的特殊性，使用时不够谨慎可能造成危险的两个原因：

1. **浮点数表示的值往往是想要数值近似表达**

   这与浮点数类型的底层实现有关，浮点数类型是基于二进制实现的，而我们的计算数字往往是十进制的。小数，如`0.1`在二进制并不存在精确的表达形式，但在十进制上存在，这种不匹配导致一定的歧义性。

   虽然浮点数能代表真实数值，但由于底层格式的问题(受限于定长的精度)，如想要完全精准的真实数值，只有使用无限精度的浮点数。

   ```rust
   fn main() {
       // 断言0.1 + 0.2 于 0.3相等
       assert!(0.1 + 0.2 == 0.3);
   }
   ```

   ```shell
   thread 'main' panicked at 'assertion failed: 0.1 + 0.2 == 0.3', src\main.rs:171:5
   note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
   error: process didn't exit successfully: `target\debug\world_hello.exe` (exit code: 101)
   ```

   这段代码实际运行时会`panic`，因为二进制精度问题，导致0.1 + 0.2 并不严格等于0.3，它们可能在小数点N位后存在误差。如非要进行比较，可采用`(0.1_f64 + 0.2 - 0.3).abs() < 0.00001`，具体小于多少，取决于精度需求。

   ```rust
      let a: (f32, f32, f32) = (0.1, 0.2, 0.3);
      let b: (f64, f64, f64) = (0.1, 0.2, 0.3);
   
      println!("a(f32)");
      println!("0.1 + 0.2: {:x}", (a.0 + a.1).to_bits());
      println!("0.3: {:x}", (a.2).to_bits());
   
      println!("b(f64)");
      println!("0.1 + 0.2: {:x}", (b.0 + b.1).to_bits());
      println!("0.3: {:x}", (b.2).to_bits());
   ```

   ```shell
   a(f32)
   0.1 + 0.2: 3e99999a
   0.3: 3e99999a
   b(f64)
   0.1 + 0.2: 3fd3333333333334
   0.3: 3fd3333333333333
   ```

   对`f32`类型做加法时，`0.1 + 0.2`的结果与`0.3`相同，但对于`f64`类型，由于精度高在小数点非常后面出现了微小的差异。

2. **浮点数在某些特性上是反直觉的**

   浮点数在某些场景下，并不能进行比较运算，这是由于`f32`，`f64`上的比较运算实现的是 `std::cmp::PartialEq` 特征，但是并没有实现 `std::cmp::Eq` 特征，但是后者在其它数值类型上都有定义。如：rust中的`HashMap`数据结构，是一个`KV`类型的`Hash Map`实现，它对于`K`没有特定类型的限制，但要求作为`K`类型必须实现了`std::cmp::Eq`特征，因此无法使用浮点数作为`HashMap`的`key`来存储键值对，而rust其他数值类型可以。

为了避免这两个陷阱，要注意

+ 避免在浮点数上测试相等性
+ 当结果在数学上可能存在未定义时，需要格外的小心

###### NaN(not a number)

rust中用来处理对于数学上未定义的结果。如：对负数取平方根 `-42.1.sqrt()` ，会产生一个特殊的结果。

所有跟`NaN`交互的操作，都会返回一个`NaN`,而且`NaN`不能用来比较，否则会崩溃。

```rust
fn main() {
    let x = (-42.0_f32).sqrt();
    assert_eq!(x, x);
}
```

```shell
thread 'main' panicked at 'assertion failed: `(left == right)`
  left: `NaN`,
 right: `NaN`', src\main.rs:185:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

**is_nan()**

用于判断一个数值是否是`NaN`

```rust
let x = (-42.0_f32).sqrt();
if x.is_nan() {
    println!("未定义的数学行为");
}
```

```shell
未定义的数学行为
```

###### 数字运算

rust 支持所有数字类型的基本数学运算：加法、减法、乘法、除法和取模运算。

```rust
fn main() {
    // 加法
    let sum = 5 + 10;
    // 减法
    let difference = 95.5 - 4.3;
    // 乘法
    let product = 4 * 30;
    // 除法
    let quotient = 56.7 / 32.2;
    // 求余
    let remainder = 43 % 5;

    println!("sum:{}, difference:{}, product:{}, quotient:{}, remainder:{}", sum, difference, product, quotient, remainder);
}
```

```shell
sum:15, difference:91.2, product:120, quotient:1.7608695652173911, remainder:3
```

[rust运算符与符号]([运算符与符号 - Rust语言圣经(Rust Course)](https://course.rs/appendix/operators.html#运算符))

**只有相同的类型才能运算**

```rust
let a: u32 = 8;
let b: u64 = 6;
println!("a - b = {}", a - b);
```

```shell
error[E0308]: mismatched types
   --> src\main.rs:208:32
    |
208 |     println!("a - b = {}", a - b);
    |                                ^ expected `u32`, found `u64`

error[E0277]: cannot subtract `u64` from `u32`
   --> src\main.rs:208:30
    |
208 |     println!("a - b = {}", a - b);
    |                              ^ no implementation for `u32 - u64`
    |
    = help: the trait `Sub<u64>` is not implemented for `u32`
```

**类型的标注**

```rust
// 编译器自动推导，给予twenty i32的类型
let twenty = 20;
// 类型标注
let twenty_one: i32 = 21;
// 通过类型后缀的方式进行类型标注：22是i32类型
let twenty_two = 22i32;

// 定义一个数组, 默认为f64类型
let forty_one = [
    42.0, 42, 42.0
]
// 定义一个f32数组，其中42.0会自动被推导为f32类型
let forty_two = [
    42.0, 42f32, 42.0_f32
]
```

**数字可读性处理**

```rust
// 对于较长的数字，可以用_进行分割，提升可读性
let one_million: i64 = 1_000_000;
println!("{}", one_million.pow(2));

let num = 45.31564;
// 打印num，并控制小数位为2位
println!("{:.2}", num);
```

```shell
1000000000000
45.32
```

###### 位运算

rust的运算基本上和其他语言一样

| 运算符  |                  说明                  |
| :-----: | :------------------------------------: |
| & 位于  |     相同位置均为1时则为1，否则为0      |
| \| 位或 |    相同位置只要有1时则为1，否则为0     |
| ^ 异或  |     相同位置不相同则为1，相同则为0     |
| ！位非  | 把位中的0和1相互取反，即0置为1，1置为0 |
| << 左移 |    所有位向左移动指定位数，右位补0     |
| >> 右移 |    所有位向右移动指定位数，左位补0     |

```rust
fn main() {
    let a: i32 = 2;
    // 二进制为0000 0010
    let b:i32 = 3;
    // 二进制为0000 0011
    
    println!("a & b = {}", a & b);
    println!("a | b = {}", a | b);
    println!("a ^ b = {}", a ^ b);
    println!("!a = {}", !a);
    println!("a << b = {}", a << b);
    println!("a >> b = {}", a >> b);

    let mut a = a;
    // 注意这些计算符除了!之外都可以加上=进行赋值 (因为!=要用来判断不等于)
    a <<= b;
    println!("a: {}", a);
}
```

```shell
a & b = 2
a | b = 3
a ^ b = 1
!a = -3
a << b = 16
a >> b = 0
a: 16
```

###### 序列(Range)

rust中用来生成连续数值。如：1..5，生成1到4的连续数字，不包含5；1..=5，生成1到5，包含5。常用于循环中。

```rust
for i in 1..3 {
    println!("{}", i);
}
```

```shell
1
2
```

```rust
for i in 1..=3 {
    println!("{}", i);
}
```

```shell
1
2
3
```

序列只允许用于**数字**与**字符类型**，因为它们可以连续，同时编译器在编译期可以检查该序列是否为空，字符和数字是rust中**仅有的**可以用于判断是否为空的类型。

```rust
let mut s = String::from("");
for i in 'a'..='z' {
    let s1 = &mut s;
    let s2 = String::from(i);
    s1.push_str(&s2);
}
println!("{}", s);
```

```shell
abcdefghijklmnopqrstuvwxyz
```

###### rust数值库 [num]([num - crates.io: Rust Package Registry](https://crates.io/crates/num))

rust的标准库并未包含有理数和复数，可通过社区开发的数值库`num`来解决，该库还可以解决：任意大小的整数和任意精度的浮点数，以及固定精度的十进制小数(常用于货币相关的场景)。

使用步骤：

1. 创建新工程`cargo new complex-num && cd complex-num`

2. 在`Cargo.toml`中的`[dependencies]`下添加`num = "0.4.0"`

3. 在`src/main.rs`中使用

   ```rust
   use num::complex::Complex;
   
    fn main() {
      let a = Complex { re: 2.1, im: -1.2 };
      let b = Complex::new(11.1, 22.2);
      let result = a + b;
   
      println!("{} + {}i", result.re, result.im)
    }
   ```

4. 运行`cargo run`

###### 总结

rust与其他语言的差异

+ rust拥有相当多的数值类型，这需要你熟悉这些类型所占字节数，来判断该类型允许的大小范围以及是否能表达负数。

+ 类型转换必须是显示，rust不会自动转换类型

+ rust在数值的使用时必须添加类型后缀，需要让编译器知道具体类型。如：将13.14取整

  ```rust
  let n = 13.14_f32.round();
  println!("{}", n);
  ```

  ```shell
  13
  ```

##### 字符类型(char)

rust的字符不仅仅是`ASCII`，所有的`Unicode`值都可作为rust字符，包括**单个**中文、日文、韩文、emoji表情符号等。`Unicode`值的范围从 `U+0000 ~ U+D7FF` 和 `U+E000 ~ U+10FFFF`。

rust的字符用**`''`**表示。

```rust
fn main() {
    let a = 'a';
    let b = 'B';
    let c = '早';
    let cat = '🐱';
    println!("a: {}, b: {}, c: {}, cat: {}", a, b, c, cat);
}
```

```shell
a: a, b: B, c: 早, cat: 🐱
```

由于`Unicode`都是4个字节编码，因此字符类型也占**4**个字节。

```rust
fn main() {
    let x = '好';
    println!("字符'好'占用{}个字节", std::mem::size_of_val(&x));
}
```

```shell
字符'好'占用4个字节
```

##### 布尔(bool)

rust中的布尔类型只有两个值：**`true | false`**，布尔值占用内存大小为**1**个字节。布尔类型主要用于流程控制。

```rust
fn main() {
    let t = true;
    // 使用类型标注,显式指定f的类型
    let f: bool = false;

    if t {
        println!("t");
    }

    if f {
        println!("f");
    }
}
```

```shell
t
```

##### 单元类型()

rust中单元类型用**`()`**表示，唯一值也为`()`。可作为一个值用来占位，并**不占用**任何内存。

`main`函数的返回值为单元类型`()`。如常见的`println!()`返回值也为单元类型`()`。再如用`()`作为`map`的值，表示不关注具体的值，只关注`key`。

rust中没有返回值的函数定义为：**`发散函数(diverge function)`**。

##### 语句和表达式

rust的函数体是有由一系类语句组成，最后由一个表达式来返回值。

```rust
fn add_with_extra(x: i32, y: i32) -> i32 {
    // 语句
    let x = x + 1;
    // 语句
    let y = y + 1;
    // 表达式
    x + y
}
```

语句会执行一些操作但不返回一个值，表达式会在求值后返回一个值。

对于rust而言语句和表达式是需要区分的。

###### 语句

```rust
let a = 1;
let b: Vec<f64> = Vec::new();
let (a, c) = ("hi", false);
```

以上都是语句，它们完成一个具体的操作，但没返回值。

```rust
let b = let a = 1;
```

```shell
error: expected expression, found statement (`let`)
// 期望表达式，却发现`let`语句
   --> src\main.rs:248:14
    |
248 |     let b = let a = 1;
    |             ^^^^^^^^^
    |
    = note: variable declaration using `let` is a statement

error[E0658]: `let` expressions in this position are unstable
// 下面的 `let` 用法目前是试验性的，在稳定版中尚不能使用
   --> src\main.rs:248:14
    |
248 |     let b = let a = 1;
    |             ^^^^^^^^^
    |
    = note: see issue #53667 <https://github.com/rust-lang/rust/issues/53667> for more information
```

`let`是语句，不是表达式，它不返回值，也不能给其它变量赋值。

`let`作为表达式已是试验功能，可能在后续的 [stable rust](https://course.rs/appendix/rust-version.html) 中使用。

###### 表达式

表达式会先求值，然后返回一个值。如：`5 + 6`，在求值后，返回值`11`。

表达式可以成为语句的一部分。如：`let a = 1`，`1`是一个表达式，它在求值后返回一个值`1`。

+ 调用一个函数是表达式，因为会返回一个值

  ```rust
  fn main() {
     add(1, 1);
  }
  
  fn add(x: u32, y: u32) {
     println!("x + y = {}", x + y);
  }
  ```

  函数`add()`虽然没显示返回任何值，但会隐式地返回一个**`()`**。

+ 调用宏也是表达式

  ```rust
  println!("hello");
  ```

  调用宏虽然没显示返回任何值，但会隐式地返回一个**`()`**。

+ 用花括号包裹**`{}`**最终返回一个值的语句块也是表达式

  ```rust
  let y = {
      let x = 3;
      // 注意这里不添加";" 
      x + 1
  };
  
  // if 语句也是一个表达式，因此可以用于赋值
  let z = if y > 3 {
      y
  } else {
      1
  };
  ```

  该语句块是表达式的原因是：它的最后一行是表达式，返回了`x + 1`的值，注意`x + 1`不能以分号(;)结尾，否则就会从表达式变成语句，**表达式不能包含分号(;)**。如果在表达式加上分号，就变成语句，不会返回一个值。

+ 表达式如果显示没有返回任何值，会隐式地返回一个**`()`**

  ```rust
  fn main() {
      assert_eq!(ret_unit_type(), ())
  }
  
  fn ret_unit_type() {
      let x = 1;
      // if可用于赋值，也可直接返回
      if x > 1 {}
  }
  ```

  总结，能返回值，就是表达式。

##### 函数

```rust
fn add(i: i32, j: i32) -> i32 {
    i + j
}
```

![函数结构图示](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAh0AAAFSCAYAAABWhm3DAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAH0cSURBVHhe7d0FuOxluf9/rt8JLI5IdzfS3QiCgISAhAoCIggijaDUUbpbUrq7O5XuBikpBZQSPCp6PH7//9fDPJvvHmbWmll7cs39vq659tqT33zuz3PXM14RBEEQBEHQAUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEEQBEHQEUJ0BEFQ/N///V/xm9/8pnj11VcrzwRBELSeEB1BEBR//vOfi7322qvYd999099BEATtIERHEATFO++8U3zta18rll122eL111+vPBsEQdBaQnQEQTBGdMwyyyzFLbfcUjzzzDPFnXfemR5PPfVUCr8EQRCMKyE6epA//elPxZlnnln86le/6tnB3nY99NBDxZFHHlmcfvrpxbPPPlv861//qrwa9AMffPBBEhe//vWv0/U2zzzzFOOPP36x+uqrFxtssEHx1a9+tVhjjTWKn//850mUBLX561//WjzyyCPFhx9+WHmmfzH2PPzww8VHH31UeSYIWkuIjh7kySefLGaaaabiRz/6URrQehEu+I033riYcMIJixlmmKHYcMMNi9tvv7343//938o7BpO33367uOSSS4qDDz44CbLzzz8/JWj2miBzXZ1wwglJXKy44orF3HPPXXz2s59NomPJJZcsNtpoo2KnnXYqDjzwwLQ/7733XuWTjWOfiRqi9Gc/+1lx4oknFldeeeWnBIxtcc1ffPHFxbHHHpuO2wsvvFB5tfdxftdaa63i8ssv73uPEM/WmmuuWdxzzz2VZ4KgtYTo6EGuueaaYrzxxiu++c1vpplHu2AUXnnllREZxUsvvTQJjqWXXrr44Q9/WCywwALFV77yleLaa6/taeFhP994443iggsuSIbw3HPPLX7729/W3H/Pvfzyy2mfiIc77rijePHFF+seK8aUkeYxYMAdn9lnn7341re+lYzt3/72t/S+Zr93XPCbDAhjfsABB6Tf+93vflf8z//8Tzpv00wzTbHSSisV3/nOd4o55pijmHjiiZNIYPR5QuoZ0X/84x/Fo48+WpxyyinpOPKUPPfcc2PtwxNPPJGEKVHqep5yyimLBRdcsNh1113TbNp5cAx83rX+5S9/uZhkkkmKySabLJ2XdhyPdnDXXXel/bNfjks/c84556R9+cUvflF5JghaS4iOHsNAy0C48cXY33rrrcorrccAaXDZdtttm/6dgw46qJh00kmTcfjDH/6QZsPLL798miXfcMMNPSs8GHvGYc4550zHeLbZZis22WSTtP1lr9Lf//734uabby423XTTJKyciy222KLYeeedi8cee+xTBtH/zXQZ8fnmm6/YZZddkjH96U9/mj67wgorFFdddVUSAc18L2zLP//5z/T3u+++m0TESy+9lP7vteuvv7447bTTij/+8Y/puQyvi/PrN4if6aefPgmLH//4x8Xvf//7NKslGhhNHgkCgTAQNhsK1S3O+ze+8Y3kIbHPPHObbbZZEh74y1/+UvzkJz8pJphggmLVVVct9txzz+K///u/i+233z5dI+uuu256btFFF02/KZSz3XbbFXvvvXcSR/fff3/fiQ5eI8eRoMr5MI5zP5FFBy8X4fnAAw+k/bj77rtH5O0KgmpCdLQZA6cb97DDDkt5D8NBCDAKbnyDM09Eu2Bkf/CDHxQzzjhjmrXWwmz0+OOPL2688cYxQiJvo1lrNjKeu/XWW5PxYEzNYM8+++zioosu6pkQEYN/yCGHJA+E40scbL311sVCCy1UzDvvvMkbYR8Zcn/bF2KBIWTUuf3NxoWSXnvttcq3fozvFg4jxIjGXHYqNk4kHH744UmUeDTzvbbHZ3iW/IYQBBHBYDMCt912W7H44osnQcF7kz0T77//frrmiA3v9/vCKauttlraf99Zxu8QY8OJDueSV4Nny3nef//9k3D59re/nUIz9p1AYrC8x7HNYTf3gm2+4oorij322COJFueBZ8Nv6xHSD+EJ+0GkExeueTkv9sM5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OuR2dJvHH388le5ahZP4uO222yqvBL2IJeSJjtlnnz1dRxYQDEaGsIp7cJZZZklL+rsXonNpMK6E6Ggza665ZhIePB1vvvlm5dmgHpYJ32GHHVKfDsaj28gJ4HHZaqutksfq9NNPr7wS9CI8G8JhDCXRoZtsMHKEp1z3PB7bbLNNqg4K4RGMCyE62ozad6IjPB2NceuttyavwuKLL56MRy9wxhlnFLvuumta+fWAAw6Iro09jBJZ180MM8yQRAfPRzBuGLe23HLLYrLJJkv35tNPPx3CIxgxITrajBm7G9YA+M4771SeDeqhO6KKEf0CrDLbC2iBLsSijNB5fO655yqvBL2GRF+iY+qpp04z9N/+9reVV4Jx4eWXXy7WW2+94r/+679SftOzzz4bLeaDERGio80o3ePl2H///Rta2nzQITrkTqy44orpmPUCZsvEBtGx2267FTfddFPllaDXsEAg0THFFFOkcxUN3VqHhmFrr7128cUvfrHYbrvtiueffz6ER9A0ITraCBekzpqbbLJJygSPTPrhITr0WfjqV7+aQhm9wi9+8Yti8803LzbddNPilFNOifV0ehjXzTTTTJOEYiyO11pUsVin5Utf+lJaDVo1VwiPoBlCdLQRfR7M2DXZOfLII6NnQAMQHUIZRIdurr3CLbfcktqhWxRLgmt0uuxd9FaZaaaZUpju9ddfrzwbtAorHxMek08+eapu0fSQhykIGiFERxux1orwiqWjzZRj7ZXh0QGRcSc6DjrooMqz3cfAygOjU6OGb8oHg95EYyvNwYjXCK+0B0sErLXWWsXCCy+cvLgaH4bwCBohREcb+cMf/lAsuuiixde+9rXirLPOitlxAxAdjIWqn0MOOaTybPcRTuF50aVx1VVXLS666KLKK0GvcdJJJyWxT7xKgAzaw5VXXplaAsw222xJeGgJEGHHYDhCdLQRs2MLl8nruPDCC1O4JRia7Okg1LjJewlCQzKpBbFOOOGEmNn1KCeffHJaot11FNUr7cW4xpOrA+xxxx2X+qKE8AiGIkRHG3nhhRdS6ecSSyxRXH755cVf//rXyitBPXT81AuAUDvssMMqz/YGygQZMgOshmGRo9ObaOAmDKa3ikX7gvbieMt1EtKyKKIqvWaEh+ow7QSi98dgEKKjjdx5550pN4Gr97zzzgtPRwNcf/31qQkXD9ER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R9QtW/a8dVp22WWX4plnnqk8GwRBOwjREdRFjwkLkn3zm99MXo92wTPxyiuvpJlms14KhoXgsIS58kiruX7lK18prr322hAeo5QPPvig2HfffVOuxiOPPFJ59tNYUM+S/xqE7b///mnF31roFrvrrrumzrE333xz5dkgCNpBiI6gJoy/pdeJjnptvFuFbpxKGpU3Nvs7Bx10UDHppJOmngzWMuFSX3755Ysll1wyrSwawmN04nwTmkOVwl5xxRVJdMjp2HPPPet664TnCBjrtNx///2VZ4MgaAchOgYEIkJnzMMOOyzlPQwHISDzn+hgwHki2gVjoGvkjDPOWDz66KOVZ8fmjTfeKI4//vjixhtvHCMk8jZa3ySvXeK5W2+9NbUfZ5R4PM4+++xUydCrIaKgeV5++eW0mJ7GX0JstVAy6z2rr756sddeeyUPSS2sy8JDprz2j3/8Y+XZIAjaQYiOAUHCpbLBCSaYIMW4xcWHwgC9wQYbdMTTkUWHbROrJyDuvvvu4s477ywefvjh5Bb3fzkbBNBdd92VRFT+XLUosq+33XZbsdBCCxXrrLNOsfjiixdzzz13WlwtPB+jA3kYxxxzTLHMMsskcVELK8vusMMO6TomOt59993KK5/g2jrggAOKaaedNgnyaFcfBO0lRMeAwBBLsNSdcYUVVkjlhEMNsO+8804SG0QHt3O9WeJIUSXwwgsvJDFhW1ZcccX0W/Ix/N7KK69cfP3rXy922mmnNKu1eNruu++etv2nP/1p2p56ogOMibi/96tcWG211dKs980336y8I+h35F84vwTFRx99VHn2E+QkCa+45r1H+K0a16BE6WWXXTblBwVB0F5CdAwQvAOvvfZamvEz9o2KDiEMq6G2Ct4GQmOjjTZK4mL++edPyaB+i5vbzHT77bdPokFoJHtlJLMq37X9PC/ZxV5LdECohuvcvojV+3sk7nPb++CDDxYnnnhi8bOf/aw45ZRTksH785//XHlH0A2cV9fmWmutVdx3332VZz/huuuuS6LDom+qWKzmW83VV19drLLKKilMUy+0FwRB6wjREaQ8CHkewhkGb8Y0iw5JmmaA1X06iJB77rknJZtyTwuLlAd1r4u154oBosB79dJ4++23U7Mm322GqeRVKIToOPjgg1PZ4nvvvTekKJLbwVAQKbPMMkuqVPB7lpBvxkU+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hbwIxjZXx4xW7BtjyIARDIQwEUNEVIuYofA9jHX1sfL/4e4D3g4eFsmkwnq8PIw1T4L8GB5BXh3ipdFQgu8f6vp0rnkXiAaeGKKLZ8jvCgUJ7/Fg8KyVS19bfY3k7Rzp54P2EKIj6FsMKMMZ56C34GEpz9aDcWe4+4AHQZhGWE04QshIDoSEUkmevGy8Aq28l5xnooHIEdKTm+J3Vb7wZnhe8zoeoWriGhndhOgIgiAY5TDkQhlCIzwuwikSSnn82iXceRiEqoTHhJr8rmRmyazCPOGBGExCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0BFCdARBEARB0AGK4v8Dg+hexg4JMUUAAAAASUVORK5CYII=)

###### 函数要点

+ 函数名和变量名使用**蛇形命名法(snake case)**。如：`fn add_one() -> {}`
+ 函数的位置无限制，只要定义即可。
+ 每个函数参数都需标注类型

###### 函数参数

rust是强类型语言，每一个函数参数都需标注**具体类型**。

```rust
fn main() {
    add(5, 6.1);
}

fn add(i: i32, j) {
    println!("i: {}", i);
    println!("j: {}", j);
}
```

```shell
error: expected one of `:`, `@`, or `|`, found `)`
   --> src\main.rs:251:17
    |
251 | fn add(i: i32, j) {
    |                 ^ expected one of `:`, `@`, or `|`
    |                   // 期待以下符号之一 `:`, `@`, or `|`
    |
    = note: anonymous parameters are removed in the 2018 edition (see RFC 1685)
    // 匿名参数在 Rust 2018 edition 中就已经移除
help: if this is a parameter name, give it a type
// 如果j是一个参数名，请给予它一个类型
    |
251 | fn add(i: i32, j: TypeName) {
    |                 ++++++++++
help: if this is a type, explicitly ignore the parameter name
// 如果j是一个类型，请使用_忽略参数名
    |
251 | fn add(i: i32, _: j) {
```

函数`add()`有两个参数，如缺少其中一个，就会报错。

###### 函数返回

函数的返回值就是函数体最后一条表达式的返回值，也可用`return`提前返回。

```rust
fn main() {
   let x = add_one(1);
   println!("x: {}", x);
   let y = add_or_minus(6);
   println!("y: {}", y);
}

fn add_one(x: i32) -> i32 {
    x + 1
}

fn add_or_minus(x: i32) -> i32 {
    if x > 5 {
        // 使用表达式作为返回值
        return x - 1 
    }
    x + 1
}
```

###### rust中的特殊返回类型

+ 无返回值**`()`**

  隐式返回一个单元类型`()`来表达一个函数没有返回值

  + 函数没有返回值，隐式返回一个`()`

    `println_num()`会隐式返回一个`()`

    ```rust
    fn println_num() {
        println!("x: {}", x);
    }
    ```

    与上面函数返回值相同，下面用显示返回一个`()`

    ```rust
    fn println_num() -> () {
        println!("x: {}", x);
    }
    ```

  + 通过**`;`**结尾的表达式隐式返回一个`()`

    ```rust
    fn add_one(x: i32) -> i32 {
        x + 1;
    }
    ```

    ```shell
    error[E0308]: mismatched types
    // 类型不匹配
       --> src\main.rs:269:23
        |
    269 | fn add_one(x: i32) -> i32 {
        |    -------            ^^^ expected `i32`, found `()`
        |    |						// 期望返回u32,却返回()
        |    implicitly returns `()` as its body has no tail or `return` expression
    270 |     x + 1;
        |          - help: consider removing this semicolon
    ```

    函数`add_one()`期望是返回一个i32类型的值，在rust中只有表达式才能返回值，而`;`结尾的是语句。以`x + 1;`结束的函数体返回的是`()`，造成与期望类型不匹配而报错。

    在rust中一定要严格区分**表达式**和**语句**的区别。

+ 永不返回的发散函数( diverge function )**`！`**

  当使用**`!`**作为函数返回类型的时候，表示该函数永不返回，这种语法往往会导致程序崩溃。

  ```rust
  fn forever() -> ! {
    loop {
      //...
    };
  }
  ```

​		该循环永不跳出，因此函数也永不返回

#### 复合类型(Compound types) 

复合类型可以将多个值组合成一个类型，最典型的就是结构体`struct`和枚举`enum`。

rust有两个原生的复合类型：数组(array)和元组(tuple)。

##### 数组(Array)

数组是由**相同类型**的元素组合而成的复合类型。数组的长度是固定的，一旦声明，其长度不能增长或者缩小。数组是直接保存在**栈**中，所以没有所有权概念。

###### 数组在内存中

![数组](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAWAAAAFlCAYAAADVrDL/AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAFntSURBVHhe7d0HmJ5F9f9/epDeQhfpvfdepUjvvUqAgHRRCPpFQboQkS5ERaogIF1AikpRQHoTbIgFUVBEBBF0/r/XuJP/nXWz+ywkO0nu876uuZ7dp95l5jNnzpw5M1EKgiAIqhACHARBUIkQ4CAIgkqEAAdBEFQiBDgIgqASIcBBEASVCAEOgiCoRAhwEARBJUKAgyAIKhECHARBUIkQ4CAIgkqEAAdBEFQiBDgIgqASIcBBEASVCAEOgiCoRAhwEARBJUKAgyAIKhECHARBUIkQ4CAIgkqEAAdBEFQiBDgIgqASIcBBEASVCAEOgiCoRAhwEARBJUKAgyAIKhECHARBUIkQ4CAIgkqEAAdBEFQiBDgIgqASIcBBEASVCAEOgiCoRAhwEARBJUKAgyAIKhEC3OBf//pXeuGFF9LTTz+d3nrrra5ngyAIxg4hwA3eeOONdPHFF6cvfelL6Wc/+1nXs0EQBGOHEOAGf/7zn7P47rTTTumee+5J//73v7teCSYk/vGPf6SHH344XXnllemyyy7Lne5pp52Wy2233ZZee+21rncGhf/85z95hMhIefnll9Pjjz+eR4p//etfu94RfBBCgBsUAd56663Trbfemt5///2uV4IJiV/96lfpqKOOSossskhaYIEF0mKLLZbLqquumr785S+n3//+913vbDcMkF/+8pfp8ssvz+3ic5/7XDriiCPSvvvum9vI0KFD0w9/+MP03nvvdX0i6C8hwA3GlAD/85//TL/+9a/Tb3/726ic4yA///nP0yc/+ck07bTTptVWWy198YtfTN/85jfTddddl5555pl8/5qw/t55551sOY/PoyJ18e23386FNeu8esP5XnHFFWnFFVdMk08+eZpqqqnSXHPNlYu/F1988XTppZfmaxN8MEKAGzRdEHffffcHbmy/+c1v8nD29NNPzyIcjFsYRp999tlp0UUXzVac+zU6iNRLL72ULrrooiw2r776atcrHx4i+Itf/CJ997vfTWeeeWY65ZRT8m/oBN59992ud314nO/3v//9/P1HH310Lscee2waPnx4uummm/Ix9PR7hFU78N5Pf/rT6dRTT83umjPOOCOtscYaaamllsoC3b3DCjonBLjBH//4xzzM2mWXXdKDDz7Y9Wz/efLJJ9O2226bNttss+xrDMYtdKyGzh//+MfTrrvuml588cWuV/6XV155JYvjQgstlOvFmJqc5Tu95ZZb0v7775+WXXbZNN1006Upp5wyzT///GnPPfdM3/ve99Lf//73rnd/cJzrT37yk/ydXC7LLbdc/r2PfvSjac4550wrrLBC2nvvvbOQqv/dYS3/6U9/Sm+++eZIi1mHZQTBMtZ56EiCD0YIcAPW6kEHHZS22GKLMSLAhrcmep566qn04x//ON1///05zE2lDuriHm2zzTZpq622So899ljXs6PCcvzWt76VhWbSSScdYwLMsrz99tvTpptumoV9++23T8ccc0z67Gc/mzvteeaZJ+2+++75GPtyE/QFAX722WfT17/+9fS1r30t3XDDDVk0WbKHH354HgU4t+222y5PrHVCEWD1O+ZKPhwhwA1Kxeqv5aqSs2ief/759KMf/SgPb1daaaU0ePDgtMEGG+TKvfHGG+cGz5oysfFhG1bw4SBKO+ywQ1p//fWzNdz9fhAVHSZxnGWWWdKgQYPGmAAT9q985St54o8FrnMuPmaugg033HCMWpfqp+/hA3799dfTo48+mifWDj300CzAc8wxR3Yz9OaKaWJ+g9W89tpr5+P1/cEHIwS4QRFgginEplMMU/kI+Y4JrsmJaaaZJs0wwwxp5ZVXzo3M7DG/8M0335xn2ce2AGsULHo+Pj47wv+d73wndxLF3+eRv5Hf0XvOP//8dMcdd+Tj+zCNymfHxPkRJeJErEyUNQtfvY7uqquuSk888US//ZA6wT322CP7Mlmj3c+XABu5XHjhhXlURKjGpAXMt3r88cfnib+//e1vXa/8N0Jjr732Giv+Va4E9XTLLbdMCy+8cJ6EXGKJJdLnP//5bG13Kvbq0M4775w7Cp2Ue81d4pyuvvrqfA5hYHRGCHCDIsCja2gsF64JjZKgXXvttdkaMInBmlCpCTBL199LLrlk+upXv5q/y2c/zFDNZ3/3u9/lSq7R3nnnnaN1Z6j8jsuky/LLL587Az5Gx7PPPvukG2+8MXcafIMHHnhg9jt6z6yzzpqHlcOGDUsPPfRQFgpoXMTI5xSf4y/sqZE5RrG1XC9+o8D6IvZeu+uuu0YRndHB73juuefm45tooolGKYbNM888cxYQQtbf0LE//OEPeWKJH3h0w2jnR5T4Ywn1mPQBu2+u4V/+8peR4u/3fvrTn2aBFBKnsx6T/lXCeMABB2Rfs2s4xRRT5OvH+jXi69Q15r1GiU33jXP5v//7vxxBpBP/MB14mwgBbkCAzYoTqebEjIZhJvyCCy7I/mEW7rzzzpsnNFQ6lZDrgcXiUdltt93SmmuuOUZ8ZBrGAw88kH2EG220UXZn8BFqOMSs+7Jp/xM61jcBZsF94QtfSEceeWTuHFj4OgY+QB2F7/PdzkUD/cQnPpEOOeSQLLSEnL9wxx13TOuss07+vIkj18I1aTY0f+sg+DaHDBmSO4iCxQ1GAMsss0wWPp/tCxb6I488ko+1af06F3G8rrHjcSysu/6gcxANsMkmm/R6j3QcXAFcAmNSgHvCfXbfTIy5R859TAqZDs1CE/faqIz7ZcEFF8x12XU0EtIx9dSxFrx27733pnXXXTdbwaxh+G6GCevdb3zYOt8WQoAbEF0CzJptCoShPPEgqAT4xBNPzCFmm2++ebaMVNym9dBp4+6E4hcU/E4YiRfBcTz+1wjuu+++UeKNrVRyDjoIlrrj0XBYnSx4nydkrHWFm6LEuGqALK8RI0Zk4TnppJOyQHOvsKgNYV0jYmzRQtPKJZiGoCw4Lg3+xgKXjmMl+MKfhPx9UFiFOkvnzX/rXvX3GnNB6GidB4ttdJ/3WwMlwOKTjUiMVFzrMRnyVlBPWN06R9eO4B933HHZciXI6rX6MzpcJ/WD/1dHrIOGuuPacOc4jzHZcUzIhAA3MMw2uUDkypDW8FslZUkq/KSGcqwTQsTKJGjNkKExtaBDJTYz7XeImuNg6fm+EkZlGG4GvTkELxNM6623Xo9LqvkViYoG96lPfSqfTxNiTTydK4FmKbPyfY4lqwOabbbZ0lprrZUbXBF/ndA3vvGNfN6iB4oLw+e4TYgdIefe6OSaOG6uG2JLwLldLJg4+eSTs4VOrNwrx0NEvbdTuENcI+fnWo5OMIoAr7766mm//fb7n2v1QfG93DXqkY6EValT1Wka3jumZqc6tnCv1V2jHZ22OiFaYnQuIvfYPTCSaQpw8MEIAW5gIsIQqgiwykmUWUqWrBJbjZZomR1feumls1XH5dBsLEWANSSNV2PTwDVe4tNTvGVPEDCiy+pmmZTPEUcTU8U3yjLnp/Q78BvcACzXnsLpCKJZcH7GZmfThDVDcJyn8/PdRFNDFWLHf0j8WfqEBKxojVdnYVTg+F1DIwuC6Ti5MzrNH8C69n5+ecdhtGG4bBVW8QXzZ5qwYo2z3jvF5J4RyuiuUaF5rbhwyrVyXu75BxFJ569jPuyww/IoSkdqSF8mxlZZZZV8v9W97gsk3INSn8Ykvldnb6SlDfBF9/Qb6qBJO/de3W+OCFwL16uTzjX4LyHADQiwSlV6dpX/mmuuyY3Uc4bOrC6CyGfG8uQe6G4tEEhDSJ8rM9msDJaDaAgWTycNiF/tnHPOyUIhjtP/RIZ1SdwJmploAsR3q8Gq/D/4wQ+y9ctf3JO4FKFkxTiengSYZUZo+Qp9L8vHc6wkIXa+24jApJ3jISoEl/CyoviTWa6Gs8TRMfLZusadigfh1gl87GMfy+fq86xxAsAPzI3iOuukREI4r07wPsdsqO9c3COdpudZ+K6x+0Vky7WyeKF0VoTG7xn5EFL3pVOcu0ksPnWCq2NxjXWmOpbZZ589zTjjjHmhhCgNw/3SYTk2IwmdEuHrjwiXzsI5jQ4jCC4rHaiImdKh+x112Os6enXOJCgXmI6sXCf+f6vlTNR2MskahACPAnEgOhqFIa9KxNIkZhobYeUPZh1ogNwBPVlARTh9TqiUBmTIS8RZ0EJ3Omk8Kr0OwNBdpf/MZz6Tv4P/zTCfZcbFwEXBGmdlEklDdb9dQqy6WyRFVMShcrn0tBKMH8/3EmlWEUvXrDfBMgFmiEwICCPrTQdBnFjLOiifI55DhgwZOWlJiPkfO0WDL24Hk42O07Xs6Zr3BwJLTEV+zDTTTCNdMSanXEOdmftO5AhxdwHW0Tgm564j6M8EoHthkopLwwSYCVLX2TXi0jFy0qlw1YhQ0PkaRZlY5Vrih+Y2IXadCrDOk4vjrLPOyp2NDlXdLp8noP5Xl3RyDAe/qT04d52pyBidteNh/Rp5MEKKALOMTQq7z6MbVQX/SwhwAxMzGoNKpuETACLD9aCBEoDeLIgC4RSfqpERQhXSsE6DZbWVIXtf+C1CyOKzjFRcsUZLNE2cETNWOkFnLbHo/BbrhSWvIbHQuwuwz1x//fVZ2Am63+gOkWbJGCKzyBTHz//63HPPZeuIi0CEAkuYCPtdFq+JOsPopquApWzpbbGqasLa5W91vYTnlWP099xzz50fdTREiODzhevgirCoFwTaOZ933nn9toAJu3vqurF0FW4Iok7M1R/XmBvCb+gYXNfiWupt9V5PlONVhxQRFtwphLVElRBXoxrXxO9xRxBf73dtSiIe8dDuLQu+HC90DiJzfJ6xEhZwZ4QAN2BlGPJpnMK+NAQWJTEzhFZRVTQCRhy9X0MxLDSkZTGwzjQyFjKhJFwmrFitPq9i90eEvNeMtbhajchwm3VeJrjAwuGbZW2X4zA0Jog9ib1j5x4QsUCgmxOITQh8sZwU72XpNC0n36+zMXRlFbsm3lOuo2vA0tYBuVbjAq6p8CnXx3ETIcXfOlyPxMe91hG5lgcffHAWHGJWokx0SEYo3f20faFDNB/g+rN23VfXVsfQ7OBFQehoWcz+1iH6zf5OBjo+bg8jEIYE8dQ5MxBY3orv9TzRV3+cp2My6mKUnHDCCfnauNfcEFbTleN1PiVWWkfS26RmMCohwN0gICw7PbjKRZysn+fvUlENDQ3HiKnKqdKyZLyusZTGqOEajgvH4o4wZDT07W6Njikca/HzQQPwW80G3R2vj4nj8Vul42lC6Fw7Lh3+8/64H8YVXD9CQ4zdP+eoE+UCGsiluOohy9JoTMKoMiHbKc6DQcFidS5cByJHdAKlI/Z8qffQyfsdpdnhd6ccG+uXy6mnEVXQMyHAHaBS8kGW2F4VzVDMsJpPl0+UP9bEXbMxdhfFtqGxs5xcI419QrgO7i9fKdeAiVBugd46uTGFa8lKF4/MYmahjiuYHBQvzk/Oh13cEkHfhAB3CEvRMJDlYOhqKEZUWA38hANhBY1PECXDVH5BkSM9TfSNj+hEjHQM2YXHDZS1VzL18U2re6zZcQUjG+4Nk69cZUZ/QWeEAAdjBX5p/mpRJYan/ZmoGpfRERuu61iITnO139iEG0CUgSgI8xPjUofPPWEOgo/aRHYYI50TAhyMFSwPNtNuArPTsLvxBdYoERbONVBRHUSOu8NcAp90MGEQAhyMcViJ/KR8pNwPY2r5bhBMaIQAB2McPkDJfPhJTRzFpEwQ9EwIcDDG4f+14oqf1NJkoX1BEPwvIcDBGIe/12SMRRyiBMZErHEQTIiEAAdBEFQiBDgIgqASIcBBEASVCAEOgiCoRAhwEARBJUKAgyAIKhECHARBUIkQ4CAIgkqEAAdBEFQiBDgIgqASIcBBEASVCAEOgiCoRAhwEARBJUKAgyAIKhECHARBUIkQ4CAIgkqEAAdBEFQiBDgIgqASIcBBEASVCAEOgiCoRAhwEARBJUKAgyAIKhECHARBUIkQ4CAIgkqEAAdBEFQiBDgIgqASIcBBEASVCAEOgiCoRAhwEARBJUKAgyAIKhECHARBUIkQ4CAIgkqEAAdBEFQiBDgIgqASIcBBEASVCAEOgiCoRAhwEARBJUKAgyAIKhECHARBUIkQ4CAIgkqEAAdBEFQiBDgIgqASIcBBEASVCAEOgiCoRAhwEARBJUKAgyAIKhECHARBUIkQ4CAIxnv+/e9/p3/961/pnXfeSW+99Vb629/+lv7yl7+kP//5z+mVV15Jv/vd79JvfvOb9Ktf/Sr9/Oc/Tz/72c/ya7UJAZ4A+c9//pMrpPL+++/n8t577+UKqrz77ru5/POf/8wVVnn77bdz+cc//pErsPL3v/89lzfffDNXaOWNN97I5a9//Wuu4Mrrr7+eXnvttVxUauVPf/pTevXVV3P54x//mIuG8Ic//CGX3//+97loGL/97W9zefnll3PRUF566aVcfv3rX+ei4fzyl7/M5Re/+EUuGtKLL76YywsvvJCLhvXggw/m8txzz6Vnn302l2eeeSaXp59+OpennnoqlyeffHJkeeKJJ3J5/PHHR5bHHntsZHn00UfTT3/60/Twww+nn/zkJ/k37r///vSjH/0o/eAHP0j33HNPuuuuu9L3v//9dMcdd6Tvfe976dZbb00333xzuvHGG9MNN9yQrr/++nTdddel73znO+nqq69OV111VbriiivS5Zdfni699NL0la98JQ0fPjx9/etfTyNGjEgXXXRR+trXvpYuuOCCdN5556Vzzz03nXPOOemrX/1qfu+ZZ56ZzjjjjPTlL385nXbaaemUU07J5aSTTkpf+tKX0vHHH5+++MUvpi984Qvp//7v/9LnP//59LnPfS4NGzYsHX300emzn/1sOuqoo9KnP/3pdOSRR6bDDz88HXbYYemQQw5Jn/rUp9JBBx2UDjzwwHTAAQek/fffP+23335p3333TXvvvXfac8890+6775522WWXtNNOO6UddtghbbfddmnrrbdOW2yxRdpss83SJptskjbaaKO0wQYbpPXWWy+tvfbaaY011kirrbZaWnnlldOKK66YlltuubTM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tnFjEATBQGAVrp2+995777zxbC2qC7BtuglwTUd4EATt4oILLsgWsJVwMqPVoroAL7300lmAX3zxxa5ngyAIxi7nnHNOFuBjjjkmPfTQQ13PDjwhwEEQtI6vfOUrWYCHDRuWHnzwwa5nB57qArzMMsvkbGgvvPBC17NBEARjF6lwRUHIB3zfffd1PTvwVBfgZZddNlvANWPxgiBoFyeffHIWYNrT6km4EOAgCAaaE044Ia+EsxnE3Xff3fXswFNdgJdbbrnsgnj++ee7ng2CIBh7/Pvf/05f/OIXczpKj1JT1mKcEeDnnnuu69kgCIKxx7vvvpuFd6uttsrb0ktNWYvqArzCCitkAY4dMYIgGAjeeuutLMA77bRTfpSashYhwEEQtIo33ngjC++ee+6ZH2+88cauVwaeqgL8pS99KX3kIx9JgwcPTgsttFCOCRaWxi1hs84VV1wxrbTSSmmVVVZJq622Wlp99dVzsYSwlDXXXDOXtdZaa2RZe+21c1lnnXVGlnXXXTcXsX+lmAUtpbfnymeV5neW32n+djme5jGW43YOpay66qoji/NTVl555ZHv9bdzbxbXo1l0Xs3imjWL69gsJjybxbVuFte/WaxSbBbLxptFIqVmkdmuFGlGm0XifaX8lud8xvf4br/nmLrfe9ehef9dz3KPy311r0yofPzjH8+7HPDtbbLJJnnTxc022yxtvvnmaYsttshDzq233jpts802abvttsu74u6www5pxx13TDvvvHPaZZdd0q677pp23333tMcee+QGaqnqPvvskz75yU+mfffdN+23335p//33TwcccEA68MAD00EHHZQ+9alPjfzMYYcdlo444oh05JFH5kQvcg3Y+FG8KUND5j/D3uOPPz7X/5NOOimdcsop6fTTT88GifjUr371q+m8887Lq7W+9rWvpa9//es5d8Fll12WrrjiinT11Ven6667Lue0Zb0ZQvNjmkyy2aQdfy0uePTRR9OTTz6ZjRthnvKt2HnGLsB/+tOf8maUf//739M777yT3n///a5WOeFjDzjC6z56dC1rUVWAVbo555wzzT333OljH/tYmm+++fLfs88+e5plllnSDDPMkKaZZpo0aNCgNOmkk6ZJJpkkTT755Fm0p5122jTjjDNm8Z5jjjnSRz/60fx5Qq5xa9Qas0as8Wq0GqtGqnFqlBqj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)

###### 数组的声明

rust中数组类型签名为：**`[T; N]`**。**T**：数据类型，**N**：数组长度。编译时必须确定其大小。由着可得出数组的三要素：

+ 长度固定
+ 元素类型一致
+ 依次线性排列

**两种初始化方式**：

1. 中括号`[]`中列举所有值，值以 " **,** " 隔开。

   ```rust
   // 未声明类型、长度
   let a = [1, 2, 3, 4];
   // 声明类型、长度
   let b: [i32; 4] = [1, 2, 3, 4];
   ```

2. 中括号`[]`中，第一个值表示默认初始元素，第二个值表示长度，中间用 " **;** " 隔开。

   ```rust
   // 未声明类型、长度， 默认初始元素为2，长度为5
   let a = [2; 5];
   // 声明类型，长度，默认初始元素为2，长度为5
   let b: [i32; 5] = [2; 5];
   ```

   ```shell
   a: [2, 2, 2, 2, 2]，b: [2, 2, 2, 2, 2]
   ```

**注意：`[i32; 2]` 和 `[i32; 3]` 是不同的数组类型**

```rust
fn display_arr(arr: [i32; 3]) {
    println!("{:?}", arr);
}

let arr1: [i32; 3] = [1, 2, 3];
let arr2: [i32; 2] = [1, 3];

display_arr(arr1);
display_arr(arr2);
```

```shell
error[E0308]: mismatched types// 类型不匹配
  --> src/main.rs:10:17
   |
10 |     display_arr(arr2);
   |                 ^^^^ expected an array with a fixed size of 3 elements, found one with 2 elements// 期望一个长度为3的数组，却发现一个长度为2的
```

###### 数组访问

由于数组时依次线性排列，可以通过"**数组名[索引]**"来访问数组中相应位置元素。索引值从**0**开始。

```rust
let a = [1, 2, 3, 5];
println!("a[1]: {}", a[1]);
```

```shell
a[1]: 2
```

###### 数组越界

数组长度使用**`len()`**获取。

```rust
let a = [1, 2, 3, 5];
println!("a数组长度: {}", a.len());
```

```shell
a数组长度: 4
```

数组的长度是固定，访问的索引值超出数组长度就会发生数组越界(Index Out of Bounds)。

```rust
let a = [1, 2, 3, 5];
// a的数组长度为4，访问a[4]发生数据越界
println!("{}", a[4]);
```

检查访问索引值是否在数组长度范围内只能在运行时进行，无法在编译时进行。

```shell
error: this operation will panic at runtime
   --> src\main.rs:278:20
    |
278 |     println!("{}", a[4]);
    |                    ^^^^ index out of bounds: the length is 4 but the index is 4
```

##### 元组(Tuple)

元组是由一个或多个类型的元素组合而成的复合类型。在使用过程中，元组通常至少包含两个元素，但只包含一个元素或没有元素也是可以的。空元组表示为**`()`**，不占用内存空间。元组是直接保存在**栈**中，所以没有所有权概念。

###### 元组声明

元组是一种**异构有限**的序列。异构：元组内的元素可以是不同类型的；有限：元组的长度是**固定**的。

元组的签名格式为：**`(T, U, M)`**，**T, U, M**均表示类型。

```rust
let tup: (i32, f64, bool) = (1, 1.2, true);
println!("tup: {:?}", tup);
```

```shell
tup: (1, 1.2, true)
```

+ 一个元素的元组，需要在元素后添加**" , "**

  ```rust
  let tup_single: (i32, ) = (5, );
  println!("tup_single: {:?}", tup_single);
  ```

  ```shell
  tup_single: (5,)
  ```

+ 以下两种方式不是一个元组，它们等价于`let tup_wrong = 5;`

  ```rust
  let tup_wrong = 5;
  let tup_wrong_1 = (5);
  let tup_wrong_2: (i32) = (5);
  
  assert_eq!(tup_wrong, tup_wrong_1);
  assert_eq!(tup_wrong, tup_wrong_2);
  ```

  ```rust
  let tup_single: (i32, ) = (5, );
  
  let tup_wrong = 5;
  
  assert_eq!(tup_wrong, tup_single);
  ```

  ```shell
  error[E0277]: can't compare `{integer}` with `(i32,)`
     --> src\main.rs:283:5
      |
  283 |     assert_eq!(tup_wrong, tup_single);
      |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ no implementation for `{integer} == (i32,)`
  ```

+ 空元组

  ```rust
  let empty_tup = ();
  println!("empty_tup: {:?}", empty_tup);
  ```

  ```shell
  empty_tup: ()
  ```

###### 元组索引

元组默认通过 **元组名.索引** 来访问元组中相应位置的元素。

```rust
let tup: (i32, bool, u32) = (-1, false, 1);
println!("{}, {}, {}", tup.0, tup.1, tup.2);
```

```shell
-1, false, 1
```

###### 元组解构(Destructure)

元组是一个单独的复合元素，除了通过索引获取元组内某个元素，还可通过**模式匹配(Pattern Matching)**来解构元组。通常使用`let + 模式`进行解构元组。

```rust
let tup: (i32, bool, u32) = (-1, false, 1);
let (a, b, c) = tup;
println!("{}, {}, {}", a, b, c);

let (.., d) = tup;
println!("{}", d);
```

```shell
-1, false, 1
1
```

##### 动态数组(向量：Vector)

###### 动态数组的定义

一种动态的可变数组，可在运行时增长或缩短数组的长度。其签名形式**`Vec<T>`**，**T**：任意类型。`Vec<T>`：表示T类型的动态数组。

动态数组的元素保存在**堆**上，这也是它可以动态增长或缩短长度的原因。

动态数组中每个元素都分配有唯一的索引值，索引从0开始。

###### 动态数组在内存中

![动态数组](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAkkAAAEHCAYAAAC6OtK1AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAIOFSURBVHhe7d0HlDVVlS9w33qz5o3xjeMYxxnFnFCUkWQARBAwgKAiooIKSlZAJekAiqCjBEFREAwgBlARQXREREEcBclBJQgoQVTMiGCo17/zen9zvN7++Lrprq66tf9r7XW7b6g6derU2f+z07lLk0gkEolEIpH4GyRJSiQSiUQikRiDJEmJRCKRSCQSY5AkKZFIJBKJRGIMkiQlEolEIpFIjEGSpEQikUgkEokxSJKUSCwQ/vKXvzR//vOfp/9LJBKJRN+QJCmRWCD89Kc/bS655JLmj3/84/Q7iUQikegTkiQlEguEj3/8481aa63V3HjjjdPvJPqAX/3qV83ll19e5KKLLmq++93vNldccUVz++23T39jMvGnP/2pufXWW5vf/OY3zS233FIsoYnE0JEkKZFYICBJT3rSk5rvf//70+8kugyu0Ztvvrn59Kc/3ayyyiqF4D71qU9tHvnIRzZbb711c9NNN01/c3Lwhz/8oYzPc845p/n617/eHH/88c3hhx/efOYznylkKZEYOpIkJRILhLmSJMrp2muvTTddy/j5z3/evOc972n23Xff5h/+4R+addddt/nYxz7WfPvb325+9KMf/ZVlBaH62c9+1vzud7+bfqdbYPX67W9/W+S2226b0Sr0ve99r3na057W3Pe+923+5V/+pVluueWae93rXs0KK6zQ/OAHP5j+ViIxXCRJSiQWCEjSiiuu2Fx55ZXT7ywbvvzlLzcvetGLimJOtIczzjijWX755ZsvfOEL5b5ttdVWxe00CoTjhz/8YfPWt761WF/uDH7/+98311xzzRK3nrEy7pzLCm378Y9/3HzpS19qPvShDxU55ZRTmnPPPbeMJ4SpxoUXXti86lWvavbff//mox/9aHPyySc3z3/+85uVV165XGMiMXQkSUokFgiUztprr10CuGcD5Ooud7lL89WvfrVYAliWvIoZSSwMWIYOPPDA5nnPe16x4m2++ebNpptu2vz617+e/sb/wP18wxveUO6RezVXiH367Gc/27z4xS8ulpt//ud/Li6+4447bs4WKhYklrBHPepRxYpJHv3oRzf/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L2te8pKXNBtvvHGz4YYbNs9//vOb5z73uc26667brLPOOs2zn/3sZq211mqe9axnldcQ78dn48Rna6+9djmGY6233nrN+uuv3zzvec9rXvCCFzQbbLBB+d/fL3rRi5oXv/jFzSabbNK89KUvLW16+ctfXtq3xRZbNK961auaLbfcsrT7da97XbPttts222+/fbPjjjuWa9p5553L9fl/l112Kde75557FnnLW97SvPWtb23+4z/+o9lrr72K7L333s0+++zTvO1tb1sib3/725t99923ecc73rFE9ttvvyL777//X8muu+5ajvnOd76zyLve9a5llvgNGT1unC/k9a9/ffPmN7+5tEv7tFO74zpck3a4Rtfqunffffdmt912K230W6Jv9It+IjvttFPpN8fXZzvssEPpT2J86N9tttmm2XrrrYvoc/fkFa94RbPVVluVe/Ga17ymjCH3xj0im2++ebPpppsuuX9ks802K++5r+6vseZeE/fduNtoo42aF77whWX8GRfEuDAWjRfjkRgvMY68Puc5zylijJH4P8S4GxW/q2Xce3ckcSxt1D5/1+et21MfX7tdh3tz5JFHlvm8DbROkkw8mDyGX8thhx3WvO9972sOOeSQ5uCDD24OOuig5oADDmje/e53l4fDQ+DBi8HuQTXQlzbIY6Ab5G984xuLzDTYY8DHoPc6OuBNMgZ5DPR6kBvgJiUPgoFdD26D3QA3sN1sA9oAn2+JByfEQ+Mhis9GPw+Jz+Lz0e+O+6z+P67H3x7ikHiwvYbokxCTAYn+IjE5EH0Zom9J9LNXEhNMiPthcvAb94eYkELct1rifhL3N8TEFhKTXYjxEGJ8hBgvxPteTZrO4e8YWzHWiLFnDIYYk8ZnLTFujWFiPBvXxjgx3kkoNs/CqGILpRaKLRRaKLVQZKGw/vM//7M8d+95z3vKM+j9Aw88sDyXVuee0UMPPbQ8r+9///vLe55fz/ERRxzRfOhDHyqT2FFHHVUsBp53iwYksC9AXlkQ3D9KyJgzfowH99X9ci/0uf7Vj/pMH+kTffDRj360WDHCevG5z32uWG1YLuKVeJ9Vg3XD91g8WD5YQ1h3HEc/6lP9G/OlezBuvnT/gkS4t+NIgrExShJiTMX8Wc+dMW/Wc2Y9X46ShJg3iWemfp7iuauft6VJ/VtSP4va4fzxLMYzqC0+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wFJOPElSIjE8WP3JSLntttum3+kutBNJ+uxnPzv9TiKR6AL23XffsuCaSJJk914b0yVJSiSGB3EEJ598csl06zqY8pGkz33uc9PvJBKJLuDtb397cd1PJElSTnzDDTdsrrjiiul3EonEULDWWms1n//855tbb711+p3uQiwSq3eSpESiW0CSZMpOZAkARaA22mijJEmJxAChgrUYH4Ulu44TTzyx2XjjjZsTTjhh+p1EItEFvO1tb5tckrT//vuXiefKK6+cfqc93HLLLc2NN95Y5IYbbmiuv/765rrrrmt+/OMfN9dcc01z1VVXNVdffXXzwx/+sLQPkeMWVB3chrzk0ksvbS655JLm4osvbi666KLmwgsvbC644IKyaed5551X9qX67ne/W7ZeOPvss0t1YXtAKaCnOJ2gVdk90qCJ94iaLMRnRPrxaaedVjJs/CZeiYBS8l//9V8lboJwDRDmR+Jzv+HasL2CCd8K/gtf+MJSxXdD/JYYiI4Z/zuv40e7tFN7td+1nHnmmc1ZZ51Vrtn16wd9on/0l77Tj2pl6V/9rv/1uWtQl8bxop9H+3qm/o4+tzFp9Hv0vfa4fu31v/ejbX7nGI7luI7vXM4ZbXXvjQNtlplpbGivthsvxs21117b/OhHP1oixpYxZqz95Cc/aW666aayE/7NN9/c/OIXv2h+9atfNb/5zW/Kdh1IAxeUgGbp8X3YtmMusCWR4oyexa7D86AEgHGTSCS6AaVEJpokiUo38bRNkpxPte//+I//aPbaa68i/t5zzz2bPfbYo9ltt92aN73pTc2b3/zm8vrGN76x2WWXXZqdd9652XHHHZvtt9++vO6www7l7+22267Zdtttm2222abZeuutm9e97nVFttpqq2bLLbcs8upXv7p51ate1WyxxRbN5ptvXnYUV/6AJW2zzTZbIi972cuaTTfdtMhLX/rSIptsskn5vpgI39dn/iZIpveIbRO4L0PEe3ER+K3jySR83vOeV6qcW8X7WxuWJr4T4veO57jqW9kvxzm1gWiXc2mv87mWl7/85c0rX/nKcs2u/zWveU3pF/2jv/Sdfnz9619f+ldf77rrruU8T33qU5sVVlihtNXv9Z1+1J+O8drXvnbJcfR/HMu9cbyddtqpHNO9i/vp2O6vY73hDW9o3vKWtzRvfetbl4yFvffeu9lnn32K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)

指针(ptr)：指向堆上存储的动态数组第一个元素的指针

容量(capacity)：动态数组允许存储元素的个数

长度(length)：动态数组当前实际存储的元素个数

###### 动态数组的创建

常见的创建方式有3种：

1. 使用`Vec::new()`创建一个空的动态数组，容量为`0`。**长度为`0`的动态数组在堆上是不占内存的。**通常创建动态数组时用`mut`修饰，以便动态增删元素，**需要指定类型**。

   ```rust
   let mut vec_empty: Vec<i32> = Vec::new();
   ```

2. [推荐]初始化时为动态数组指定容量(**容量不等于长度**)。需指定类型。

   如：指定一个容量为10的空的动态数组(长度为0)

   ```rust
   let mut vec_capacity: Vec<i32> = Vec::with_capacity(5);
   ```

3. [推荐]使用"宏"创建动态数组。这种方式的创建方法类似于数组的语法。有三种方式：

   1. 创建空的动态数组，需指定类型

      ```rust
      let mut vec_empty: Vec<i32> = vec![];
      ```

   2. 创建带有默认元素的动态数组，支持类型推断

      ```rust
      let mut vec_marco = vec![1, 2, 3, 4];
      ```

   3. 创建指定长度且初始化所有元素的动态数组，支持类型推断

      ```rust
      let mut vec_maro = vec![0; 5];
      ```

      ```shell
      [0, 0, 0, 0, 0]
      ```

###### 添加元素

使用**`push()`**方法向动态数组的尾部添加新元素(必须于动态数组元素类型一致)。添加元素需要将向量使用`mut`关键字修饰。

```rust
let mut vec_maro = vec![0; 5];
println!("{:?}", vec_maro);
vec_maro.push(1);
println!("{:?}", vec_maro);
```

```shell
[0, 0, 0, 0, 0]
[0, 0, 0, 0, 0, 1]
```

###### 修改元素

修改元素直接使用**`动态数组名[索引] = 修改的值`**即可重新为元素赋值。修改元素需要将向量使用`mut`关键字修饰。

```rust
let mut vec_maro = vec![0; 5];
println!("{:?}", vec_maro);
vec_maro[2] = 1;
println!("{:?}", vec_maro);
```

```shell
[0, 0, 0, 0, 0]
[0, 0, 1, 0, 0]
```

###### 删除元素

删除元素动态数组的元素有两种方式。删除元素需要将向量使用`mut`关键字修饰。

1. 通过**`pop()`**方法弹出队尾元素。调用`pop()`方法会返回一个`option`枚举类型。如果动态数组不为空，则返回**`some(v)`**，**v**：弹出的值。如果动态数组为空，则返回**`none`**。

   ```rust
   let mut vec_maro = vec![1, 2, 3, 4, 5];
   println!("{:?}", vec_maro);
   let value_pop = vec_maro.pop();
   println!("vec_maro: {:?}, value_pop: {:?}", vec_maro, value_pop);
   
   let mut vec_empty: Vec<i32> = vec![];
   println!("vec_empty.pop(): {:?}", vec_empty.pop());
   ```

   ```shell
   [1, 2, 3, 4, 5]
   vec_maro: [1, 2, 3, 4], value_pop: Some(5)
   vec_empty.pop(): None
   ```

2. 通过**`remove()`**删除元素，需**传入将要删除元素的索引**，并返回删除的元素。这个操作会引发动态数组中的元素的移位，被删除元素的后面元素都会向前移动一位。

   ```rust
   let mut vec_remove = vec![1, 2, 3, 4];
   println!("vec_remove: {:?}", vec_remove);
   let remove_element = vec_remove.remove(2);
   println!("remove vec_remove[2]: {}, after vec_remove: {:?}", remove_element, vec_remove);
   ```

   ```shell
   vec_remove: [1, 2, 3, 4]
   remove vec_remove[2]: 3, after vec_remove: [1, 2, 4]
   ```

   **如传入索引超出动态数组的长度，则会产生程序错误**。

   ```rust
   let mut vec_remove = vec![1, 2, 3, 4];
   let remove_element = vec_remove.remove(4);
   println!("remove_element: {}", remove_element);
   ```

   ```shell
   thread 'main' panicked at 'removal index (is 4) should be < len (is 4)', src\main.rs:278:37
   note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
   ```

###### 访问元素

rust中访问元素两种方法

1. 使用**`动态数组名[索引]`**访问指定元素

   ```rust
   let vec = vec![1, 2, 3, 4];
   println!("vec[1]: {}", vec[1]);	
   ```

   ```she
   vec[1]: 2
   ```

   **索引越界：如果传入的索引大于动态数组的长度，则会产生程序错误**。

   ```rust
   let vec = vec![1, 2, 3, 4];
   println!("vec[4]: {}", vec[4]);	
   ```

   ```shell
   thread 'main' panicked at 'index out of bounds: the len is 4 but the index is 4', src\main.rs:284:28
   note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
   ```

2. 使用**`get(index)`**访问指定元素，**index**：索引。其返回的是`optin`枚举·类型 **`Some(vlaue)`**，**vlaue**：指定元素。

   ```rust
   let vec = vec![1, 2, 3, 4];
   println!("vec.get(1): {:?}", vec.get(1));
   println!("vec[1]: {}", vec.get(1).unwrap());
   ```

   ```shell
   vec.get(1): Some(2)
   vec[1]: 2
   ```

   **索引越界：如果传入的索引大于动态数组的长度，不会产生错误，会返回`None`。**

   ```rust
   let vec = vec![1, 2, 3, 4];
   println!("vec.get(4): {:?}", vec.get(4));
   ```

   ```shell
   vec.get(4): None
   ```

###### 动态数组容量增长策略

动态数组的**容量**(Capacity)是指储存元素所分配的空间。当动态数组的长度(Length)如果**大于**其当前的容量，则会重新分配空间(会在原有基础上 **x 2**(目前2是动态数组的增长因子))。

###### 获取动态数组的长度、容量

用**`len()`**获取动态数组的长度，**`capacity()`**获取动态数组的容度

```rust
let mut vec = vec![1, 2, 3, 4];
println!("vec.len: {}", vec.len());
println!("vec.capacity: {}", vec.capacity());

vec.push(5);
println!("vec.len: {}", vec.len());
println!("vec.capacity: {}", vec.capacity());
```

```shell
vec.len: 4
vec.capacity: 4
vec.len: 5
vec.capacity: 8
```

**首次填充元素的默认容量**

首次创建空容量的向量，向其中添加一个元素后，则会将容量变调整为**`4`**。之后的容量调整则遵循增长因子为2的规则。

```rust
let mut vec: Vec<i32> = Vec::new();
println!("vec.len: {}", vec.len());
println!("vec.capacity: {}", vec.capacity());

vec.push(1);
println!("vec.len: {}", vec.len());
println!("vec.capacity: {}", vec.capacity());
```

```shell
vec.len: 0
vec.capacity: 0
vec.len: 1
vec.capacity: 4
```

###### 数组(Array)与动态数组(Vector)的区别

+ 签名

  数组：`[T;N]`；

  动态数组：`vec<T>`

+ 大小

  数组的大小是在**编译时**确定的常量，也是类型自身的一部分，其大小不能更改。

  动态数组是在**运行时**才会确定长度，可以动态分配增长或缩减数组。

+ 元素的存储

  数组的元素可以在**栈上**存储

  动态数组的元素**只能**在**堆上**分配

##### 切片(slice)

###### 切片的定义

**切片是对动态数组或数组中部分元素序列的引用**，只要在数组和动态数组名称前添加**`&`**，即可变为**切片引用**。签名为：**`&[T]`**和**`&mut [T]`**，分别为**T类型的共享切片**和**T类型的可修改切片**。

```rust
let vec = vec![1, 3, 5];
let arr = [0, 2, 4];

let vec_slice: &[i32] = &vec;
let arr_slice: &[i32] = &arr;

println!("vec_slice: {:?}, arr_slice: {:?}", vec_slice, arr_slice);
```

```shell
vec_slice: [1, 3, 5], arr_slice: [0, 2, 4]
```

###### 切片在内存中的表现

![切片](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAiIAAAH8CAYAAADsaMToAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAP+lSURBVHhe7N13mFXV2Tf+54/3eX/vU9OrJsbee28pxlgTY0zsscQWe1dU7Iq9996wK/beK1ZUegcRUFBABBRprt/5LGeR4zjADMwwe7PX97ru65yzzzm773V/113/LWRkZGRkZGRktBMyEcnIyMjIyMhoN2QikpGRkZGRkdFuyEQkIyMjIyMjo92QiUhGRkZGRkZGuyETkYyMjIyMjIx2QyYiGRkZGRkZGe2GTEQyMjIyMjIy2g2ZiGRkZGRkZGS0GzIRycjIyMjIyGg3ZCKSkZGRkZGR0W7IRCQjIyMjIyOj3ZCJSEZGRkZGRka7IRORjIyMjIyMjHZDJiIZGRXBjBkzopQNzz//fDjppJNChw4dwrHHHhs6duwYTjjhhLjslFNOCaeddlro1KlTOOOMM8JZZ50VzjnnnHDeeeeFCy64IFx00UXhkksuCWeffXZ8veKKK8KVV14Z5aqrrgrXXHNNlGuvvTZcd9114frrr49y4403RrnpppvCzTffHG655ZbQuXPnKLfeemu47bbbwu233x7uuOOOcOedd4a77ror/s/nu+++O9xzzz3NknvvvfcbYt2Wd+nS5Rty3333NSn333//t+SBBx6I4n/p84MPPjhLeeihh74hDz/88LfEOrw+8sgj4dFHH43y2GOPRXn88cdnyhNPPDFTnnzyyShPPfXUzNf65cT5ss70uf7/SerX31jSPhDHax/T/lm347P+eqk/riSNz0E6N+lcJml8rt07rr/7IV031zFdX/dCEveI3xL7lsR95P/E9U/3mXuOuP/ch+mevPrqq+N9XH+/ek/cx0nsm98S9zpx3/svufzyy6Ncdtll4dJLL43PB7n44oujeHYuvPDC+Bydf/758Zk699xz4/PlefKsuT4TJ05seFLnHpmINBNfffVVFAP59OnTo0ybNi1MnTo1ypQpU8Lnn38evvzyyyiTJ09uUr744otviP/MSiZNmvQNccGbkgkTJnxLPvvssyZl/Pjx35BPP/10ljJu3LhvyNixY2fKkCFD4uuYMWO+IZ988kmUjz/+OIwePTrKRx99FOXDDz8MI0eOjDJixIgw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nTq1YWnroK2JyIXXvx9OvGBgw5LmY7/99osWoDISkTvuuCMTkYyMAiERkULHiFSdiDz33HMxayYTkWLiyy+/DKuttlo47rjj4vvWxKefjwqn3bdpbMff2pgwcVo4/9qh4aQLBzUsaT7233//8Prrr5eSiNx+++2ZiGRkFAgsyYjIE088UUwiYnAXI1JlInLbbbfFyp2TJk1qWJJRJHCZ/frXvw677rrrPJsVG6NNLSI1InLeNUPDyRe1nIgceOCB4bXXXgsHH3xweOWVVxqWlgOIyCabbJKJSEZGQVAKIsIiooBSVaGp2pprrjnPQTwZbQOFzP70pz/FsuGtfY3akoh8ViMiZ189NJxyccuJyEEHHRQLuWUikpGRMa9IROTJJ5/MRKSoMPv8zW9+k4lIQTFw4MBoDaHc9AVqTbQ1ETnrisHh9EsHNyxpPhIBKSMRYWF0raTDZ2RktD8yESkB/vjHP0b54osvGpZkFAkKeyGLGsH17NmzVVN425qInHHFkNDp8pYTkRQbUkYicuutt2YikpFRIAjyR0T0U5vXgP9MRNoIJ510Ulh99dXDRRddFK699trY7dWsTvS/KpHamt9///3hoYceCo8++mj0s7mgetO88MILUWF07do1Kgwpl2+//XYsRkWBKtOteZlOqpQo8Z6k5V6tU28R4nPj/8jsIQZ3r5axFAwaNCi2i1fsiwwZMiSK7/r16xd/az32xX4JgMSKVet8/vnnwzPPPBM/K3SjuVyXLl1i+qV+IY7de+fhxhtvDDfffHN8ve6668I111wTrrzyynDppZeGiy++OFxwwQXh3HPPDWeddVY48cQTw2mnnRZOOeWU+P7444+P95l2AkcffXQ46qijwhFHHBE7ywrK5IZANLyXtqqGhpYD4pa22267sO6664ZlllkmbLnllq1eXbUticj4CdNCp8sGhzNrZKSlSBVVk4umTEBENt1000xEMjIKgkREnn766eISEQpFt88qQvAjJbjeeuvFGARBkd5vsMEG0V3zu9/9Ls7uNttssyhbbLFFlM033zx+9koMvMpayxbwal3+6+Jb54YbbhjXuf7668f1U65J1llnnbDyyiuHtdZaa6aIWVljjTVmyqqrrhrJkuyRVVZZJay00krx1f9WXHHF+NlrEsG3ZPnll48i/TUJpa7bcGOxXPM/NTvSb7wSy3z2f+tL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XsfMXWVV3ZgF5yprXzZkIpKRUSwYT7h5C0tE9EjRT6XKRESDMUTEucgoHhQwU+J94403Lg1ZREQeff7juS7xLjiXMt9oo40alpQH6p8gUZmIZGQUAyyshSYiGp/p5FplIqLDKyKiGVxG8fDFF19Ey52AYvdrGTD5yxnhkWc+Dv88tnfDkpaBT1fA5+9///uGJeWBDtIKBH7wwQcNSzIyMtoTYs4yESk4xB8oH647bUbxMGrUqNgNWEBxWcgiIvLg06PDP4+bOyKSKskiX2WDQmziWzIRycgoBgpPRLSDP+mkkypNRLTbd/za92cUD4iIwnt///vfS3ONEJH7nhgV9us4d0REASKN4wTolg2IiIyfTEQyMooB5QAKTUTMME888cTKExGumXvvvbdhSUaRoIvr5ZdfHs39YkXKgC8mzwj3PjYq7H/83BERJe3VTdl0000blpQHKsJmIpKRURy89NJLkYj07Dl3WXz1aBMiomS2gaPKRKRLly6RiCAkGcVDnz594n1qpu1alQGffzE93NxlZNh3Ll0z7777bthhhx1iplDZkIjI8OHDG5ZkZGS0JxARNYkKS0TOOOOMONP861//2rCkerjrrrvCzjvvHB5//PGGJRlFAjeFxowdOnQIDz30UMPSYuOLydPDnQ9/FA48qU/DkpZB226Tg6222qphSXlw0EEHhf322y8TkYyMgkATzUITEQGaJ5xwQqWJyG233RaJyHPPPdewJKNIkAMvxZwLsSxkERG5/aEPw0En921Y0jLokllWi4geOcrTZyKSkVEM6KdWaCJCCXfs2LHSRETmkNlnJiLFhOty7LHHRuudKrhlANdM5/tHhoPnkogIKtOI8Y9//GPDkvLggAMOiJKJSEZGMWAMLTQRuf3228Nxxx1XaSJy1VVXhS222CJWWM0oHtR5ER9y3nnnxa60ZQAickuXkeHQU+aOiPTq1Stsu+224U9/+lPDkvKANSQTkYyM4sAErtBE5I477oizzaoSkalTp8aMjN122y2awzOKh/vvvz9W6tRnRqGvMgARueneEeGw0+aOiPTu3Ts+k3/+858blpQHiAj3zIgRIxqWZGRktCc0DdW3qrBERDpklYnImDFj4kxbwK6UyYzi4Z577ok1RGTOlIUsTqoRkRvuHhEOP33uiIhMIQXctt5664Yl5QEiImA1E5GMjGKAtb/Q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m9YsXbp08KkbWLNmTck5SyKSSNQD49sZZ5xRNxHRUKyvRIRg1Pnnn1/YYp/c/l3CP//5z5JMffbZZ5cGeH3B1KlTm2984xuDT93AqlWrkogkEpXhuOOOKw9yY/GwPW5EhNtbU7E+QlMxoRlekSQi9eLoo4/uHRGZMmVKs2zZssGnbuC2224rye9PPvnkYEoikRg2PvCBDzRnnnlm3UTkrW99a2+JiDbrCxYsaD7xiU/0Kv+gayA498lPfrJXTe8mT57cXHnllYNP3cAtt9ySRCSRqAzvf//7m3nz5tVNRFQkHHHEEYNP/YLupgY4bDGJSL0g8X7WWWclEakcK1asKFV4SUQSiXpAJ8vD9lgUZIwbEVExIzzTR1CuxBZT0KxukHhHFvuUrPq6172uc0REOfzhhx+eRCSRqAjHHnts0cuqmojwiLA+wo1efgivSJ8Gua6BR0RCcZ/O0d577905IrJ8+fJCRMjyJxKJOjBnzpwyxo2FMvW4EREVM29/+9sHn/oFT3Ckb7n9k4jUC71mTjvttF6Fz/baa6+ic9MlXHfddUWXKIlIIlEP9FI755xz6iYiQjN9JSKrV68uqqqetnXi/fOf/1x0K/72t781L7zwQvPiiy82L730UvOvf/1r8IvEeMOxZo57GIl36oB96jWz5557do6IXHzxxc0hhxySoZlEoiIIbX/qU5+qm4i4yfeViDzyyCNFVZWWimSer3zlK0XNkqcESdFN1E1148aN5SR6IpcwyX73u981v//975s//OEPzR//+Mdif/rTnwqZCULz17/+tZjpXhEc9ve///2/LL5rW/w+zDLbFuvakll322Jb22YfmPLY2CevYabb35Fmetvi2ITxMo00xzBMyXSY48sc6w0bNpTOyI49uXCie/5MfVK/3WOPPZqrr7568KkbUIWm10x6RBKJeiC0rZXJWEhUjBsRefe7312UK/uKn/70p0U4imbDV7/61aIrQoVOoyAhAcfGzfXggw8uFQHyaYSzxN10NSQ9TixGDO7Tn/5087nPfa754he/2CxatKjcmC+77LJyESxZsqQ84X7zm99srrnmmuLGvv7664tde+21Zbrv5QXYHoRIMq3fE16jebJ48eLy1Llw4cLmy1/+cnPhhReWdXn9whe+0Hz+858v6//sZz/bXHDBBWVfbBM2zDVnGyUtIV2SP223fdWB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)

切片在内存中由两部分组成并在栈上保存。

+ **ptr：指向切片的第一个元素的指针，这个指针是一个胖指针(Fat Pointer)**
+ **len：切片中元素的个数**

###### 在切片中的使用范围

在切片中可使用范围来切割数组或动态数组，签名为：**`&数组或动态数组名[范围]`**。范围：使用**`..`** (range 序列) 确定范围。

**切片范围：**

+ **前闭后开**。如：`[a..b]`，表示从a到b的范围，包含a不包含b。

  ```rust
  let vec = vec![1, 3, 5, 7, 9];
  let arr = [0, 2, 4, 6, 8];
  
  let vec_slice = &vec[1..3];
  let arr_slice = &arr[1..3];
  
  println!("vec_slice: {:?}", vec_slice);
  println!("arr_slice: {:?}", arr_slice);
  ```

  ```shell
  vec_slice: [3, 5]
  arr_slice: [2, 4]
  ```

+ **0到指定位置**。如：`[..b]`，表示从0到b的位置，不包含b。

  ```rust
  let vec = vec![1, 3, 5, 7, 9];
  let arr = [0, 2, 4, 6, 8];
  
  let vec_slice = &vec[..3];
  let arr_slice = &arr[..3];
  
  println!("vec_slice: {:?}", vec_slice);
  println!("arr_slice: {:?}", arr_slice);
  ```

  ```shell
  vec_slice: [1, 3, 5]
  arr_slice: [0, 2, 4]
  ```

+ **指定位置到结束**。如：`[a...]`，表示从a到结束，包含a。

  ```rust
  let vec = vec![1, 3, 5, 7, 9];
  let arr = [0, 2, 4, 6, 8];
  
  let vec_slice = &vec[1..];
  let arr_slice = &arr[1..];
  
  println!("vec_slice: {:?}", vec_slice);
  println!("arr_slice: {:?}", arr_slice);
  ```

  ```shell
  vec_slice: [3, 5, 7, 9]
  arr_slice: [2, 4, 6, 8]
  ```

+ **全部**。如：`[..]`，表示从0到结束，等于直接引用。

  ```rust
  let vec = vec![1, 3, 5, 7, 9];
  let arr = [0, 2, 4, 6, 8];
  
  let vec_slice = &vec[..];
  let arr_slice = &arr[..];
  
  println!("vec_slice: {:?}", vec_slice);
  println!("arr_slice: {:?}", arr_slice);
  ```

  ```shell
  vec_slice: [1, 3, 5, 7, 9]
  arr_slice: [0, 2, 4, 6, 8]
  ```

+ **如在使用时超出了数组和动态数组的长度，则会发生越界的错误**

  + 如：`[a..]`，a**等于**数组和动态数组的长度不会报错，但会返回一个**空切片**

    ```rust
    let vec = vec![1, 3, 5, 7, 9];
    
    let vec_slice = &vec[5..];
    
    println!("vec_slice: {:?}", vec_slice);
    ```

    ```shell
    vec_slice: []
    ```

  + 如：`[a..b]`，a或b超出数组和动态数组的长度会报错

    ```rust
    let vec = vec![1, 3, 5, 7, 9];
    
    let vec_slice = &vec[..8];
    ```

    ```shell
    thread 'main' panicked at 'range end index 8 out of range for slice of length 5', src\main.rs:297:22
    note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
    ```

    ```rust
    let arr = [0, 2, 4, 6, 8];
    
    let arr_slice = &arr[6..];
    ```

    ```shell
    thread 'main' panicked at 'range start index 6 out of range for slice of length 5', src\main.rs:298:22
    note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
    ```

###### 访问切片元素

切片的访问与数组和动态数组类似，同样也会坚持切片的索引是否有效，存在越界错误。

```rust
let vec = vec![1, 2, 3, 4, 5];
let vec_slice = &vec[1..4];
dbg!(vec_slice[0]);
```

```shell
[src\main.rs:305] vec_slice[0] = 2
```

**越界错误**

```rust
let vec = vec![1, 2, 3, 4, 5];
let vec_slice = &vec[1..4];
dbg!(vec_slice[3]);
```

```shell
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 3', src\main.rs:305:10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

###### 切片的常用方法

+ **`len()`**：获取切片长度。

+ **`is_empty()`**：判断切片是否为空，返回值为**布尔型**。

  ```rust
  let vec = vec![1, 2, 3, 4, 5];
  let vec_slice = &vec[1..4];
  
  if !vec_slice.is_empty() {
      dbg!(vec_slice.len());
  }
  ```

  ```shell
  [src\main.rs:307] vec_slice.len() = 3
  ```

##### 字符串

###### 字符串定义

字符串是由字符组成的连续**集合**。用**`""`**（双引号）包裹起来表示。

字符与字符串中的字符的区别：

+ 字符是`Unicode`编码，占4个字节；用**`''`**包裹。
+ 字符串中的字符是`UTF-8`编码，所占字节是变化的(1 - 4)，这样利于降低占用空间；用**`""`**包裹

###### Unicode 和 UTF-8

+ **ASCII编码**：在计算机中常见的编码是ASCII编码，但范围只有0x00 ~ 0x7F，无法存储汉字，少数民族文字等。

+ **Unicode字符集**：由于ASCII编码无法存储汉字等，从而出现了GB2312，GB18030等编码，为了统一字符编码，制定了通用的多字节编码字符集——**Unicode**字符集。其包含世界上所有语言和各种字符。常用范围0x0000 ~ 0xD7FF 和 0xE000 ~ 0x10FFFF。

+ **UTF-8编码格式**：Unicode字符集每个字符都占用4个字节，为了节省空间，**UTF-8**是最简单且高效的一种编码格式。

  rust中`String`和`&str`类型都是使用**UTF-8**编码格式表示文本。

  UTF-8是以**1字节**为编码单位的**可变长编码**，它根据一定规则将码位编码为**1~4字节**。

  rust中，一个英文字符或数字是占一个字节，汉字占3个字节，还有一些Emoji表情占4个字节，可用`std::mem::size_of_val()`方法查询内存占用的字节。

  ```rust
  let word = "中";
  let num = "1";
  let ch = "a";
  let emoji = "🙂";
  println!("word: {}, num: {}, ch: {}, emoji: {}", std::mem::size_of_val(word), std::mem::size_of_val(num), std::mem::size_of_val(ch), std::mem::size_of_val(emoji));
  ```

  ```shell
  word: 3, num: 1, ch: 1, emoji: 4
  ```

###### rust中由两种字符串类型：

+ **`&str`**：原始的字符串类型str，常称为**字符串切片**，它通常以不可变借用的形式出现，即**`&str`**。它是一种**固定长度的**字符串，不可随意更改其长度。我们常用的字符串字面量就是`&str`类型。
+ **`String`**：这种字符串类型是一种可变长度的字符串，可随意更改其长度。String是一个结构体，它是通过封装了动态数组实现的。

###### 字符串字面量（&str）

字符串字面量就是**`&str`**类型

```rust
// 未声明类型
let s1 = "hello rust";
// 声明类型
let s2: &str = "hello rust";

println!("s1: {}", s1);
println!("s2: {}", s2);
```

```shell
s1: hello rust
s2: hello rust
```

**转义**

在字符串字面量中，英文双引号(**""**)需通过**`\`**转义

```rust
let s = "hello \"rust\"";
println!("s: {}", s);
```

```shell
s: hello "rust"
```

**忽略换行符**

当定义的字符串过长，通常在编辑代码为了美观会进行换行，这样也会同换行符一同输出。 可通过在换行处以**`\`**结尾进行忽略换行。

```rust
let s = "hello
rust";
let s1 = "hello \
rust";
println!("s: {}", s);
println!("s1: {}", s1);
```

```shell
s: hello
    rust
s1: hello rust
```

**原始字符串（Raw String）**

原始字符串：字面意思是未经处理的字符串。原始字符串以**`r`**为标记，其中的所有`\`和空白符都会原样包含在字符串中，**转义符在其中无效**，如原始字符串中包含英文引号(`""`)，则需在字符串的开头和结尾添加**`#`**标记(#的数量可自定义，但开头和结尾数量要对等)。原始字符串常用于输入文件路径等。

```rust
// 转义测试
let r_s = r"D:\rust\helloWorld";
println!("r_s:{}", r_s);
// 英文引号测试
let r_s1 = r#""英文引号"测试"#;
println!("r_s1:{}", r_s1);
```

```shell
r_s:D:\rust\helloWorld
r_s1:"英文引号"测试
```

**字节字符串（Byte String）**

字节字符串就是前缀为**`b`**的字符串字面量，字节字符串是`u8`值(字节)的切片`&[u8; N]`，只能包含**ASCII**字符和**\xHH**(转义字符的格式，以\x开头，后面接两个十六进制数)转义序列，其不能包含任何**Unicode**字符。

字节字符串不支持之后介绍的所有字符串方法，但是它支持跨行，转义，原始字符串(以**`br`**开头)

```rust
// 字节字符串
let b_s = b"hello rust";
println!("b_s: {:?}", b_s);

// 原始字节字符串
let r_b_s = br#"hello "rust""#;
println!("r_b_s: {:?}", r_b_s);
```

```shel
b_s: [104, 101, 108, 108, 111, 32, 114, 117, 115, 116]
r_b_s: [104, 101, 108, 108, 111, 32, 34, 114, 117, 115, 116, 34]
```

###### 可变长度字符串（String）

String本质是一个字段为**Vec<u8>**类型的结构体。每个String都有在堆上分配的缓冲区，不与其他的String共享。当String变量超出作用域后其缓冲区会自动释放，除非String的所有权发生转移。String在内存中的形式与动态数组是一样的。

**创建字符串**

三种方式：

1. 使用**`String::new()`**创建空的字符串

   ```rust
   let empty_string = String::new();
   ```

2. 使用**`String::from()`**通过字符串字面量创建字符串，**实际上复制生成一个新的字符串**

   ```rust
   let s = "rust";
   let rust_str = String::from(s);
   ```

   rust_str是一个新的字符串：

   ```rust
   println!("s指向的地址: {:?}", s.as_ptr());
   println!("rust_string指向的地址: {:?}", rust_string.as_ptr());
   ```

   ```shell
   s指向的地址: 0x4b5188
   rust_string指向的地址: 0xb06f80
   ```

   `s`和`rust_string`指向堆的地址是不同的，也就是分别在堆上两个地方保存。

3. 使用字符串字面量的**`to_string()`**将字符串字面量转成字符串。**实际上复制生成一个新的字符串**

   ```rust
   let s = "rust";
   let rust_s = s.to_string();
   ```

   `to_string()`实际上是封装了`String::from()`，所以`to_string()`也是生成一个新的字符串。

**可变长度字符串`String`与动态数组`Vector`**

+ 联系与区别

  可变长度字符串`String`是由一个封装了`Vec`的结构体

  |                                        |       Vec        |       String        |
  | :------------------------------------: | :--------------: | :-----------------: |
  |              自动释放内存              |        Y         |          Y          |
  |                 可扩展                 |        Y         |          Y          |
  | `::new()`和`::with_capacity()`静态方法 |        Y         |          Y          |
  |      `.reverse()`和`.capacity()`       |        Y         |          Y          |
  |          `.push()`和`.pop()`           |        Y         |          Y          |
  |        范围语法`s[start..end]`         |        Y         |          Y          |
  |                自动转换                | `&Vec` -> `&[T]` | `&String` -> `&str` |
  |                继承方法                |  继承自 `&[T]`   |    继承自 `&str`    |

+ `String`的构成

  `String`类型由三部分构成：

  ![String](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAk8AAAEACAYAAACqIZJKAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAIVqSURBVHhe7d0HuCxVlTd832femXd0RmfUcQyjo5gVERxGkkgQRYIKgoiIAgaULFGiEkTQUZCgKMkAEgQVEQQVMYARJAeVIEGCiBhQRDDUd377O+vOtu17uX08t7vq1Po/z3q6u7q7wq5de/33SvtBTSKRSCQSiURioZHkKZFIJBKJRGIEJHlKJBKJRCKRGAFJnhKJRCKRSCRGQJKnRCKRSCQSiRGQ5CmRSCQSiURiBCR5SiQSiUQikRgBSZ4SiUQikUgkRkCSp0QikUgkEokRkOQpkUgkEolEYgQkeUokFiH+/Oc/N3/605+mPyUSiURiLiDJUyKxCPGzn/2sufLKK5s//OEP01sSiUWL3//+983999/f3Hfffc0111zT/OIXv5j+ZjyY9PETiXEgyVMisQjxyU9+sllttdWa22+/fXpLoutgSfzpT3/a/PjHP27uvvvu6a3jh/PQr5DzX/7yl/O2fetb3yrbnN9KK63UnHLKKeW7cWDY8U844YTmRz/6UZlIsMQmEnMBSZ4SiUUI5GnJJZdsfvjDH05vSQS66tK8+eabm7e97W3N6quv3nzkIx9pbrnllrFfh7a78cYbmx133LF5yUte0nzjG98o23/3u981b33rW5t3v/vdzXe+853mP/7jP5rPfe5z5buFxV133VUsRixHo2LY8Y855phmjTXWaPbcc8/m5z//+fQvE4luI8lTIrEIMVPyxKJx0003zTl3H5Lxq1/9qrn22muLcr300kuLmwe4ehCC3/72t+VzG4G0HH744c2//Mu/NM997nObJz/5yc22225brCzjJFC//vWvm5122qn57//+7+b73//+9Namuffee5vtttuu2W+//ZrTTjutWXzxxZurr756+tuFw8c//vFiLf3JT34yvWXhMez4F110UbPXXns1j370owuRShd2Yi4gyVMisQiBPC299NLNddddN71l4fDFL36xedWrXlWsHHMBrBgsNBdeeGFz2GGHNWuvvXaz2GKLNS9/+csLSQTX+trXvrZc+yigjLmEgoQtSjgGcrD++us3d9xxR3POOec0K6+8ctl2/fXXj41AnXvu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veaiiy5qLrvssuaqq65qfvSjHzXXX399c9NNNzW33HJL89Of/rT5+c9/3vzyl79sfvOb3zT33ntvc//99zd/+tOfpve08PjjH/9Y/msfv/3tb5u777677Nf+f/aznzW33357Oebll1/eXHnllc11111Xzufqq68un53jxRdf3Fx44YXlvL/97W83559/fvONb3yj+epXv9qcc845zZe+9KXmrLPOas4444zmc5/7XPOZz3ymOe6445rjjz+++dSnPjVfOfnkkx9QTjrppAXKkUce2Xz0ox9tTjzxxAeUE044YcZy6KGHNh/5yEfKfhzX+Z9yyinNqaee2nz6058u13zaaacVOf3005vPf/7zpT3OPPPM5gtf+EJz9tlnN1/84hdLW335y18u7XbuueeWNvza177WfP3rXy9tqm2/+c1vln6irb/zne+Udr/gggvKPfj+979f9uX3l156abk/7t0VV1xR+pL79oMf/KD54Q9/WPrbtddeW+6p/vXjH/+4ueGGG0o/u/nmm5uf/OQn5d7fdtttpR/od/rEnXfeWfrHXXfd1fziF78o/eXXv/516Tv6o370u9/9bl6f8vqHP/yh9M8///nP0z2vXXBungXned999zW///3vy3nfc889ZdskkeRpPtCZ3Kj6ZumEv/rVr0rn1EnvuOOOeYOYTn3jjTeWjq7jG8g8DB4MD4kHxmDmIfrKV75SHjAPV8h3v/vdvxIPYC3+U4sHdZh4iEM84Oedd155uEN8HhQDQC0e8lrsZ1AMICEGlsFBhdiX/cexY4AhrsF1xfVGW9QDDqGwtN0ll1xSJAafegCiMIj2Js7D9wYj4n4YlAYHpnpwcv9IPUjFQHXrrbeWwaoesNx/YuAi+kQMXsTgRfzXYKb/1AOZPmUwI9HXDBCUpoHBoEH0xbYObnMd7hFlu8ceezS77LJLs/322zdbb71185a3vKV54xvf2GyyySbNa1/72ubVr351s/766zfrrrtu8/KXv7xZe+21mzXWWKNZffXVmxe/+MXNaqut1rzoRS8qryG2x3fDxHcveclLyj7sa80112zWWmut5mUve1nzile8ollnnXXKZ+9f9apXNRtssEGz4YYbNq95zWvKOb3uda8r57fZZps1b3jDG5o3v/nN5bzf+ta3NltttVWzzTbbNNttt125ph133LFcn8877bRTud4999yzyF577dW84x3vaN75znc2e++9d5F99tmn2XfffZv99ttvnrzrXe9q9t9//+bd7373PDnggAOKHHjggX8hu+66a9nne97zniLvfe97F1riP2Rwv3G8kLe97W3N29/+9nJezs95Ou+4DtfkPFyja3Xdu+++e7PbbruVc/Rfom20i3YiO+ywQ2k3+9dm2267bWlPon9o3y233LLZYostimhz9+T1r399s/nmm5d78aY3van0IffGPSKbbrpps9FGG827f2TjjTcu29xX91dfc6+J+67frbfees0rX/nK0v/0C6Jf6Iv6i/5I9JfoR15f+tKXFtHHSHwO0e8Gxf9qGbbtgST25Rydn/f1cevzqffvvF2He3PMMceU8XxSmAh5MiBh/mYEtRxxxBHNBz/4weawww5rDjnkkOYDH/hAc9BBBzXve9/7ykPj4fBAxkPgAfYALKjzxwOg8++8885F5vcQxIMQD4PXwQfB4KPzxwNQd34d32DlAdHh607vIdDxdXidQEfX8Wdb4oEK8TB5uOK7we9D4rv4fvC3w76rP8f1eO/hDokH3muINgkxSJBoLxKDBtGWIdqWRDt7JTHwhLgfBg3/cX+IgSrEfasl7idxf0MMeCExCIboDyH6R4j+Qmz3ajB1DO+jb0VfI/qePhiiT+qftUS/1YeJ/qxf6+NEfyeh8DwLgwovlF0ovFB0oexCwYUi+5//+Z/y3L3//e8vz6DtBx98cHkuzeY9o4cffnh5Xj/0oQ+VbZ5fz/FRRx3VHH300WVwO/bYY4uFwfNuMoEcdgVILYuD+0c56XP6j/7gvrpf7oU2177aUZtpI22iDT7+8Y8Xq0dYOz772c8WKw9LR7wS21lBWEP8joWEpYT1hDXIfrSjNtW+MV66B8PGS/cvyIV7O4w86BuD5CH6VIyf9dgZ42Y9Ztbj5SB5iHGTeGbq5ymeu/p5W5DU/yX1s+g8HD+exXgGnYvvbfN9LXHeIfUzGeIaaxl8TmsZ9rwOPrf1Z+0az3Et2t25O179PLtH9XNcP8P1szv43Hpm9Yt4ZuN51Xc+/OEPl37EChjPazyzg6LvhXzsYx/7C4n+Pri9lvr/IfZbjxVefSbOybk5R/t23vq66yDugXYy6Z4UJkKeDBIIRLBMM7KVVlqpyKqrrlpmXLZjmJipgYtSpmwpVAoyBjENGJ1dx40OSZlExzNA6HQ6XN3ZdDSdrO5gdeeKQSo6WNxYN3mws0WH0FEMdOQTn/hEEYPfJz/5yWIO994272uxPWTw84IkjhHHqT+TOI/6fOJz/d3CSn19gw/H4GeiHbWR9/XDSKL9BiXaNyTanbgPIXFvQjxoIe6jY9fivob4vhYPZohBJiQe1hADRYg+U4t+FKLfGcz0L/0sZs36Xa3QQqkZEGMyUCu0YTPhWpkNKjKKoVZgteKihIL0kyD+NfmvJwBBYBFazyESjOjGhCCIMUFWhxFrgrzbByXj2edK6AJYCCki45BnMqyTYZUMa2RYIblNwmXCQu06a3dJuEzmJ/EbYoJJ/Nc+wmppn8T+w7LpeCQsnqyfzoWEVTSspM6VsJ6GhEXV9YSwtoaEBZa49lpYaQclLLiDov1mW+x3Qcetz2tQBq+llvqah0ndPoNSt+MwifYeJuHJ8LuwcNdWbhL3lMR9Dqt3bfmOvkiirxB9JyT6E4k+NqrEPod990AyeOz6fEh9rvU1GFONWbwXk8LYyRN3CoXAIkI5mVnxkZuZGVjNxvic+eLN+ri4uIG4fjQUtw5XDvcNl03EHhCuGC6YeGiik0en1PGio0WHihtX+4PNjsN9UgtXyjAJ90oIP+2gzG/7/GRwn7UMO4dhMnj+tcQ1PpCEO2l+MqgABkX7UgLe18phmITCWJC4TwsSiuaBJB7U+Un98M5P6od6mBj86sErlBoZVGq1QotB1P9jMNaPDej6tL5NaejnXI76PDck87VngIuShUcMhWeDK5Nb07PCjcnlGe5P7lDieeIm5S4NF3K4icPF6rmtXby1O9d33LaeVSIuxGfPLxEzIt4DIUPyXFMXoC8gttwKrimRSLQDJq8mer0iT2bvZtVmzwZ8SpMyRxgSicTcBcsTixcS2AUgwazVJnri+RKJRDuAPPXO8oQ8cU0YlMywE4lEP8DlyEXIetYFsExzp3I9srIl2gleCu79Gogv9zVLKxn8PtFtBHliIZ8Uxk6exJRw24n74KpIdBNcV9xA3Et1Jhg3FDcQBcldxKXE/ZdIiN1iaud27AK4WsWhie2SKZpoJ8STmozXUHIAUQ9XeRCpxNwA8uT+9o48CXaVCSDmI9FNiLURDM2dIb4qIF6GZdFsUOaQ4GnxNEmgEgLgBaB3hTxRuvqyc56keyCxYAySJ7GYkikkkUQMqRpOkjoScwO9JE8ynQSNIlECZxPdxPzIk9mdDDLbuWVlvnHVskBlraJ+gxVHpp5g9y7A5E58JveAQPpEOzFIntSEe9SjHtUstthizdJLL13kGc94RvO4xz1u3meTu0R30UvyJGVc+rRZAbN4olvglpMNiSCJBREELEMy3HdBqtSuUaE3fidrCYGSLZjoJ/QFZQ1kCXYB+jprmSB31tNEu4AAIUJIUhAjtbaQ3ac//emN7G0hBLJFlbtRHsVnkrqn2zAhV2qlV+SJJcJgpO6PgMxEt8BqwOUqA8lgpQaO+kVS0rnpgizJplSbyL1WX4j7w6AlszLRT4hzVP8Jwe4CxGSqFadOFmtGol1AgIwpJmbukfcm5Q972MOaJZdcspTtABnd6pxZ3SAxN9BL8qRzCxpVdE5GRKJ7EEOgdo/sqXDbqSNk1ic+ZNllly0BmmoMZQmKREBhOxYAFsgugNtZkLu+LDki0U6E206skwKzLEwKLy+++OJlgve85z2vuPG47nwmGTzebXDb9Y48qQytSrHlBxShS3QPij6yJKkILyYEcYqAcLN1FgYFUGMB0BDB5GaBiX5C8gCL5STXoxoFXNGspyq4mwgk2okgTyZqUViWhXNBbjvFchPdRS9jniyxoWNby0lF6ES3YIBi/raExxJLLFHSuMWxqTwt9slyFrEYpdgDy+jE+lRdWpojMftAqC3z0hXypLI766rsYFXbE+3EYMC4LEnjUrrt5i56SZ4UKzP7PPPMM9MK0UFwZciU5KITDyJoUwFBdVWs5YYsrbLKKmWhYMt2iHESeMtVk8Sp3xAbZ93KUGpthyVxxO5ZU9DSN4l2Yhh5svp+uu3mLpCn3hXJtPCrIGMBxtY7S3QLigUK+mdFEsMyWKpANp21ChEpa6shTgY3CwlTRon+QpDnS17yks6QJ4VeZZOaJHQlyL2PCPKEELE4ccul225uI8hTr5Zn0YFf+tKXlgVFs3Bi92DBWSLOaVidJ2AeF2j7wQ9+sCQGsFTJxLOg7GBF8kR/4Nlfc801O0OeFPPcYYcdSnxfVwp79g0IU5QqUATT2CTmKd12cxu9JE9KFMiE4NJJ8tRdzK9IJgiulRQg7omVykKwXHuKJLJcZQZeP/GJT3yiWWONNToTP6Qsh4K+rBppNW0fhIBQoMYg1qRYKzVjnuY+ekmePvaxj5WYGAHGWfOnuxhGnpAkiQBcdDKUECgFNZnIY6mWrlgdErOPE044oVl99dWLdaALENi+1VZbldplrBmJdgFZco8GJ28LE/MkZlNpg0Q3gTwpedQr8kSxrrzyyiVtPatNdxc1eVIxGmkyExQfwsLA+iTQVt0n8U9q5fg+l+TpL5QnETDeFfIkyQF50n9vv/326a2JtgN5qmOehomJXlrAuwvkSca3eLZJYezkiWJ94Qtf2FxwwQVJnjqMmjypvmwmN1iKQFwUc7lZ31JLLdWceuqpmWHZY+gfq622Wkkk6AJYNVSuVn4jFzHvDliiuFwzIWnuopfkSQDxiiuuWNh/Mv/ugjL58pe/XOJXFnQfo/K4BaHN9hL9hZXtxTt2hTwpT4D8C0TOtdASfYUEH2M8UoqQqs/o9Z577invFbs2abbcmgLKnhUeBpnWxnyTEIlCXN9iB22TyYrg8lqw8HrWhHRcccUVzSWXXFLGCN4Ln+kQFf7tJwpreyZf//rX94s8SStdYYUVSgNl1lUi0R+oCbbqqqt2plq3wdzajIp75jqciUBNJsTtip1iUUcmxHdS8JYeGyQSVl8Qp4UwIAcLIhFc20EeEAf6koWf0YHXRuaycj9ifgTCiyG2TWKOZKyvfOUrzTnnnFN+Yyktz57aimJQ1eQzkRFqwRrMI3DKKaeUEjMnnnhiiU1k5DjuuONKmI045RCfeY8kftUik7YWJYmIZ0dJm/hMhG8MiuLZxO9Zer2vV6cgzlfhWkCehIRMMglg7OTJjVluueVKh0jylEjMDJ6dGMS5v0kM5kQmq0HdDNFAbnCPAd5skcRAHzNHv4tB3wwxBn8ijoQSYHEkoQyIeCBKgRXSDDNmmQY6Yl/Ecj2SRbpAnlyvJAcV0cXuUUqUVsj3vve9eUJxUVbiOGtRwG+YUHjDxCy6Fvt1XO8piUGRubog8f9BoWAJxXr22WfP+xyKd0GivMwwoaxDuPG1Bas0pU15f+ELXyjKmgInPteKPJS5FQpOO+20oiRDsVPoJtyUu5g5Ylso+FD2fhMKn1DwteKntCnkWoHXCtqaqzKDrZagxApLOfFeqRVCscd3lLcEGKELlD2CoAispakOOOCAYm2XaCBTc9999y3lLqy7ZzkrtcO23HLLUs7F8j/6l2KsKtn7TnkMCTfbbbdds+222zZbb711ib3zH5bQzTffvGQT6pvWd5N1JniaGwuhkKiz0UYbld+8+tWvLvFfFmtX60rxYv+TCW0lCDUXic9KCKnDJi6RhdizKj6ZeG+bbFm/lfjh9z4rPyJA33b7XmeddUoh7Die985jww03LOfk/DbeeONyrqqEm6B4dS2uwXeKLcei8trIPUckYxKj/f3OczApTIQ8WTiWKS4xtzE/BU+JU+61gieh3EPBz0+5h3k4lDvlTbGHcqfYQ7n7LpT7/BS7GSBRywcBMBOM2aDYLmJWyGxMYnZI/N4skYSpOWaMZophclaPxuwxZpAXXXRRETPJCy+8sEitlEMB+75WrKE0QxnWSo3iqpVXKDDPHEVZK65QWrXCoqwQBiSHAjBgeT84Gw3FFDNOyojyMciHAqJgQuGITzDY7bHHHmUg1jZth/tM6S2//PJlkHYdrnGYCCjfa6+95n0ebAPX7/+KhEZbULwHHXTQXylf+woFrLSHBIx3vetdQ5WwY/q+VsI77bRTUbyDiljJBcpY4ob7RNlSaJQxBUUhU1ahjAlFxjVCyWmDUMiUYCjk9ddff56CJPHedt/7TDlTtpQp8V2I34TYZ4hzI5QxJe24Ic7Df30Xipg4T+KcifN3ncT//Md1haImlHQoam3gP/apXcS7EaQFedFu2lCbIjXEZ22rjW3X5sR9cA+CFPns/sR9Ip4H9899JO6pe0vcZ/ffPSfuv36AkPnOfnyvn+gv+g3Rh4j+pF8RfYzob3Wf0w/1R32R6KMkyCJBFhck0b/td9j3pN5PvW/iGfGsDG4P8R9jkDG4hnN3n4yFk8LYyZOBmOUpBtCYKc9PKNRh20NCCddC8Q7bHkJhDwoFXSvwmJ0PKm+Ku/btxoycEqaMh83IF6S0mW0JxU0pG7RDaVPYobR9F0qbwg6lPaiwKWtCGVPUM1Xa9QzZ+3r2WyvvwVlrPRutFXltPjaLjJlnPeMM5c2MHDPMemYZittMMhS32aPBo1Za8ZBSVrWiGjZDrGeHBi0D6aBSivXNDIAGxmGzQoNsKCH7MECbgcVsMJQPxUMpUBYUCUXjdzELNIszm6NszAApiJgBSrR4wQteUBQ617f3tlmg2fdcYmaHgrLNIO3Dfn2O2aFj2FbPDimVUFiUjPe+rxVQKBzX5hpdaygY108oymEKhVAi2t15t33i5PnUvyhNiQ7aTh+oFV0oQdv1Ae0S/SOUJ4nlXbwSvw8JJWpfRH+jUL3WfS8Ua0itYEMoUuL/jqNPh8IN0dcJq4jf2pdnIMQzEfsYppBDKYeEcg4F7RnzrHnmKEUWHc9iKGXPqOeV1Qfh1sbhCvJse8Y965SlZ984EaQduTdOGC+MGzFemBQYW1jSjDXGHOOPMcn4ZLwydvlsTDLWGfuMgTE+6o/GT2Opz74z7hqDTaKMzcZq/cIYbiw3rofEBG1QQg+EDojPMbmrJSy6tcSEMISuIfZHf4RVuJaYVIaEvqolJqEh9ukc6DgSOo+YwNYSejEmucTv7IfujIkwqfVrrX8H9bfvTapNrk20Y9K9IOhnxtPekSeDv9m4Dqqj1+bb2ifrlSKNzx6eEA9PbeKtZ84exHgfYj/h263Nv/GQetDjQa1n1rWJ18M/TDkbNAyCBtdaMS/IdDtMOVNKFFLMFkNBmy2G2Zayitmi2VIo6nq2SFlTwJShDhYzxpjNUY6hvAmlR1kSyjoW9g2TLqVrf8QsMMy7YeKlnCntEEqcwg9l7jehzP1XwkAo8lDmtbnX+cSsNJR6mHxdixlHPeN0DNcdbTA4s4xZZSj8UPoxoyQUoM/ORXvXJCCIQE0GSK0k3b8Qysur862VZUgozUEJhRxS/z4UbP2bep9xzDgf4vz0uzj3wRlynHccI5S3Pkphh4KmUO0nFLB+rX+HYg1F6tXzQFF6RjwroTA9UxQYwtd28mSCQvlrg2WWWab0MbPr+rmPWbHXQYlrR+YRfNdekwWfjTPIgvEQWTAeGZ+MVcYuY1sQBdZCJMHkw0QESUAEgiRQIAiCSQ6ygDQgCCZLCIKx1sQKOTDpcn0mZEEOTN6CHPjsO0qeAqdYKWHKkZIMxUk5UnyUXii8RGJcQJ6Mr/r6pDARt53Zp4fagERhUdQhobhj1kyBUqwhoeRD0YZQuISCp2gHzcOhjEMo5RBkIywCXuenqAfNwmESprC9hsIeprTrmXrM1ilsQlkTbeFzKDsKrlZyFGQozpidxmw4ZqQxE/Vbr6HsSMwmY+Y4OFukFMJsWysA9wm5RCIpA0ogAgcHZ4yUADIapJUiCHIbhJdCDXIchJmwRlEUERsRCiNiJ8IVZZYZM02vFFIoE9avmHUSiiUsZhRMbU0La5vZaFjgEHMzz1A8ZqKEAgqrHkUUM1XWv7AEUkphPfTq+BQUS2LMYENRhaWS1DPZwdkp5RWzynrmWM8MQ6HFzC9memZ2PpvRtUG5UegsZW0nT9rPTF9/MT54Fnw2K0ck3IuaQGjnmDUnEolFD3qLnjbWTwoTIU9mn5QR5UKpUWZcRRQZJWaQpfQorVBUoaQopTCphjKiiMKc6reUjsEuzJihbCiaYYMeSSQSixaeeVZHr12AscfEyUQhx4hEoj1AnkxsekWeWCkMoEhOIpHoD1jqYuLUBbCQsT6zsKZVKZFoD5AnHiSehklh7OSJr5/pPslTItEvsBp3iTw5T64BrumM6Ukk2gMhJ0JxhGpMChMhT7KDzEITiUR/IPYLeeKO7wKCPBmzEolEe4A8iXHuFXkyi5NWneQpkegXBMpLFukKeZIwIIlEqEEikWgPZPdKGlOaYlIYO3mSaWX2meQpkegXZBN2iTzJukSeJLkkEon2AHmSfd8r8qTmS7rtEon+QQYs8sSi0wXI/EWe1GJKJBLtgVI7ShkpVzMpjJ08qR2kgKLSA4lEoj9Qo6pL5EnplCRPiUT7gDwpsKzO36QwdvKk6KKK0Go2JRKJ/kCRSS577rAuZK8ppKqWjOKviUSiPbCygVUqekWeVKm2VEeSp0SiX1AJXY037rAukCdV7y0tlDFPiUS7gDxZTcSKE5PC2MmTZT4svZLkKZHoFywfo8ab1QS6UHRSGrR0aOtfJhKJ9sAyY3iEpbsmhYm47SwOax2wRCLRH1gzDnmyHFMXyJOlH1Qxtl5jIpFoD6zVquSRtVAnhbGTJ6uVr7nmmkmeEomewQr8yJP1LLuwVpylH8Q8Weg6kUi0BwcccECJnbZ4/KQwdvJkhXKBXpZqSCQS/QFrkzIlFgTvAnn66le/WsjTqaeeOr0lkUi0Ab0kT4ceemhJMUzylEj0D8iTWCJWqLZDDRluuyRPiUS7gDytuuqq/SJPhxxySCludc0110xvSSQSfYE4BenF999///SW9kImD/L06U9/enpLIpFoA8Q8qRd51llnTW8ZP8ZOng4++OBSVj3JUyLRP5gtypC57777pre0F84TefrMZz4zvSWRSLQB+++/f5mI9Yo8WQ3Zgn5JnhKJ/kGcwplnnlky79oOLgHk6bOf/ez0lkQi0Qa8613vKiEAvSJPyqqvu+66zbXXXju9JZFI9AWrrbZa87nPfa659957p7e0F2KdWMmTPCUS7QLyJHO3V6UKFLdab731kjwlEj2Eit1iiBTMbDtOP/30Zv31129OO+206S2JRKIN2G+//fpHng488MAyIF133XXTW8aHe+65p7n99tuL3Hbbbc2tt97a3HLLLc1PfvKT5sYbb2yuv/765oYbbmh+/OMfl/ND8LgXVUO3kDG56qqrmiuvvLK54oormssvv7y57LLLmksvvbQsdnrxxReXdbu+//3vlyUoLrjgglJN2RpZCgMquidYVraRdG1iG1FThviOSJM+99xzS8aP/8QrEchKvvSlL5W4DMLFQJgxie/9h4vEMhMUgRn/5z//+QWK34b4L9FB7TM+O679x3k5T+fr/F3L+eef33zrW98q1+z6tYM20T7aS9tpR7W+tK921/7a3DWoq2N/0c6DbT2/9o42t6BrtHu0vfNx/c7XZ9vj3PzPPuzLfu3fsRwzztW91w+cs0xRfcP5Onf9Rb+56aabmptvvnme6Fv6mL7205/+tLnjjjuaO++8s7nrrruaX/ziF82vfvWr5u677y7LliATXFkCqaXxd2H5kpnA0kyKTnoW2w7Pg1IF+k0ikWgHlDzpJXkSJW9AGjd5cjzVzd/5znc2e++9dxHv99xzz2aPPfZodtttt2aXXXZp3v72t5fXnXfeudlpp52aHXfcsdluu+2abbbZprxuu+225f3WW2/dbLXVVs2WW27ZbLHFFs1b3/rWIptvvnnz5je/ucgb3/jG5g1veEOz2WabNZtuumlZoV2ZBpa3jTfeeJ689rWvbTbaaKMir3nNa4psuOGG5fdiLvxem3lPkE/biOUjuEFDxJNxNfiv/clsfNnLXlaqupv1e+8cFiR+E+L/9me/6nNZT8gxnQNxXo7lfB3Ptbzuda9rNtlkk3LNrv9Nb3pTaRfto720nXZ829veVtpXW++6667lOM9//vObpZZaqpyr/2s77ag97eMtb3nLvP1o/9iXe2N/O+ywQ9mnexf3077dX/vafvvtm7322qt5xzveMa8v7LPPPs2+++5bxEPJJCwgkeivIdJjiQnA/CR+Q+r/xv6I/TtOiOM6BxLnY7vf2qd1nLi7xQtKuJCxethhh5WaaUcccUTzkY98pCx7ZN3IY445pvnoRz/afPzjH28+8YlPNMcff3zzyU9+sjnxxBPLAreWGkFeEFRuKVYgIiiacFENSnznd/bhPRI5k7glSyo4F4Sx7UCa9G8Tj0Qi0Q4gT+G2M5mfFMZOnigEytaMfZxgnaH4reqOEFCqNWFCoChVrwjV/BRsrVxDoVJw3JGh5N73vvcVoewOOuigovA+8IEPlPe+90r5kcMPP7wIRUhUYB8U2xE/QlGGUJjkqKOOKkJ5hgIlxx57bBHK1Ha//djHPvZX4vtBif/GvvzfMSnrOK7Pzsm2OE/X4roo+Lhm7eC6tZH20nbaUZtqY+3tXiy77LLN0572tLJ4LCLkGPZv3/arRljsU/vW+3QvgpQEGXHv7N+9dAzEyv1GprwiWLY5VpDiIMQIGqI2SIKRwte//vV/QXqD6FK0QSqD1AaZDRKLnCKhquzrj0gtQsEiQ8QEEeTRdt/7nd/7XxBaYt+O4ViItOMi10Fog9QGISdB0L3Ge1JfQxB2+3UMx3LezkHAt/ukXWayPqVrQeh+85vfTG9pL7jrtAkLVCKRaAdY5o319AQPyaQwdvJEuRmox02euGIoWooTYUi0D0jVMsssU5S32T7XDiX761//uri5fv7znzc/+9nPiguMKyxcrtxl3Gb6VLhaWUbCzRpuv9rdNz/XKvee5UO4HsOtOsydGi7V+blTawnX6oKk/n3sZ35S/3Z+Mrj/cOvWrl3C7B3uWAMRohAuXuQhLE+sTqxVrEbLLbdcIZ/aflQgYJ4/7sq2w3UjT5McoBOJxF9CgV26onfkiUXATHcS5IlliAXCzDfRPrBcPfvZzy7kiZJPtA9M5qxTSNVMqoSznrFkdoE8cWsiT5N0DSQSib+EuFCeBm67XpEnrhTuAkG24wRrgwbnxhG3kWgfxOY861nPKuSJtSTRPhi43B+WqZmsT8ftyMXMmth2sLQhT5MMSk0kEn8JBXbpcpanSbrUx06exJmIoRh3qQJuGi5D5I0LItE+sGY85SlPKXE2SZ7aCRYj5MkzNJMlVjz74thkGrYdguuFGHBv9g0C+gczQSkt1sa5nA26KKHNtJ021JZqnQlNiLb2TNThCbLCZetGaEJkgQtL6EKdtEWFmjxNMplj7ORJrIRg1HG77cS3IE7chhkA2k64L4sttlgJks44k3ZCuQVuO7FPM6nVJHiee/aXv/zl9Jb2wjUiT31zIVNOJjLHHXdcycwU+8b6Jq5PDKD4QGU9ouyIkiJ1OQ+lPCj4iD801otJJEqSREkPpIDUJT1IlJOJ8h4hEe/oVckPJKMWJUBGEfuM/9ofsW8Sx3S8KG1D6vI2rsG1uKYoceNaERzXrh3EXmoT7SPuUrtpP/GU4iURc+3LysnyLllHIo7kGEkxYnt4SyQ1ySiWyCJxRbu6T30ksYh8L8mTzCeZSuN226n9s/vuu5dMrK5bNcSdeJDFcXng5woEZot5Qp4M3on2QZ/jehM3OJNaTbIDKYYukCcZntbP6luRTPdYVqTlLyIjNDJBLcYq1mSFFVZoll9++fJKiWknmZh+5z/6iHstY5SlUmaqLFWKX+kSmaxKvMhuHSxbEpnQkQEdGc/Gbtt4ECjPwSznyHSW3RuZzrXE9sjURUpk6kbGbmTrRqauY9MZzsd5ycxFYJwzEiPjVBau60KyXacMVdZViREyZrWXDG8ZqhIttJ129D0Le2ToahftoZSKpCbHd07OWzYzUoXMGxeJpBDJL10oNjvbYHXTNkmexgRsn8vQw0FJdxlmHGbDHmizQWRqLkCm2xJLLFEG4L4prK7A7NmgP1PLE4VKOZndtx2yAlnJ++a2UxoEETrhhBPmFbJlOUGqWPDrgr8sxALrkWmJAKyKSrMgNsgOMsDbEKVBfEZAok4ewoBIqeGGVA2WBFEzDvGKsiB1GQ7vQxCXKBPing1KlPOoJYhLlPgg9osIkSjpQV85D+ej9txg/TrnT5Q4iXqAg2QQCUP8ohROCH0YpXJC/JYgkASZ8huCwKmDt+SSS5ayLSxofYNxB3lCSidZwLY35EnquQfVDEZqepeBeTPzGhDMjpmPmZOZmbtQfHB+EJdmUDDjNSAn2gcuCcpG/5tJ3AWlZQLTBYspQkB5muX3CSwd//3f/12IcqJ9QMgQO8S1y+P9TMHijZwjT5OcZPeGPJkpmdEw1XZxUODbFlAoroDlTDv+13/9VzGPm9VRSKpK+76rfnBm6Oc973nFvC1YN9E+IE8IkIBxFtBRYXbPbdKFGbOYH+RJba0+wXiCPLE2JdoHbm/kyTPYhWKzs40kT2MmT1xCzKxcBhRAm4H8mNWbnUvp5pYTJKdII+uZOAKxQY9+9KPLciZIIROwwZ6J3e9lbSj2yPSOiLC22dZmsDwhhAiU4MlE+yAQlsVT3MVMsu24SsSoCNJtOyxvw13EPdUW3Hfrrc3FK67YXL/HHs29t9wyvXV2wSXCutj2cbKvYJXnVjS2d6Fe2myDtU2cWpKnEYFYYNssSQINF1988VKVWsbHgiDmia+a5Un2RFshjVXbGLi1lcBGM18ZHdwH/OoCFQ3q4hL4vZGi2trEQiVLRmyKQEVkyxIjzPFiGGZS3HAcENT/nOc8p3nSk55U3JGJ8eH6qefi5ve9r7n1yCObO6b6zh1ThPvuKTL7+xtuaP5UzW4ReP1J9e2ZWJ7ElJg1doE8seSyPLWJPMEvvv715odTY8CFSy/dXDU1lv7qvPOmv5kdiLFxn8adEZ1YOMjUMwkRRN6Fkh+zjZo8TTKxqFPkCUGQIoo8POIRjyhBeYIYWWceqGAfxey4LDRtNnUidgiTlP3HPOYxJZvFLFAQp8FM6qwHRporQqRmTh17wuokHRZRlCXjex2Mq0TAoZTjtmZoOG/ugic+8YklyyQxHvxh6nm4dvvtmx9MPVdXTZHyy9ddt7lsqu9cvMoqzUVTA9T3n//85sKp+3LRVF/83tRE5ewllmi+teqqzZWveU3zgze9qblmu+2a66eeq6umZsO3TvW3O046qbnri19sfn3BBc3vrruu+UM1wJsxy5TqQsyTgHGTFM9aG3H3hRc2P95jj2KJuuKVr2yumyI9swHB0MIB0vLUTgjNYP21fmjbvQmLAvR3kqcZAFFSofgZz3hGecjV0FiYGB9uK35iAeNtzU5DALkXkR7k0PUx0YpBUB8nZvssR9ZWM7BzgdRBg75XL8nDhTBFJWfbo17JTNwt4wC3nZRnLkmzqkR78Md77mnuvfnm5oKp/rb9C1/YnPuOdzS3Tj2HN7///c0N++zTXLvjjs2lq69eLCFXrLdec/laazWXTBGsQr6mCNf3ka/ll2+++tznNl9fdtnmkrXXbq5CvqYmANdsu21z/e67NzceeGBzw777NneceGJz11lnNb/6znea311zTXP/hBSEPmis8qy1He7DpVPjhja/aYqcIsQzhcmpEAeJKIn2wQRaqQN6wfu+gauS3lOqgE5cGP2/KNDJmCdEgCVJ+igXj/RNBckW1IjSax0XoWgrxDVh0jLOEAiuO1YlrhLuAy5HgbZIFrcl61KQJzMQ5IP1xowZ2eJ2aCtRGgZp0GqgPPOZz2yOPfbY6a2JNsEyR9x2oxaO/NPUMytGZ/+p/35kiy2a66f65m1T9/jmgw9ufiwOcaedmh9Nbb98qt9esf76zWVT5OvSqb5w0dQA+f0pssVFddFyyzUXr7RSIWmXvuQlxdJ19WabNddss01z3W67NTdOkYZbDj+8+enxxzd3nXlm86tvfrP5+dR53j9DF6E+2BXyFPj5GWcUAnvRVJv9aKpdfjN1v0aFSVmSp/aCTmAZZJ3vo+UpyJOaWUmeRgTLC8KARHHbmSkJNDagz68hlSpQp6PNsTSuCeF56lOf2vznf/5nqZMiQFdRTzEIan6oj+MaL7jggnkB8OJHWNbUUVFjRRyUAm0Gf23UFSBPFDOropoxifZBpqp7NNNCsxTzTOo8IV/33X5789srrigxPj8Wuzg1SfjJIYc0N+y3X3Pd1Cz8mi23bK7eZJPmile9qrl87bWbS6bIF5KFdH1/inwhYRdPDbi2XT6lfK7cYIPm6qkJyDVbbdVctsYazY3779/85LDDmp8ed1xz5+c/33xqaqzacb31mq9xDXSsltpvr7yyuXZqUol8IplcqQsDky1hAqzeSZ7ai7Wn+jcC0cc6T0GeeCmQp0l5kjpHnlhnBFAjB6rFKo9vQFeQTAzQ/DoT643jHnnkkdNb2gdVl8U2ifsR68R9J0Bchh0rkwWNw0UnQB5pVM3WNYnlCjef8v4KTQ7GQ7UdCCFz9NOf/vQkTy2F5Tj+FvLkGVQ9WU2ysWJqgL3vpz9t7rnqquZXUxOpO08/vZCkm6fIEtLE5YVEIVNI1WVT5Or8KdL1tec+t/ku4jVt+UK8kDLkDElD1n409Yxeu/POhcQhc0gdcofkIXtIH/I3Cfzxd78r53fJ1Hgpju0nUxOrBYElwz0iklQS7YRisyqiz6UVJhYWQlGQJ9XbkzyNCMwTSbAMwMMe9rDmn/7pnwqJMKDPL3Bc0KdZb5tjaYI8OU8F+lTrRZxYnAR61yZaM3fkUVo/Bu4/SJR1l7gouSdl53XJbccVizw97WlPS/LUUogxROpnut4bVzv389jJ0wzgWWSFiYBx7r+7pq6bO5Bb8PapZ/SWqWeUu1DMFvchNyJLD5LFvcjNiHRxO7J8sQRd8qIXFbck9+Tl66xT3JXclmLHuDFvner7P5tSCr+cmvD97NOfLu7O2SBfrE8In8B/ljrB/IOQkKO6N+U8F7LtWOnphAdKKOoa1EsTv9uFSv2zjSBPq666aql1Nal720nyNBOwVgUpaSuCPBm8uOEwai7KYa5IHUZNJ2QQ0RDrFFYphAnB7Nq6R+pSsSg+5SlPyZinloIb2aRlpuSJ69wY0IVAV5MRY8ZXv/rV6S1/GwS+C4AXCC8gXmC8APkbDzigkC8xSgLoBdJfMUWqWMMK+Vp++RJwL/BeAL6g8MumJhmXv/KVJb4JUROw/+MpZSpw/Pajj25+dsopzS+nSJ+yEwL9BfwHfj11fIRNBuUPNt20uauqoG5MsWyK5Jpxj9GzCWMmL4QYPV4HMaKyB3ku5gLcH89RFyYhsw3Z5siTUj1JnsYAVilB11xfbQVGzeWm6GXXl5CZCVROZ4pN8tReWAZIMsJM3XYmBtbv6gJ5OnqKhLCUUb5tgJIPrEVKQCgFwZKkNARLl1IRSkao/3TV1DgX5IuViSBKSk4gX0pQiPES9yUG7IIll2wuXGqp5o4pRcQta/25Ltd5QpxkFYv9lM4uAedZz3pWsaapkTQXCBRSb/27PmbbBXmSXIQ8Tapu4djJk9gcJGbc5Mninti6NbnaCmXnVdYWu8RVMMziNJdhYGPVEPNEcSXaB4M1l8FMlywRu2fQb3Oh2kDbyNPfCsVOFT1V/PTG/fZrrp5SwIVYTREnli11o1hq1INzj7ta54kFX5X0SJpB+Fm1hQOoMyYzu63lahYWDBDuUxeeo9mG+6sWovvbO/KENY+bPJ155pnz1gNqK7jbzOgpGC6Drrnd/lYgjPzYsg3bHNjfZ4iJoYDE082E3LOqqmCd5Gm8+NPUxOzanXYqhVAV1eT64yYUW1VD0gbLoDpxXLRdhHGExUkIBEUrnIEV6glPeELJYlbKJerfdRX6pVjYPlqegjxJihL7PKm43t647axAbdCfZEXSBwJlJCBX8bM+PhjiaLjt1HnKIpnthOfW5MeaUjNxf7CqdmXGLJawy+Tpt5df3tx04IGlaKmA9SjH8JuLLpr+xV9DuIAx2tJOXSVP1nxba621mtNPP724HilYkzIxMpZ/4r5TusbktKvWfTq0r267XpOnSVieVN3mx5/kQoILA6XnFeVjIetbATT3aKWVVioDXMY8tRNcHuKW1CObyexdORGTAzEpbQfyREnNVsD4OKA+laV2Lp0iCuKbZP/dNnUdf5hSOAsD5InlSVyb4PEuQvIMl44QABNmpV/e8pa3NJdddllJbfeZW1KYQJdKudToc8wTvYg8yfpFniZVy7A35Olzn/tcs97UDKzt5KmvMANkFVx++eVLcGfGPLUTFtjm/j700ENnlCatsOvOO+/cGfLE8tRm8nTPFJm95YgjmqunzlNJBLWclCG4bYrczgSWhzJGU0wWEe8iKFPjPYLEAqUwsv4mzsl3xhnWp/32269s76L1qc/ZdkGekONekSdVsJmExx2MaMahpH2bY576AGml3D3iEJhfFXkzgDGvq2u1zDLLlIDxjHlqJ8Q6mc3vv//+pabYqFDY1RiQlqeZQ52p69/xjpItJ+D7yvXXb35y0EGlDMLfCvfXclfcXEIIEu0EYjiRYrMtQJAnNQElgE0qe3Ls5Em8g8Jz40yDNePQyIIIHV/mnWwhg6L6H2ZbCjRaG85yL9J1rVzNbO08b7jhhrJUgawNZlIzbjVEpEyqp4QI8J+7ifyvCEJXfeng3GUwuB4ZgK4R0bEMjOvXDtqD9VBchEFWm2k7bcj0L7tF6QHBm2KZxJyx+pkpnHzyyc2JJ57YHHfccSV4k5I67LDDilVCvNN//Md/lFmh4GTH1LZdz46ZK/C8vPWtby2unZk8w5tvvnlRzvpQ29E28nTP1HN28corF1E1/I6p52hqsJn+dnbg+bUsFPJkDEy0Eyxnis32sUhmb8nTJNx2yEwsuMuqwR9ubSAkzuAoDkNhOPVNxGNQDFi98vdSXtWUUM37kEMOKUpeFod6TEcddVSJzRHcLAYEGVBqADFQT+pTn/pUubkIA4sX65fzQCKYlQU0ivMhyEWIzMBhEt/7ffzXq30R+yWO4ViOSRzfeTgfgrw4P+cpuNI5Kx7q/C1GHISG9ce1umaLUGqHd7/73WVtMvdR1pRlcbbZZpuiULUjNwfLhIdbG1vKw0BsEUcuOQRW4KaOzwK5wQYblP8onsitalmapZZaqmRlaXPHdA9YOrx3D4444ohyfs7XtbhG7YAUmzmLGaPkETjV1hFj6+aJhbjkkksKybOkD8J31VVXzRPKYn6CIMYr4dIIEQeEaBNkkrCsOpb/eG+b7/2W+J/9OK7zEI/h987RuTpnJNT5u45zzjmnXBdF7r3rREpVntcn3H/3XVu419ol7m3cU31Vu+m37m0t+vMwscRPLZ4PrhDfIbajQuyJ56wL5Em/d60zLQi6KPDrqf6wKIE8mbiooWMJqC5PAucy9Evu7z7HPGkDY92k4tZ6E/OkfgnFLA2edcPis+p++PzkJz+5WWyxxZonPelJRZ74xCfOE6mtRJrr4x//+CIsI4973OOaxz72sUUe85jHFHn0ox9d5N///d+bRz3qUUW8J/Fd/Jb4r/3YH7Fvx4mUWhLnEedWi/MmikoS1+KaEETiGl2rGKJnP/vZRRZffPFmiSWWKPLc5z63EEpkxcLKSy+9dAmmJMsuu2xpL4SHyIJDemQ4iIfQcS2JgwAhPUgQ0iQoHxlSbgGZoiwR07e97W3FXYNwCXTUDywvYJaLpBqwPRAWNkaOZGXxaRvEBZE7lxVXXLEsRWMbUuZ7BC3OQ3q1c2HSjvMRG0DUFtPvCMJMnOegIHHE937rf3Fd9otwO4ZjOabAWsfXHgii8/GeOE9thSgS5+tzLa7DdUW7zk+i7WMfCKljcEXHPXB+ztV5uwYElGjLEN/FtXqvbeLa7APpjWtyLc7b8RzbpCPuARfrTAYtJLsr5AnJ1J6xPEsfgLh7HvUzZD4tvu0Dr4Bn1NjZx7XteH3oCuMTY0CvyNMkimQarCkcSoUlgyWFUNbxSqwJ9573vKf8hkJnZWEeVZ+K0jfzRgK4Hrbddttm6623LuvPIQniOSgoCktGEgUcyimUbijcQYVKOREKl6J0rggLoaxqoaQJUrHccsuV9/X3/uP/xL6I/YZQgiFx3Fr8L35rX44T+4n91scjfhMkh3WJkiXe2xZErP4+zjvO1/4pb4obqfCqzbSfPjNIeij/mgBoY/9xHtrVPuIaHMtxtZe4KkQRaUQgZfchmIim74OQIqC2IZ1+47f+Yz3BIJrPf/7zy/6QTdu8+my7721DTP3Hf5FVEgTWvpHlIPSOiQAHIQ5SH0TebxHtIOCI+b/92781j3jEI5p/+Zd/KWs9/vM//3NZ7/HBD35w84//+I/l9ZGPfOS89w95yEPK9w996EPLfx7+8IeX7+0LoUfiHU87OB+E2zm7HvfQvXMtM7E8IU+I9EzipcYNljr9jcWvL5DCb7zzHEkOmFQaeGL+EC5irOvKGpGzjSBPJo7I06TqIfbG8iQuCUnhhkKkxPEoC2C7uB43AIP1GmI78Rvi98R/iVRtHZmICWJOJG4uxULMDPiliY7OzEoEzIY4H0KhELE+RFxRyE033TRPrHYeEjFZtWjbQfG7EG6kWsLVFK6ncC2FeykkXFbhyqpdXkz8hAuKcIuFcEkR7jLCNaUYn8GZS4qLjYISh4YIUdYscggVghdkjdKmvGvyEwQE+QhrG6VP2SMtSAxCE8QtyFu8J74LQZ5IkKAgQN6z9Pivc0HK9CcPMMLHcuPckTrkGZFGqrk0wx2MfBvwzBi5P7khDQJqeskIYnFj7eAG5mrjcuN+43blkuOm5aZTSJU1RP0h6dZi9rj3fBeuSe0d9yLukXvovrkn3DP+xw3ILcXdy5Xr2J4RkwgTB+e95ZZblutC+hFR5A7B0hdGhX2ZdHSBPGkLY1WfyJO+ps965tJt107QDfplV5Y5mm3UlqeZlkyZDfTG8mQQYOkRx4DoJCYH94I7gIhHY4Y2w5VyKqAfCWIJmkkgoH0hwchukNsgtCE+R7B/EOggzxH0bz/OL5XHX0I7iXdCMCUEjArkiXLuEnnqk9sOGRdfaKxEthPtgwmQkAghEF3IWp1tGMORJhM6cbvG70mgN5YnYDkw6+1jh+sKuEpZklg3Eu0DYok0cbFKShgV4t+SPLUXkhGQJ24h1uhE+8BiL1xEwHgXYgdnGzw6ltsR0sEy3yvyNAnLExgIxVv0scN1BcyxrE5cd4n2geVJLBD3ncy9UWHQJ1wPbYcMxb6RJ1m9yJPrlu0ZIQV1mAGxvM6gREhCLUIVQvym/jxMhu1jfhLHFb7gHONzLYPnXUtc2/wkwijmJ0Ik9OP4XP83jlGfS33ucb0R0kGEeIRE2MegxdxruFa32267EsbRN2gXekL8cJKnMUEMCunCwN1XiLcRmC1QOjN92geWJ64dbh2zv1GBOLE+dYU8GatYY/oC1kSxbqz04vYkxhAlSSTKsHYQsXCEm51wIRHxe0R8H5FoQ4z79iNOx3sS39lG7Mfk1v4dz6t4QdsQBV4DpCH6kOQDWb3iDr1G0o56cSSSd3xHEEIKVzKPz773O//xX7qBW9n+Hcfx4pghQf6JTEz/i986T+frOqOtom2iTaItlMKJbGPWdhmOSKtEJQlLJGIixR8q20K8566S2KNvcuH1bZxEnrSDWFNxmkmexgAPu2w7M4ZEO2EAQZ4EfU+q+Fli/kCeZGRRWO7VqKBoKKgukCdL0EhYYI3pCyQjUOYSApRaqbMwZXlGCRXi87AyKnVmKJH8EaVdZIgq2WLfkSUq2zOyPn3vmL6LMi8hUerFfqLMi/O0Ld6HDCv3Qmx3jnF+flufm+PEddfn4Xv/j5Iy/u9/tpnoRYaucjF1yZgoieNYYVGPzN0oEyMTV3JLlIupxTa/iaxgYn/2g/SJS+sreVIuBnkStzoJ9Io8mRHIkpJFlvhLCJY+/vjjS8FFgdSTgkKMBggD0STPIzEcAull6Ulllzk4KszQu2L95ZaURakgaV8gG1csCcXEqmKsltWk/Acl79mUwar0h7g3GaesVPpD1BsTL6XOmvIhXiMTlUTNtNgex4oSJfalxIh9O0adYSvzNUiG97ZFJi2pM2m9xv9DbAups2vtyzXV5UWitAjSEmVFXHuUGvGdz15r4lPvK87PdoQnvvMapMi+7BsZQq4QLmSM1GTPdr+T6cti5rxYovqIXpMnD9EkyJPOpv5P38mTtpeezvcesxaZaczOlJvvJ5VlJkjXIGGw4d/vImQQalsWGi4ucQksNnMBSLbK5RQcN8So4H7hblF+o+2Q0dM3t53SJcZIFjeJAUpfCBwXn6NsSxtgbPKMeaZk6EaJmSgrM6ykDInPJLJsSV2qhrXbPk0SHGPSVh3X6rxkiIuhQm6tJoD4qU/YRwR5QryRJ/dzEugVeaKYDfozqU8zV2BQ0A5mP9wuBkUPqNR9FgEzQtYnlYbVAkI0PbzjgjgTMzoZdwhI12AwFq+gUOWDHvSgIv/n//yf4goQp6HeUpfLH+g/lo5hjZgJeULQWTSSPLUTFLR4I2N0on3w/Ankl7AhHqqPiIDxJE9jhHW/FH+jwPoKiltmiAKMAn5lKyAs4sGQFn5+/ntmZeZ3bjTZH+NS+M6F5UnRyy4G9pu5xtpzkVXD+iTWTnVvcQqTethnAwZvxJq7RtDrqOgieVK8tS+QKYbkc8G1xdKU+F9EzKGAccHlfURYnuinJE9jAoXGD67q8mxDGiml6Wba/7BgZxYc5mMm4vmREaZiqbeyKPx22O/8htnZ95TZ/OA3Kng7J7OVGAztk4XEeSq2FoGOAiNZpBRBVEuEhQ4BcAwkQHCijhrnxLzNjIxcOafZQJAnlqdJ9JHZAnO/ttP2MtMsmyKOgcl9XER0UcB9VsFc7KDA4lGBPG2++eadSLFW5d1Y1SfyZMKC3Ion6arbfC7DmGsyZvIiQ6+PqMmTIpm9I0+U7rgh0JUv37IDswWKkCVLEJ/MCgF+ghPV4kBeAGlykx1bgKBgRFafcJkFuM6sr8efzfIi9kAQN2Lmd4QbTTqsQET7MjNGjDxUNQyCrEmIiHPyavmP2JdzQqDENVjWQ9uId5LCq31qUmbfSghQehZYju/Uv6FEuf8QqNmAIF2BlgIkZf50EdrXvdWW3HZ/93d/V+655VTq+91FIE+WfuE2OPDAA6e3Ljz0Ieb2STz/o6KP5IlF0FjmurnwEu1CkCfhJ0od9BFBniQc0KuMCJPA2MmT+hfiJSZRvZbiZ3kyc54tJUYJqLshG8VK80GSxIOwPLD2HHnkkeWazdRZvw455JCiTKVCx1IxzsfaZUiOeBmxR2qQIBJHH310IVaUloVKKWVBu/bl2MSacUFqFFxDnJAwpl21W2S4CNR1vjobwmNRRb/1QLKGmclQbNqptiT5fdQ7UebBubKsUZ4yRo477rh5RPFvhXgsmSRkNknuuBGEWb+I+CcL8qoPwzXSVehjrJIymgxgo0KZAoq5C+VC+kieWL25mIn3XQWrmfUijaFdjJ2cH7jtwgjAENFH9JY8KRjGny6rY9ywICryJOB1NrIo7ENcBAWp6jIigiyx5lio1XtZbfzTMv24KpAU6+upKcJqI36EQvJbqd9ijszqLZbLykTZIFKsPCxOCr35DlnhRrNvtVIE0IWZncVGOi5yYyZJkTuORWCRMIRNcLiZSwwsyBDyJF04zinAiuUcBJLGbNT981uWBIsJz1ZWivaUtouUIXFzAdoSWRBn5n6ZNSLCXYT7zKWMmJsEjArlQii0Lrnt+lSqgKvcs26c8Vx3FcYpE0hWNOPyXIFJLf3F88AQ0UcEedJPe0WeEAS1PiZhtmedQUwE3NXkYKZgsUFQuNfMBiIdHREhvpdOyjIk5df3YpCQLUXW/vVf/7VYo7i8gtCIjWFdomw9KNxlgtwFbqsRwlqFJCFB/qs2iEwudVHs2zFZb7gOxdcga+DYSBTlZ5tZi+DlKLLmWJSheitBnsy4ZY65NveM5QkZi2Mjolb75/6bLVBY4p3EXhkk5hJY7ATriylhPezioK6vIPUsotzAowJ5EmfXBfLEoto38sTapCq3SRGS3EV4zoxTJpssvZNSrosCrs3Ey/M3k4SNuQD6spfkyQ2fFHnirkMyzj333HlE52+BjiwgWHySGY7U/jpQ3PcGYERAqX7EkUWIa4wrTVYLC4s4J+SKGZJrRxVb2+xPgC3ypJMI4vRfhI3y9QAp4S/IGpHRrixeSI/CbALqWJuQG+cScI7vfve7y/4cN4gk9yJrmP+LjWKJouiRTmTJuRpUnbfgZ+crqHy2rE7AOoM8KV6HDM41IKnaXD88/fTTp7d2B+41q6P+5V6Niq6RJ89Un8gT0uEZ99yz1HcR+iirmTAFY1g9Js8FGHNNjvtOnoSipNtuTKC0uE240mbrgXIjWZfEMLECITEGHnFKSIeHmHlVGQDp/64fKUJoxH2ZHfmfGCkKlfWJIFwRNI4cMUOzJMmyUEDS8hg6jgUnZRuccsophfggNNx6js/KZt+IiNkkq5V4Fb8X7+Q4gt0jvumss84qx+RPZ4FyvixeLF3O1SyOxcxSCoKgxU0xoc4mXJNzQDIRv67BwM0VKmFgmDuT9Q9xFtvmtWtAwrl2EOmZkD/kyUQjyVM7wUUvVMAY1lW3ufFMvCT3sLin2ZzctQEsv1z/1snrIwbJE0/IJDB28uSGU8yTIE/cQAiKVPzZanDKhJUGIeIusxCk6zMzR4iYwREfZQE80BRrWL085MgPkiVjTgyR2CUzCwOX8+VWc64GgCBcfq/9aouSGCjK2u+99x/H1Lm0OfcDgmdW6TcIUcRDBXRK54x0GUApd+fr2M5VoLNjI01mPrL06v/PBlgzxDyxtnUxUJcV74ILLigB7//3//7fQpK4R7k/LbiqbIH1srrqttPfEB+TARbcURHkqQvByEGeupr1ORNwK0ctLmNaF+EZFHrA8m4SOdfAa8NrYOLdR/SWPFlFmnVkEuSJG8jaSVxts5UdFkAw7FOKus6NuFAyYpAWBpSJB4IVC7GhpIIYzRT+LxCdkhZDxQyv3edXZ8o1OH8kzCA67HcIGzIm/sm+ZntWx4Km5IOASLPGLsLgrQ+wFHKxylwyU0ScKCX9D+HuIvQHfdX9mUlMGvLECtoF8sTiy03OItsXGLNYbDbZZJPOxhzyKph4scQjTyarYkqNhXMBnh1xtJY66iN6S57EzhiQJkGezJRf8YpXFJcKEtAmsOqo2YQ8zbYrbDaBWCGgAtmjzMJsQhV4bknHULoh0T6wliK4JgijokvkieVJUCo3f1/ASs76zFrKGt5FmMTKfo7lkUJU+Bf20PV6azwGJmEmsH1Eb8kTlw+31iTqPHEzse6wbrSNPEW2Hbcfq08bwcqkaCeFUmcXziaQJxXPxcV1MaC6DwjypCTHqOgSeeJC5rbrE3lybyW2sD5JPukigjwFWXL/WNRNnsVzqrk3iYSl2QIvAuLA+tRHBHnybPaKPMnyohgnQZ5YnPjBKei2kSdERIaduCfuxUVBTP5WcDWZ7Qh6F1i6KGZv4qwik4/bK9E+BHmayfI5XSJPCtFSUn0iT0gG74C4NBOkLkK4wWmnnVYSXoSJRFym8IU99tij9F2xm10F66DsbdanPqK35EnFawSB+2fcENCNPM12baLZghgD53fmmWfOq8/UJjg/WVYI8KJyLSJPMvq4MN2nRPuAPFk+h/tgVAhyFSzfBfJkYO6b5QnBMEaLeeoqeWIhl3Rjko4oSRByDxEpiQ7IOwLSVXj+kECu1T6it+RJ1hGz6STIk1R98VZnnHFGK4MHzZDM5nWG2Q7Eng2IgVCoUxXw2Q64DyBM6lxRsF1M5Z/rYG3k8lCLayZxi10iT/q5iZ7xoi8IxcQl1FXyBBJ1jFd0jXgnpVVMyiS7dH25FufPtco62Ef0ljxJxRcMrFbFuMEcbTaibstsp9j3Aax1UeZgUZE75vZHP/rRpaQCN2uiXeBOPuecc0odspmsPdg1yxP3sVjJvoBrXpar0iZdJk8Bma/iSV3XXCmWybWq7AkLWh8R5Kl3AePWeFP4cRLkSbyFdHFFJBe2hEBivBCPpg6SOlMC+xPtAmUkkJgFUtHZUaGGUFfIk/5ngOZG7wsooqOOOqqQxq4GjM91cK0yBCiB0kcgT3hE7yxPUvFlQLBgjBNidFZeeeWSyaUAIwtUZCxYLkIp/2233bYsfivjTVC0lF1mXuvAKYWPfMV7ZlPiM/E7fmj/Cdltt93+QlToFuhn3wuSwf8R+6N4fO84xDFHFddof8O+C4n9O1evrtciwmIGPLSWPBD3pIaR9pJBqTN/4AMfKIsef/CDHyzrnlnuRZ0VsWbWrJOpp7Orbo4kcdHJqKOcxCSozeJ41vyzBIz/elCYqZVyEGMj00Sygf6jOrraVZbdUdCPmd5s+bzzzivpyOrzsHQqcmj/IbaHsCqEOA/CTcPq5ZUomRDifEMEtIewmBEKV8yI/9m/a5LlIzNNIoAK7wr4KYLq3IlFn7nAEAqB+GIaDJBmy5Ye4GI2a45K8JMEy5PrUUl/JpYJtWm6Qp70UZmlfXLbedaMD+qS6auJ9sHYYMzta7ad6++l205GmSVSxk2eKCfkiVVD9WfrwtViJs0VoTK494oAWjl+hRVWKNW0KQvZG96rUu4akECWLH519aOUYFAKQdC3QRc5c4NDWNx8F599Pyj+F2L2F+J/juk4jiF2K8R3w6T+f4hB0f/r4wyK33m1BEzs36v/uUbEUyyI67H0S7SFNGBto0K5ttZmllnRjtpTu2pf7bjkkkuW+yCAU0Vxy8hYlsV3T3rSkwrZ0tYWUbY/Be+cu2NGe0c7OFerwFPKiLAsFJYrBMx3SC+SR3ET722L7VxJ/uMVQUUwlY2QWYjwEvuKpXMQbb8XsCmwXVo3cWyzQedocWeuD23pPLWl64h2QuBdW7RPtI2AfG3y1Kc+tbSHmlcIvwzEJz7xiSVuQ3v5/IQnPKHEHmlH7WYf9qXdHYtoV8d2DvpXtJPzdS3aw/JBJgzaAilGOBFj1ylOEBkWdHvMMccUQardz5lUGHe8rmTbIffuX5/qjXHNmzgIrehqkcy5Ds+Occ042Ef0ljxZuZ/ykA0xTvATUwwaXDwNBeI9IkDZUTSIACVkGRPKgfK23fkGSfJ7Ctug6v+UJ0VpFiBDxX4F8lGkFFQsskkREUqYUqagCWVCYVPcJBR6iM+UG4nfhPgfsY/YXyh8xyFIBAml73zinPyuVvzOO4SCcz3IgGtzjRSv63Xdrj/IVRCrmlwhNwiW3xBtRwzKRHuGaH+ChCGrSAVigCAgUkiDz49//ONLMPnjHve45jGPeUyJjXrUox5VCPEjHvGI5uEPf/i8V9Yra/B5JfFdyCMf+ch54v/28+///u9ln/btGHE8r0iL83A+zos8+clPLiKjB9FBVBAbS7Igh4iQ/oQUIeY+14Q8iJPr1ceiCrn28VlbRl9DfJAx94B47ze2B0HT3trYPuxL33V+jhUELYir83TeSFoQM9emDbSJdtAuttsHEhfX6F7YhlSNCn20K+SJRVHb9qlkhlhQVlj9xzJDifYBeWD9Nwb0EUGejH3IU28WBjaTZZ0YN3ni6qH0uZa4T7hRzJy5VbhyuGu4YrhemOu5ltwYhfJYQSzVYNbNFcUlRXG4FvvjsjJbl0nIZcOkqnNzcQns497jDuN643Yzyzd7R4iQoyBCQX4I4oPgIC5BgAYlCBISFP8LQkbsN8iX4+2yyy7z3ILOhxuOS8451u441+F6XBdLoesURKqquIWCxURoi3DHqcRMtBXRbuTkk08u7ciVZTYre85snjLS1mb02p17yz3wG+SMQkcsXDsih+y5XteGULoG5++8tfV73/veedfkPN0b58paEuK+OW9Sv3e/SGwnfu//sS+vca+5nT243IuOqb20nc9xXrbHPdb+cW/dKyTW9SCsBj9kB/Hx3mwSWUI+Ec4gQaxuiFBY8nz2DCFhtrFiIVLIp/8grvZjf/arHZH6eHUezkm/iHPmlnX/WZy0p1eftTG3tN/5vT7m+O7TTGJi/B956sLCwPon8mRM6Au4ZS3IrU/NJKYtsejBEGDcps/6iN6SJ4rIg3n11VdPbxkPZAZRQBT1A0EmmcBYQeVmYmJOmLPvvvvuYiKM7A1ZZ2KpxOUQN5Xo3CERr6NquJgdM27lCLgRETqB81yYyKQ2EcfjXC19oZAb03nExtTie+K3/uO/9kHsz37t33GkljumeCFKy3k4H+clvsY5Olfn7jpcl+tzrTqmsgTaQO0psTcGWO0z2xl32pKiVkPI4CD+xzmqyTLsPP0+zjXa3jkPE9cxTOyDDG53jwcl7jmJexv33fs4R6/EeTv/uP+uh2h/98H9qO+P1+gXSnmQYf3De69ivcRPiU2JCYH4KdmkrAcIKsKKvBpkxKMhw4iiCUB8RpQRSCRQuyOC4cZDwoPIIz0shMiT380k2w7R7wp5Quy5rPtUb0xcnb7EGqp/JdoHY4yJuMlRH2G87SV5MqMXDzNu8kTZGPQnFfypPg7C0Yag37YCOWGlE+PDKtbGKuttgj6lP+lX2grZR24RfkQX4Ud8kX4DTBD9mvTXxC/I3iDBC2KH1CFaCBRL3EwKDSJirIldIE8mWshTn+qNmRBJvGDhvOyyy6a3JtoEzyqrNqtoH2HiyivSO/JkxsvVgMyME47HtTGpWSTrh2Mjb5Rd4q/BusPyJM4Geery4p1zFRFEjTwhX6OC+7Ur5EmWJvLEjd8XhOVpEmP0JGCMIUgjcf31hMRExITExCQmJ/UEJSS8EzFhiUlLTFxIWMFrS7aJiwmMz7WF3P/sI/YZln9iQsP9zTXfx8l4kCeWt16RJzEzAljH/WCydIkJEYMzCZilM7WKG/FQJP4aBg/xQgKSxV0l2gdxgmKnDF4sWKNC0oLkhC6QJ/GQyFOfirUiDMp88A5wAwsd4L5TWoMoCSKQXDiBkiDk/PPPL7Xz/E+8FNcx4hmlO7SfVzGQ1gsUH1nHkNIJEUPKjaxvibvjSq7j7iJmtI4ljFhPpDwyYPWvOuElkl0irhD5J+5tnegyLMklklnqJJfYtiCRVevV7yNJhtinfTsOie8d3/k4PxYV5+qcPWuuw/W4LtfnvdAXSU3uBXLVJ/SWPHlYBLrOJF7ib0GQp+OPP356y3iBPIkjETCMJCT+Gh4CA6MMN1mZifbBYC3YnULjPhgVFFxXyBMSYHZP8fcFFLGECVmX//zP/9w8+MEPbh7ykIcMFd8Nfv9P//RPzcMe9rDyGtse+tCHlszXkMh2HZbl+tjHPnZehqtJFJEFSmR4SiaR+SkuMrJcZY4SJT6i9Elku8qcrjNeo7SH8jTIhwxrmajclMTEflBY4RYkw37juI4X+yX0XohjOr7fRYau987be+fp2lyna3bt2kDWr7axDRGjU8TA9gk1eTrppJOKlW4SGDt5Eqyq44ybPAmyRZ7MdiaBSZAnJl0B6lwtApbbDhY5AcUGU8HMifbBc2SmzyIwE8tTZBqKqWo7WFKQp0lZqycB7ioWJ7GHCAZlrg1YQMKC45Wwjsi6ZN0hLCS1hKXHbwiLCisPq499hrWntvJEHbQ6wzRqxhFEpSYjQUKCDBHfRc2+KD3jOohSG4hJEK6oL4ewOA5CFqQsfkuQMr8hyFdNzkhN0IhtPtcEjTjnOH/XI2vWNXllhWKN0kbRbtqVpclzQ3fI2pVFzRLHSue+9KmUBgR50o96RZ6YaHUumWLjhIwlad8C1icB5CnS2cdlZuWbpwAMWjKvaiBWOiH3aVv85trF4GD2aVHWrkI8BNLKzaHfiWeY7czESUEwuZIXLE9iNkZF1BXrAnniiqLkuZv6BBZgVkWuMUqb2068z1zpw8MQyRddirNUKkT/VB6mT/GhvSVP3HaK9Y3b1EiJmdWo0zNu6NjcFFxS0sEp13HAcWQMmdmo2yNbSjtIhxeoKMVdYPZM3C+LAoIigzx11VWCjErxNys1gzV7NVkwgxR4LMuty2BtMuslM7E8RdxGF8iTeB7KyQDdN5jIIMmyI8edGZ1YOIhHU+tJDNlcJraDGCRPk4ohHjt5OvbYY4tJ1o0f5w1XM4dpWFHDccN1Ii4GIu4OFqFFBcei1FicBNM5Jj85MzJFQHEhUoiToE4mYrWC2gADtpmu2IdBS1lXoJ9FodBzzjmnmNQRDQSKG8Jns/iuAiHXhwX2z4QIhguCBavtsHafZ8a97BvcW8ktxNiVaB+MNSZlgu/7lHUX5EmAfe/IE/+u6rXjJk/cdmZT44brdHxERixP3dFZpaSfskyRYVYpv2GVoXAGTZT2hSypzxOptZS2bAyEie9eDJG4AkHYXBCIlWPJktEBZRW1Aa4RuUOe2nJOowIx5vbg0mKFUhRWDIeAWIpYltI4+/1sQ1+UCcmKKsV6VKhwLmi8C+TJBMM9kyHWNxhPIrZGanyifVB/TWyZON4+lb/pNXkSKMfaMU4lwlUlQFHF5HEjyJNA26OPPnp66/+vaLWDFFyDNDOkQE1mclYYJOj4448vg5dYMcGaUnoFnNunOCpLpJjNCxy0bzNGLlHLllhORLqvasHeI1bhG/ewObZ9cu3ZH4UvvmFSweWIIeUqA2dhKsG3FQitlG330xp5iJN25gZClLsO1X0p1pn0EzNl5Qq6QJ4Ui3S+Asa7THhngqglxMrYhWSTPsIE2HgpCatPBYWDPNGZjAHjiiEexETIk6wGM/BxmhoF8GLpsrnGDdeJ0Ij3iFks4sTyg/ToBMgOqxjXDguRmZ8OwlomLkkMjRRVxNP/DG6WyGDFk4Xht4KsES5ECTHSqbwijWK96tlJkCdZHdanQ9jMMtUcEdg+iZgUD4UsHpYnwbpdBYLKLU35SC+Wzs36x0WqSGrX63zJ8tFXZxIr5/7qr11QyO4Vq6HnY5xjVRtgWSD3SUV5z2WifeBxcI8kQc2FSdnCotfkiUuJSXycpkYzXTPeN7/5zdNbxgfXqbgc64PFRs1imVwFRwsoRlaQHgOVlFdxWc4XmVELhftNnIhYJWSJ9Ym5UgqtfYhJYLVBgJRDECBu1hwESfrrYKxV/Z0Cc5Qh4kTBS6O1/3ErDARXeq7aLuJNugxWPgRYVimLEwWMuHvg1dGZSXXutgC5FwszE4Lt/upvXSBPKm27XyYl4xyr2gClZIyVSZ7aC54C5MmqHcPCPeYq6E4GBWUukCfhHpPAxALGDUzjHJAoMhVpmTnHjSAq4SIzS1B9Vx0QlieDlJgk2YBIkkKEzpflQqE6Csc6U2JopHmzOHHnKRbHZcdFxFKDbKk/wtWAfCE//qfeCvLkAePWs1aXAoCOwwpoP6xTCNPZZ59d3iNT47aQyATk6hLg3nXyVMN9MDsK96t7KJuwq7NFcVyepZkEEssOEvvXBfLkmfXsCMjtcpD/TMByylLPyjiTrMrEogfyRF8I3YhwjD5AuIoK9PRU78gTU7jZ+Dj9tPzD4SIbN4I8RTl5dX/M3hVMs/q9VfDNbpEalgoWIorWGmDKOiA09oFQiYliwWDJYrFSgI2liDWPJUp1XoO937NwKWqIYLEUeNgE6iOQTL0Iiv9LqRd0KNCZUhMMzCLFEjROIHrrrLNOWdsOuZ5rMMCx1pg1yVYTB9dF6DuI/UxS2D0D/qsvtx0x4TFm1VbbPsDESn24ProsuwLWaxOxLtfEmwnCbUdX9I48ieNRxXacsznxGervTII8ITEUDTJEcaqxZPbOSsTaM0giKVkPBhKEZCFbwDXn98iQWT/zJQuW2lnWhUKKkCHECkJZI1tipvbcc88Sd8W6o/0RWFZAyiziV1ibEDzVbsdt/TFgOy7yZA21LkKfZiGkeJHVwRkhEo+csjZ2dVkF1jPuAskFo0KGDFdzF8iT50iMlkSMPsWUAEu2cWKuVFc3dkoaMg4bW+cCIWSBoVMQ3D6hJk/656SezYmQJ2X4KcdxzuYQEGUCmPomARly6iqJeUJQVIfls5UJh/xQROIMuPXEGbAwITeUTChgDzzLEOIzuMSLDrX33nsXlx1yFnAs7JyVS7tT3AJhWZns2/nUNbccA+HiVuJGGyfEwTlH6zi5xi5CPxOTpr3dW/E9CDAiLMU/Ysps66o7BJlA/sTVjQrXz+Vn4G87WGk9T57PSQWlTgrGIVZCSzt1GcY1FnSWdX2W90HyDIt81zPUTJ5NmNWV6xOUSJHxyzvSO/IkMFmA9DjJExKBoVPOkwACJObIDMh7gzECqXQChYJIIXYecNvEGrBSLAzsj5tP+jgCVQcPGjzMtBxLnNP1118/z+Lnf8hSkLOA/zCFjttVIXYL6XjKU55SalF1EdylrIIIqwxKVj2uHxYX8TOIA+shEhyEtWvgJmD9nMk9MlvUT7tAniyvI+5HgP+44/8mDVnBlFPX100zERTvKUtZkWSiYK2JdFj0uwqTW+S+b3XIgjyJE0aeJhUsPxHy5KLF+YzzorFTy2PIVmsLzHyQHjFNLBMC/1inxD0J7F6QcqWkKWAWK0SMqwjxYhXoqlJGqGUKWkkc0esqEFLk0z10jxBW8U1ekQb3q6v3CLh1Ddqj1uJC0pEn7vO2LAm0IHi2kETZTCy1fQLCyIV+5plnTm/pHkz+EAsLwstCNkYSVn0TAG71LkP/ZB2UPNQnMAb0ljyxskTW2biAqFDIXClzIXOGy8fsSQyXAcJyJmaKYqC6Cn1CzJaq6Mh1op3gyuH+kOAwChB+Vlau466QJzEl3LBdt1KMCm4u8V6swV2FyYsM5ChQa1JDhDgYP/XHLkNMqslIX8kTD5bwkkklc0yEPHFPmdGMkzyZ6YupYdnwUHUd3H8WhJSNJ/Bc4LdSB+I0ugqxWMjT4osvPrJVIzE+sJRSrKMO2maIyP5MC2yOG2IBZTOZpHTdSjEqZFSyLiIdXYV7xmXHQsGqLxYUeZorQO6Vsenb2otBnlhGkadJGUMmQp4MoAKVx5liiDypai5Yl6us63A9BoNTTz21mG6f85znlIDkLrsXxFe4PzID9Y9EO6HCvVggy0KMAu5KsV9cKF0gT0pnqKMjk3UuTLhGgVUK3GOJLV0Fy5JYT7GGXHe77rprCY9A/hFjSrjL1ifZoOqm9Y08eRaRJ/HLvSNPUmC5l8aZwSLeQgCogGQxRXMFZvNS3lnyxl2XabahbwgWV3eqy+7HuQ6kgkUGqRgFiL1nnxKrM0LbCs+VRYwRCYq2T7AmJg9BV7NeA7wbrgFhFyyuxiB3j5ItJp5dLgAq29XEWQJKnxDkiafFPZxU1uREyJMbLnh73BksBkOWDe67RPsg+I/LTrFP7xPthAwmGaGq/I4S+M6N4tm3EHZXyJMlnaS2922Jkn322acop67WWxuEibryBKxOauwJGhe712XypJCwQqZ9GyuDPKkX2SvyxPfM1MjcNm7yxOIkZbXtQZCsZJQS/7yOEWUDWJnMpHzWduKe1HsSzEoxGQh0LKmcajhZ3NOgT2wze/Y98Vv/If5P7It1wHvKzf4dx8BDHJc4B+cS4tyYTkNsGyx/sDDgtuOyW2KJJZI8tRj6DwKkwr33Cwt90GDvf10gT8YLdboM1F1wM84m9ttvvxJoLdQh0U4oFWIyQpf2CXSXWohcscjTpFyvEyFPgtzUwaGcxwmD4fLLL1/MnNZ+E/ukJo96GQpU8iGbmagOzSQqm0F8B7OvGZhUeqUELKkiuJn1zPXovK5H1psgbmmwtisbIOVXqrPZDjcHF4AaTiqkGpTN3i34azFgwY1mfES9JmvYEVXJZY2oEE7xUFyW9+D+YI4mgnBlMVmGRSq4ekJmzWrqEJ9rYTmYn6gArSp5vW3w/7HfHXbYoYhjEsePc3FeztM5O/+4FvWPXLNr1w4KRmofNZFUF7cwsHMXn6AAHCIls0sclPsja1IFZPfH4M4d6966j8oBqI+lJACyiAgifgjeKFaSxPzhuaVcubRGWaIFATFx0he6QJ6MCdbh00+7UJdqNuE5NU7PpBBqYjxgBBA/3LeJZpAnSVJ08KSSACZCnmTqIBrjDG6mPMUFLbbYYuX4QUyYb0OQEbEcBvcgKyFBWhAC5CJeBSNS+gZZCt//1YaRyu07MR5iB2LhXzfcYrzchwR79qqEghpUPssiUHZAWreZuqwXYr/2T2mZETsuIoO8BFkJi4DrQcAQMQMhZYecIS6IGhHXQBCZWgSe65xeSf2d3/uv/dif/ZKa8Hl1HtrTeSFVCJjzdd7aSU0qcQeuSWCq9tI+FinmttO2iCgCJW1alWcp48iWYyNoyJv9e419C/C1X5V3tZnZs/tgkHEfiFIZ4h8EHDpmiM+1uA/iI4jfu4fEvbGPEOnC9usYcRzbYvvCiHutr5hJIhjO2blrF9cRn12bPqD9iOu1VIpr1we1g7YLwquvaO8gttpL2wUBHiS2yK7+7v4j+kitCYB7oMis55Y4B31jlIxZExYKWd/sghtMTSDt6TrV6+oLWI09z/pckqf2wmTeeNM3yxNvifGJvuwdeaIMpDmPkzyxPrBeUHyUEMVDsVA8rCyhcCigUDCUPxJAmYSC8d7vgnxQOGIiwnrCykTZsDqJ76L8KR0WFFkRrF5mCsyNgqJjyRbWLO89EGYULCusXrJdDGCsYcoQSE8VsMvSYnDnx+eeY0GzdpOS/SxqrC9iUygsAz83ntmz2b9Zf+3OC5deSLj2aqm/J/4T+7A/+7V/x3E8x3UOzsd5OUdWCuct08W1DFr4kCX3A0Gg6N0TStogHoQH0UAu3EPECzFwz/wWGUAS3LsgAe4ZchfE0kOHgLlnLIDuG4ugeyc92/2Tmu6zNdx8Rtp8tr22HtpHWBDt13H0B8qW8kEu/ab+XfyWNSMk/ocUBhGNvhYkFEnSZ8Oy6FpdN9Lk2il5/Ti2aUdtg4whZf6rLbWt2kW2y3wzkdC+8Rl5NyghjmIKkP1VVlmluLslWyglgejb36hp7Pol8oRcd4U8aUfjgf7bF7DQ6nueOc9lop2gL5AnuqRPCPJkXEKeJuVRmAh5YhVAKAbXZ1uUEDuETAgWp7C5e4jFaAlFzm1HKHbCHYSsEAqfeO/3TPrIQBAXJEG2G9KAQCAUCAYTI9cRVwcCJx5IXBC2PJO4oLkM1kFKVbsrksk9Z/DW3oJ3gyiOShLdh/jsfgyL9yL6Y8SQ1Z/rbST+S2J/JEglshmEtP5+UGpiGr/3SoKUBjF1fSQIak1Svcb7EG1CtBPRTiH6qW1cnNow3gfJ5f6MV22unyMS+rz74Dnw+1FLjXhOkDVEsQvkyXUjpcieduoLjEvIk8mKSVyinVALD3katVht12EMRp54cFz7pPTo2MmTizajYZnRCInEICJInksI0RQQmPFK7YHBaiYDFiKGPLGuIYhth/Pl3kT2+kSewD1i4c3M5PaCt4K1uOuLN4+KIE8s4L0iT1wfYki4JcyqE4lEP8BlK4aLe7IL5InlTTwhyxNLW59AOSG6XS6SOdchaanv5GmS1z528iQTTcyTeKEumO4TicTsgMs7nv2ukCfxZQLcuRz7BJNcsXBc53UpElZgIuyglvg+xH+EKERpFSELUV5FrCtXOHc0HVC7set4ynBZk8F4ynBNc0Mj5ayErIMh4a6uBQEO4aIeFPc4RNgGF7UFvYdJ/dv4v/3GsZwD9/iwsIIIIXCNrtU1h4tfu2gnbfZAIR7IUxSc7hOQJ7Gi3HaTvPaxkyexToIwBeF6WBKJRD8grlBmpYB6yqLtEPclQF/8D4XZJ3CHyBaVHGOsjmQKyTAyXy3NU4vvJFj4PhIt/IfUCReRdCFRQ0KFxJthyReUY2T6ItvDsny5UyVX0CfOM7KmSWRMD2ZNExmmkT1NJJeESM4g+qmEIsknJJKIvNZSf0ckphD7iH06RmS1OjdeF+ft/F2Ha3J9rte1aw9tpN20pfaVfCReWAISAwQ9qm9K8LAofJ+AYOofAsbVBpwUxk6eZNnJCvLwdGEATSQSswMuIMkiFEQXnn2WMtmLlCErVJ/AWoLoRBZyiM+R3VlneIbI9IxszwWJTFDkhGtQ1qxtUW6D2G8cg8RxnYM4tCi1QfxWFqn/DdadGxSEY5gEGbKvuFb7DlK0IInfxj5qGXYsMnheyJbf209cX7R3tHG0m8xbmbPaDSmTfd0nIE+I5qqrrjpR4jh28iRlX6cww+iC6T6RSMwOlDYw4LMwdMnyREGOUgy0D+BGksTBhRduOi467rlwzXGvhDvOWM9VFe43biwuLZmbMjnD3cbVNSiRJRoSmaS1yAR1v8KlV0tko4ZEtmotzovYB7fbMNdaZMe6pjpT1+9sd92xIkO43cLlKQkm3J0Lm/xSt7H/25/9O7bzct6O1TfoWyySyNMkF5AfO3lSYA+7ZqbU6RKJRD+gfhnyxC3RBZe9GBYWAQSKck4kEpNHkCf16HpFnhSJNBjx9WL3iUSiH1C7S4FOLvsukCeWEPEqJnuChxOJxOTBwoc8KeAraH5SGDt5sgacGAKBc8ykiUSiH7A8EvIklobro+3gIhJTIu6EeymRSEweyBPP1Ytf/OIypkwKYydPliaJLAc+20Qi0Q9IKxbYK4uoC+RJLI3sLAG7qt4nEonJI8iTTEOFQieFsZMn6/AYkGQWCOZLJBL9gAVMkSdp7F0gTyZ30stlOFm2KZFITB6C5pWtQJ4sUTMpjJ08WchPnQupmGZ2iUSiH7AwNvKkTpCMobZDTKbwAosgW18xkUhMHkGeXvrSlzZnn3329NbxY+zkSfE1BcKYwsUUJBKJfkBxP/VprGvZBfIkG9hET5yWRcQTicTkgTwpLIo8ffGLX5zeOn6MnTyJe1AhVoE02SyJRKIfQJqQJ9WS1cdpO9T3QZ5Um84FchOJdkBtKxxizTXXLBm8k8LYyZNy6kxuZnOKkSUSiX5ArBO3nSUmukCelFOwvMbLXvayUqMqkUhMHjV5+vKXvzy9dfwYO3lSTl2NBguEKkKXSCT6AfWdkCfrcllioe3gWmR52mCDDZpvfOMb01sTicQkoXq7OpFrrbVWc84550xvHT/GTp5UBLW2lRW7rUidSCT6AUsyIU+WaOoCeXKOFm9N8pRItAdBnliEv/KVr0xvHT/GTp4UtTKIrr/++r1bqTyR6DO465EnWXeWWGg71JNBnl75ylc2X/va16a3JhKJSSLI08tf/vLm3HPPnd46foydPClqddBBBzWveMUryiKMiUSiH5BlK9ZRodwukCexFQZpY1XGPCUS7YAFqE1qkKdJPpdjJ0+KWn3gAx8owV4//OEPp7cmEom5DkudIE8K5bLqtB0GaeRp7bXXnqh7IJFI/C+CPK2zzjoTtQiPnTypyyBwVI2Gq6++enprIpGY67CqwGabbdZ89rOf7QR5uvfeewt5MlZNMqsnkUj8L5AnVux11123+frXvz69dfwYO3lSlwF5EilvpfI///nP098kEom5jO22267ZdNNNS7kShe7ajvvuu68M0qzkaXlKJNoBkxrP5XrrrTfRRI6xkyczOG47JjeLbf7xj3+c/iaRSMxlbLHFFqXgpIzbLpAnYxPL06qrrjrRZSASicT/Annae++9C3k677zzpreOH2MnT+oyHHzwwc2rXvWq5sILL2z+8Ic/TH+z6MC6ZSA0k9TwovW5DRTqu/POO8syDBYBtdaequfqT7GKXXXVVUW8v/TSSwvZs0Coda6++93vNt/+9rebb37zm+UGYsD8r6L/XSOSyEVp0BUkL8uQ0lDnSpV1rgtL1Vjrz4KpgmhPPvnkkol0wgknlHTu448/vtTEsQr9Mccc03z0ox8t1ZlVaiZHHXVU85GPfKT58Ic/XAoQfuhDHyq/Pfzww5vDDjusOfTQQ4uVD1nV5gL13//+9zfvfe97S7mIWmRAPpD43Xve856/+L39EfuuxfGIYxO1vWx3PiHOL8T5hjj/WlyT//udaySutxZtUIt28XvvtZP20obaTzsq1Gi5EHLccccV0d7anrgPJ510UrknxD1yv9w7lhP30j0Vw+c+n3HGGeWeC2DUD/QJfUMf0Vf0GX1HH7rkkktKf9KvyBVXXPEXfe0HP/hBSaaQjaqcx3XXXVf6pKKy+ueNN95YljbSXy2ufcstt5T+e9ttt5X12PTnO+64o1TIVuiRWIhXfydS8PV/BEb/9+q58Hx4Hv/0pz9NPzmzC/FO6rtpN8HYbYd2UIxvxRVXLOecSCQmj9///vel/pqM/fPPP3966/gxdvKEXBxwwAHNhhtuWBoglDvFShlT7BSltGYDl5kfEx2maZHOPffcswSeqvy76667Nm9/+9ubXXbZpcRT7Ljjjs0OO+zQbL/99mXhYW4Css0225S19Lbaaqsy+7VKuuVhLPhpuQjp0wZ28RjcCmbH6lBtvPHGzete97ry+trXvnaebLTRRuX7ENdSy6tf/eq/SQb3t/rqq5eOUh/TOcS5xHk5zzhnQlER10NcG0FcbXe9rptoA6I9BkU7hdiP39Xbhv0n9hX7lbHkmN7HMYdJnJPXWqyg7VrjGmqJ66vFsfjEtUu0x/zu5bD7Wd8P7UXU+yHxuRaZH2ZCg9vjP7XYZ9zDuGdxn7x3vdF+2nfzzTcv/VW/1X+33HLL8qo/69vbbrtt6ef6vL7vGfAskJ122qk8G54Rzwrx3Hh+QnbffffyTBHPF9lrr73K8+nZ8+pZ9NwGeUaYPbcIKoIb5B1RRU6RfhMAJNTEwCRhlVVWKddsQmEC0wVYQ2uFFVYoZDmRSEweQZ6Mpd/61remt44fYydPZsgGW0qBwqD8KEvKwrZQErVioBAoAkrA4G+wN8DH4F4P7AgYi4dBncUirBBhfRhmeTDQ19YHgz6prRCDlggKgbBIEBlEFARhoWBZIiwVYa0gLE8GYsJaYUZLWKdYMQjlwppBxIj5nVcWLbEXBAll5QhLh8A51i/C6oGRs3zoXIQFxPpcrCC+995ipywiw4RVcFBYTsKKEtsG/0fsN8TviXMMK4xj12L7oMR5h2gL1+aaiOurxTXV4rfay3FDtJV2I9rQ92El1L4krIVxL8Jq6B6E5dB9jPsaFsToA2FFDEuiPhP9p7Yo6mv6nn6oP7KK6Z9HHnnkPGtibUnUn8OCiLggQxR7TDS8l4ESZAf5qScZnh1ECrFCtDxX8Yx53t489ex5DhE4JBOxRPAQvTXWWKMQUeRQzSPvkWGiUJ1sNILg+q0YoWHy7Gc/u5wHy9o4LM6zAePKcsstV+55IpGYPII8GZvoiklh7OQJlCgIBUgxhiKnjC+66KLi1rjsssuaK6+8smTk+T0XBvcF1wWXBXcFIsZNwUXBNWE5BS4JbggzW66I+++/vwzU3HbM8Bmgnpg09EF9UZ/UP4kBQX+VSaLvcmtxrRF9mrtN/+Z+09e58bjpuOg8A54F7jvPheeDay9c0J4b7j+uQM8S8sJF6PniMvSscSN67sI17Vn0THo2kc4gpUi6z0E+Ec4gmzEpCIJZu6aDWDqG6+wKTMiWWWaZci3jgL5hvNIfIrzAfSbcsMY699zn+p5z4cY9N0bGfTdu1vfdeBr3nYs47r37buyNe28SZKJjfI5JmNeYmOgHMSGJvlCHKtQTj8FJx2CfqCcZMcEIK2ZMZuuJrN8OTmRj8hKyoEmsc4lJrEkRMYGx75jEuoaQmMS6PhKTrpjExkQ2JmnDJrIkJn7a0rnZd633wpU/P73n/rrP7nk8++GaD92nvxg7jCP6UOi/ReWKnwSEF/BC8RAYnyaFiZCnRCKR6AJYsp/5zGcWF0GEDLB8s+ixgodFj2U8wgVY9Vj0IkygDhFg3WNtZ3XnouXGpATsn2uedc+rz7azAHrlAvbefxfk1uXSdVxhCmG5d04sjuHODQt+uG/DXVu7alkxhUyQsHAikhEvGfGOEd/IKhpxjHXcIstpxCiGF4B1dX5xiOERsE//De9AWGnDUus/IfYRllti38RxwpIb1lxSx0qGddf182DEeUf8peshEafpOsMCTCLGk2gP7RKinYg203Yh2tJ90Pba1nEHrcbOp+5nLM3D+tlgOIq+oE9EeIT+4re+rz09+o7fRhiAvsIiPejlif7hnITOOE/ny9rtOlyba3X92sf1uOa47/U9H3a/3dfw/CDKtYQHaFD8h/Vb0lnvLE+JRCLRBbBKcFVyO3JncmsGgSERckAZDZKZkJrczE/q34YCjH3aPxdqkKeI6atj+CJuL+L0kC+ECxmr3a0UDnerGL1wuSob4/q4XNW0EmPJBfviF7+4WW211ZoXvehFRby33W/81v/8377sN1y7jhvnF7F9ztG52ua7uh2jDUm0X7yP9qHkQ9GHBEkMQQCCMEa8ax0LGFITSYSEBHFAGur4wJA6ThCpCUEsRhX/q/fpldgW8btBlGqy5ByDmAdp0g7Rf7Sb/qI9tav21dbee3Uvgqi7R0i6flH3iegPdV/QD9x/Wacrr7xys9JKKzUvfOELmxe84AUlHpBbe9llly2y1FJLNUsvvXSz/PLLl+8lW/it//ivuEf7if5E7J/oW44X4vjDxG+f+9znFhLL2jYpJHlKJBKJ+YAbhMuE25NLLIQLJYSbtBaulWHCvTZMuGFqie31f+tj1ceuz8l5hnDbhesuxDXUwiUUwrUX7r0Qbr5aapffMHdv7fYLN1Ud9zgY38h9Fe5AEu6tcBGH64sbLNyE4SokdfxiSLgP6zjGOpYxpI5pDPdixDZGfGO4G8PlV0u4AwkX3KBE/OsDybD/knr/cUwux9rtOOh6HBY7S2q346DLMdpSuw66GsPF6F5FaA13rnvsXkfmsH6gP0Tf8Eoig7gWfWhBMtjnhonf2T935STDcJI8JRKJRGKhQFmFiKMJEb8XIsZGXAqJmL5Bie+HiVidYSJWbpiI7xkmYsbmJ+KChol42UGJ2MP5iXUaZ1uGHSdk2DmSwWsZvOa6bQbbMNq4vg/1/XJPI3aYxH2PvtBHJHlKJBKJRCKRGAFJnhKJRCKRSCRGQJKnRCKRSCQSiRGQ5CmRSCQSiURiBCR5SiQSiUQikRgBSZ4SiUQikUgkRkCSp0QikUgkEokRkOQpkUgkEolEYgQkeUokEolEIpEYAUmeEolEIpFIJEZAkqdEIpFIJBKJEZDkKZFIJBKJRGIEJHlKJBKJRCKRGAFJnhKJRCKRSCQWGk3z/wELbgX18fAHaAAAAABJRU5ErkJggg==)

  + **ptr**：指向**堆**中字节序列的指针。对应方法：**`as_ptr()`**。**栈**中的地址：**`&字符串名`**
  + **length**：堆中字节序列的字节长度。对应方法：**`len()`**
  + **capacity**：在堆上分配的容量。对应方法：**`capacity()`**

  ```rust
  // 创建一个容量为10的空字符串
  let mut s = String::with_capacity(10);
  println!("s: {}, s.len: {}, s.capacity: {}", s, s.len(), s.capacity());
  
  s.push('a');
  s.push('b');
  s.push('c');
  
  println!("s 栈上的指针： {:p}", &s);
  println!("s 堆上的指针： {:p}", s.as_ptr());
  println!("s: {}, s.len: {}, s.capacity: {}", s, s.len(), s.capacity());
  ```

##### `String` VS `&str` VS `字符串字面量` VS `str类型`

```rust
let string = String::from("rust");
let str = &string[1..];
let literal = "good";
```

![string & str](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAmsAAAIaCAYAAAB/MgqAAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAP+lSURBVHhe7N0FtCTVuT78u+7/3u/6vTESEiwJECC4u7sTkuDBPbhrgODu7hLcPbjLCDAzDOPAwAAzMAzusr/+7Tn7UNM5I2c453RV937Weld3V1WX7Kra+9mv/lPIyMjIyMjIyMgoLTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyMjIyMgoMTJZy8jIyOgBfPvtt+Hzzz8P77//fnjrrbfCK6+8EgYNGhReeOGF8Mwzz4RHHnkk/P3vfw+33357uP7668NVV10VLr744nDOOeeEU045Jfz1r38NxxxzTPw87LDDwr777hu23HLL+LnXXnuFPffcM+y2227hz3/+c9hpp53CDjvsELbbbruw9dZbh6222ipsscUWYZNNNgm/+93vwkYbbRQ22GCD8Ic//CGsv/76Yb311gvrrrtuWHvttcMaa6wRVltttbDCCiuEVVZZJay00krx+3LLLReWXXbZsMwyy4Sll146LLnkklGWWGKJsPjii4fFFlssyqKLLhoWWWSRdll44YXDQgstFGXBBRcMCyywQJh//vnjJ5lvvvnal30fmWeeedq/p30ncVySzoM4ryTF83X+6Rr8J11XUVxvEtef2kB7LLXUUrF9tJP20m7LL798FL+158orrxzbdtVVV41trc3XXHPNKO7BOuusE++H++L+/P73v4/36o9//GO8b+7fxhtvHDbddNOw2WabhT/96U9RrLfdNttsE+/99ttvH5+DHXfcMey8885hl112CbvuumvYfffd4/Piudlnn33CfvvtF/bff/9w4IEHhoMOOigccsgh4S9/+Ut8zo444ohw5JFHhqOOOio+f8cdd1w44YQTwoknnhhOPvnkcOqpp4YzzjgjnHnmmeHss8+Oy+z/oosuCpdcckm47LLLwhVXXBH+9re/hWuuuSZcd9114cYbbww333xzuPXWW+Pzftddd4V77rknPv8PPPBAePjhh8Ojjz4aHn/88fDUU0+FZ599NvTu3Tv07ds3vi8DBgwIAwcODIMHDw5Dhw4Nw4YNCyNGjIjv1MiRI8Prr78eRo0aFd58880wevTo8Pbbb4exY8eGd999N7z33ntRrPcf72QVkMlaRkZGRjcCMTMozjjjjGGWWWYJs846a/jNb34TZptttjDHHHOEOeecM8w111yRbCAuCAKigBQY+A36BnySBvZEqpACg3MawJExgzdihsilQduAjcAZtA3qe+yxRzwnRM8gffDBB4dDDz20fXA+/PDD43KD8kknndQ+IBuMzzrrrLg8DcaXXnppuPzyy8OVV14ZCWZxUEY6b7jhhnDTTTeFW265JQ7OBmnb3XHHHfG7ffjekdx5551TJPaNFKT/IQC33XZb3L/zsN45EETBORHrrr322ni+V199dRTn7/wuvPDCSFJcm327TueKQLt26y+44IJw/vnnh3PPPTecfvrpkVhrH23lt3ZDXrThscceG4mQz6OPPjruG/HW1ogRgmQ9wqTtEShEyn1yvxAgZBzhQrwSIUfItt1223i/kTpEb/PNN4/32bPgmfBsbLjhhu1kznZIO0EK/cczhSx6rlZfffVIJBFKxHLFFVeMhB3hrCftntEiaff8er49w0WSToqk3LM+77zzRvHszz333FG8C8R78dvf/jaK92T22WeP4r1Jnx1J2q5+++Jv5zfDDDPEY7hfVUAmaxkZGRndiE8//TQSAwPZyy+/HMlbUcz8k4wZM6ZdaAPIO++80y60A0WxjLZg3Lhx7ZI0B0lo8opi2QcffBDlww8/bJePPvpoArHs448/jvLJJ59MIJa5rs8++6xDoa2oly+++CKK77ap//59paP9FI+dvk9K0vkT1+c6i8vS8o5Eu6TPekntWGzT+vZObV6UdJ/qpf6epvvq/he1R2lZvdimI6l/vpIUn8F6Sc9pEs9u0mYVn+eJSfH5n5TUvzdJaM/eeOONqCmjUXvttdeido28+uqrUdvmvRs+fHjUwNHE0dghsoiq/1cBmaxlZGRkdCO+/PLLaNqhjUAIMjIyug/cDSYnSB0NJcLmdxWQyVpGRkZGNwJZe/DBByNZo+HJyMhoLGji+O5xF6gKMlnLyMjI6EYUyZrvGeXDc889F33Q+vfv37YkRPMlHz0+ar169Yr+bMX1GdUFkynNmsCbqiCTtYyMjIxuxFdffRUeeuihSNZ8zygfROOedtppMdowwXfO/yIU+YkJMhB4wBcro9pA1mjWBOBUBZmsZWRkZHQjElkTPff111+3Lc0oE+rJGg0oYibqVXoIfk1SRdCySR2RUW0ksiZSuirIZC0jIyOjG1Eka998803b0owyoZ6siSCUDuV///d/w09/+tPwi1/8In7+4Ac/CNNMM038vffee8cow4zqIZlBpT6pCjJZy8jIyOhGFM2gVYk8axXwVZN77Cc/+Un4v//7v0jEJKGVSFjuMElgJS6WAgKZkzeNZo2DurQY2axdTSSyJu9gVZDJWkZGRkY3okjWMsoF0bnyiEmsKxnwvffeG+67776YBHammWaKAQZIGbNoSowrt1dGtYGspUofVUEmaxkZGRndiEzWyo9kBvWpwoHKBCoGyJw/7bTThp///OfRBEr8ZgZVfikHG1QT8qwha6QqyGQtIyMjoxtBK5OS4maUE0WfNVUIaNv8VkZqYmZQEaLZB7GaQNZSrdSqIJO1jIyMjG5EImtqKmaUE/UBBspBKUyuTmg2gzYfkDX+aiJCq4JM1jIyMjK6EQb6+++/P5O1EqMjsiZthyLi2QzafFA/FFlTGL8qyGQtIyMjoxuRyVr5kcjaTTfdFA488MCoVUPGshm0OYGsSduxxx57tC0pPzJZy8jIyOhGIGsiDDNZKyfkVNt0001j6o6NNtooPP7442HMmDHh9NNPz2bQJgWyJiFuJmsZGRkZGRHSQyhZlMla+UDjKSLw8ssvjwlu33333UjMss9ac2PkyJFh2223DXvuuWfbkvIjk7WMjIyMbgSyJn9XJmvlw2effRbJmAjQYsLiKfFZkzz3iSeeyCXEKghkTeLjTNYyMjIyMiIyWasekLUzzjij3WeNj1q9vPPOO+Hzzz9v+0dGlcAHcauttgp77bVX25LyI5O1jIyMjG4E7c3NN98cs+JnVAO0bAgbjVsOImg+IGtbbLFFJmtTCzOVRx55JNxyyy3h9ttvjyVA7r777vYSIHIVyQRum8ceeyyqoJ966qnw7LPPht69e8ew6+effz7069cvDBgwIAwcODAMHjw4DB06NAwfPjz6JLhJnAvNjPgecCR1XL4K1OEifD766KPwySefxE6WrwI1d67pl5GRMTWgfbnrrrvC8ssv37YkIyOjkcADNt9881iMvyooDVkbPXp0OP/886MfwFprrRXWXnvtKOuss067/O53v4vr11tvvSh+J7Gc/P73v2+XP/zhD+2iWC/ZYIMN/kE23HDDGAVULxtvvPEEsskmm7TLZpttFv70pz/FG46hb7nlllGtyg6ehAOjiJPtt98+irwuRD0y2ZM5tsqgTCTn82mZCBW2dKzfw0T22WefsO+++4b99tsv7L///uGAAw6IIeYHHXRQOPjgg8MhhxwSDj300PCXv/wlHHbYYeHwww+Pte7++te/hiOPPDKGoftOjj322HD88cdHn4wTTzwxnHTSSeGUU06JYel+W865Vl28c845J5oC3BslWC666KJw8cUXh0suuSQ63HLMTZ+cca+55ppw3XXXheuvvz6WbREKT6uAgJNbb701ym233RYJObnjjjuiIOfEwIakk3vuuSeSdQ7aCDuH4ETaH3744UjcH3300UjeiUiuJ598MpL4p59+OobkI/P+Z799+vSJpF4BZ8ReGD5y379//0jwX3zxxUjyX3rppWj+QPaHDBkSCf+wYcMi6R8xYkTczn8Rf3XmpoT8m6UbuE0A3n///dhhjBs3ru0NyGhWZLKWkVEupAjgTNY6iQ8//DAO6hLUIQcGcgN6GuCJAf/aa68NV111VSQB5IYbboiCGBAkwTbpE3EgV199dRRkwv/JlVdeGeWKK66Igmwk4iHihyAkBDlBUi688MIYzo3QOM+zzjor+jVYhuicfPLJkfgcc8wxkSAhRUcffXQkSn4jSkgUMoVUIViIFtKFfCFscr8gZh4iZA1xU6NO21iPzCF6CB/i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Gp/GTNYyMjIyMqYEmax1MaTrOPTQQ2Mx90zWygcVMtZZZ51w8803N6zQvijhPffcM2pgMzIyMjIyJofSkLWbbrop/O53v4uFtqsMubuk7VhjjTXCM88807Y0oyx46KGHwmqrrRbN7V9//XXb0p5FJmsZGRkZGZ1BJmtdDKa1ffbZJ6y88srh6aefbluaURY8+uijYZVVVokpVhqFTNYyMjIyMjqD0pG122+/vW1JNfHVV1+FvfbaKyy33HLhqaeealua0dP49ttv47344osvwqeffho++uij8P7778fJwIorrhjNobZpBIYOHRqfEfn4MjIyMjIyJofSkDU+RM1A1mhNFHJfeumlwxlnnBHzxtULE1wS5IHw1SPpN9EW5LbbbovrtNEtt9wSIxqJ70msI0hvEn6ARZHLjqjDmkTesaJce+21USSPrRdF0IuiGkCSq6666h/kyiuv7FCuuOKKCYSz//cVqTiIKE9y8cUXxyoF55xzTjjzzDPDaaedFk466aSw4447ht/+9rexTZG5RiCTtYyMjIyMzqB0ZM0gWmUwsyEGsuQzty277LJhmWWWiZ+Exm355ZePssIKK0RzKVlppZXi9mTVVVeNflWrr756FP5va621Vgxa4BxPfBeQQbTb+uuvH0Wko0hUIn3IBhtsEMU2fktbQTbeeON22WSTTdpl0003najIHTcx2XDDDeP/fZfHrCibb775P8gWW2zRLs7L/3yXsDbJVltt1S5++5/tHMd5O+baa68d28D1unZt4Fot007Wr7nmmrEdtSutmvuBKDXKZ23YsGGxfmwmaxkZGRkZU4JM1roYH374YXj88cfDAQccEOaff/52IpEkkSdEA2lCZhAPZC0RokRwEnlBVnzfeeedo2Zou+22ixUSCOKiMLl1f/7zn+On/++6667tog6l/1mfltkOSdpll13i8iSWE/v0H580hcTx0qdz2HbbbSMpdX7OO51zIlX1xMp1J1JVJFSJlBIkFmFFqpBZ5LZIci0j1hPrfCK89SS3I7GNYyo31ShkspaRkZGR0RmUjqwx71UZAgwMxAsuuGCYZpppIplI2rL0mUhF+kQ05ptvvvi9nmzQqhW/F8W2SyyxxATLbLfUUktFbRJtHM1SURuXNHJ+L7LIIhNo45ImDrFKZDJp3oqaNWSwSCS33nrr+InAIXNIXiKPiKEEsBzq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iTXYsWOHu+eeeyJdDZrM0LqDXDy+I9+SJFTodccEA4m1BIBY4wAisSZSAcRa+fLlgx/bIpIHPxuUAoMQxRrtLRhlRHsIER6I/UGDBlk1aOitRxBrjz76qMRaIli7dq0dQD744IPYEiGSFy/WyAERIivAM/LSSy+lizVmVYbE7t27LQzaqVOn2BIREuQ8tm/f3nrhhV7V2rJlS4m1RLFu3TrLo5BYE6kAV6iItXnz5sWWCJFYyDkKXayRvF63bt3YEhESR48etW4NhBdDn2DQqlUribVEgVijnPz999+PLREiefFibe7cubElQiQWqvkqVqxoYo384NDEGk17c+XK5T7++OPYkqyHXKzMTDh36NAh6x1XokQJyy8MGcRagQIFkmqcXzBibf369TavTmJNpAKDBw82sTZnzpzYEiESC+0xGHEWslgjHwrv34EDB9z+/fvdvn37zIuD142ctrS0NLd9+3abArB161a3efNmqyIl/47t4TzChT9pNeRB02WA6Th4WDhxcx+jpxtGU2ps2bJl1kaH6mzySCn84dYbj9ET8d8x/1r/GfGW8bkZjdewntx68+vMdvj73vx2XczYF9z6/YGxLDNj32Ecq5gKULx48eCPW4i1/Pnz2/onC8GINf7REGsNGjSILREiefFibfbs2bElQiQWPEQ0Yg5VrCHOnnzySRMDbdq0sV5ZjGliBCEX8e+++66FbxnZRHNf2pAgPsuVK+eee+4598wzz7inn37aFStWzN6HkU+0b3jooYcsJIanJV++fDZVAMudO7flX+HNw5g0wAQFWlNw3xvLaNbrnxdv8a/nefztjfePN8RDxmX+eX59/LrFG+vt7+fNmzfd2Kb4vzMztjczo7s/t7w370Nok32F8RiizBvzTDGec+utt9q+qF+/fuxbCxOJtQTCgaNOnToSayIl8GJt1qxZsSVCJB7EGr89xA4eqJAgaR1B8pvf/MZEAflrd999t7WKoEr03nvvdQ888IB78MEHzRBhCCT/tzfERLzxOp6H8EIcIWIQKRgCBFHH3E5EHkKRZaVLl3bPPvusLUMAEj6moTBhQIzRiO+99551L/joo49cs2bNXPPmzV27du2sQKJHjx6uX79+VtlKKxIa/Y4fP95NmzbNepWR/jB16lRbhpeK48DMmTPdN99842bMmGHPw2iV0adPHzdx4kR7LqO4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)

###### String

+ `string`： 一个可变长度的字符串，它在内存中的表现与动态数组类似(String是由封装了Vec的结构体实现的)，真正的字符串则放在堆上。在堆上给`String`分配一个可伸缩的缓冲区可以按需调整大小。
+ 除了`String`类型的字符串，rust的标准库还提供了其他类型的字符串。如`OsString`， `OsStr`， `CsString` 和` CsStr` 等。以`String`和`Str`分别对应具有所有权和被借用的变量。

###### &str

+ `str`：一个字符串切片，是对其他变量拥有的一段UTF-8文本的引用，它只”借用“这些文本。`&str`也是一个胖指针(Fat Pointer)，包含实际数据的地址和其长度。`&str`可看作一个`&[u8]`，它能存储格式完好的UTF-8。

  **胖指针（Fat Pointer）**：一个双字宽的值，它除了指向对象的地址外，还会储存长度信息。一般普通指针占内存8个字节，而胖指针占16个字节。如切片类型，它保存着指向堆上的地址和长度。

+ **截取特别字符串的切片(如：汉字)**

  切片的索引必须落在字符之间的边界位，也就是`utf-8`字符边界，如汉字在`utf-8`占用3个字节，不落在边界就会报错。

  ```rust
  let s = "泥猴啊";
  let s1 = &s[..2];
  println!("s1: {}", s1);
  ```

  ```shell
  thread 'main' panicked at 'byte index 2 is not a char boundary; it is inside '泥' (bytes 0..3) of `泥猴啊`', src\main.rs:275:15
  note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
  ```

  每个汉字占用三个字节，截取没有落在边界处，`泥`字截取不完整，所以报错。

  改成`&s[..3]`，则可以正常通过编译

  ```rust
  let s = "泥猴啊";
  let s1 = &s[..3];
  println!("s1: {}", s1);
  ```

  ```shell
  s1: 泥
  ```

###### 字符串字面量

+ `literal`：一个字符串字面量，它通常跟程序的机器码存储在**预分配区的只读内存区**，当程序执行时创建，程序退出时会自动释放。

###### str类型

rust中是没有GC的，程序的内存由编译器分配，代码最终编译为LLVM IR，其携带了内存分配信息。要更合理分配内存，编译器要先知道类型大小。

**`str`**是无固定大小的字符串，它只是个类型而已，在运行之前无法确定其大小。在rust中大部分都是可以在**编译期确定大小的类型(Sized Type)**。也存在**动态大小的类型(DST: Dynamic Sized Type)**，如：`str`，由于它无法在编译器确定大小，因此就**不能声明**。rust提供了引用类型来解决这种情况。字符串切片的引用类型**`&str`**，因为它是胖指针，可以在编译器确定大小。

###### `&str` 和 `String` 之间的转换

+ 从`&str`转`String`

  ```rust
  let str = "rust";
  let string1 = String::from(s);
  let string2 = s.to_string();
  ```

+ 从`String`转`&str`

  ```rust
  let string = String:from("rust");
  let str = &string[..];
  ```

​		这种灵活用法是因为 `deref` 隐式强制转换

###### 总结

str字符串序列存储与程序的**堆内存中**或**静态只读区**。而`&str`和`String`都存储在栈上，指针指向`str*`。`str`在rust中仅仅作为一个类型存在。

##### 字符串常用方法

###### 追加(Push)

在字符串尾部追加单个字符`char`使用**`push()`**，追加字符串字面量使用**`push_str()`**。这两个方法都是在原有的字符串上追加，不会返回新的字符串。

由于**字符串追加操作是要修改原来的字符串**，该字符串必须可变的，即字符串变量需用`mut`关键字修饰。

```rust
let mut s = String::from("hello ");

s.push('r');
println!("s: {}", s);

s.push_str("ust");
println!("s: {}", s);
```

```shell
s: hello r
s: hello rust
```

###### 插入(Insert)

在字符串中插入单个字符`char`使用**`insert(index, string)`**，插入字符串字面量**`insert_str(index, string)`**。index：字符(串)插入位置的索引(从0开始，如越界则会发生错误)，string：插入的字符(串)。

由于**字符串插入操作是要修改原来的字符串**，该字符串必须可变的，即字符串变量需用`mut`关键字修饰。

```rust
let mut s = String::from("afg");
s.insert(1, 'b');
println!("s: {}", s);

s.insert_str(2, "cde");
println!("s: {}", s);
```

```shell
s: abfg
s: abcdefg
```

###### 连接(Catenate)

1. **使用`+`或`+=`连接字符串**

   使用`+`或`+=`连接字符串，要求右边的参数必须为字符串切片引用(Slice)类型。当调用`+`的操作符时，相当于调用了String标准库中的`add`方法。

   `add`方法源码：

   ```rust
   #[inline]
   fn add(mut self, other: &str) -> String {
       self.push_str(other);
       self
   }
   ```

   `add`方法的第二个参数是一个引用类型。因此在使用`+`，必须传递切片引用类型。不能直接传递`String`类型。

   `+`和`+=`都是返回一个新的字符串，变量声明可以**不需要**`mut`关键字修饰。

   ```rust
   let s = String::from("hello ");
   let s1 = String::from("rust");
   
   // &s1会自动将 &String 解引用为 &str
   let mut s2 = s + &s1;
   
   s2 += "!";
   
   println!("s2: {}", s2);
   ```

   ```shell
   s2: hello rust!
   ```

   `+`和`+=`都是返回一个新的字符串，在这之后如又使用原字符串是就会报该值的所有权被转移后又被使用的错误。

   ```rust
   let s = String::from("hello");
   let s1 = s + "!";
   println!("s1: {}", s1);
   println!("s: {}", s);
   ```

   ```shell
   error[E0382]: borrow of moved value: `s`
      --> src\main.rs:327:23
       |
   324 |     let s = String::from("hello");
       |         - move occurs because `s` has type `String`, which does not implement the `Copy` trait
   325 |     let s1 = s + "!";
       |              ------- `s` moved due to usage in operator
   326 |     println!("s1: {}", s1);
   327 |     println!("s: {}", s);
       |                       ^ value borrowed here after move
   ```

   由于`s`使用`+`连接字符串相当于调用了`add`方法，由`add`的源码可知，所有权被转移到`add`方法里面，`add`方法调用后就被释放了，同时`s`也被释放了，再使用`s`就会报错。

2. **使用`format!连接字符串`**

   `format!`适用于`String`和`&str`。

   ```rust
   let s = "hello";
   let s1 = String::from("rust");
   let s = format!("{} {}", s, s1);
   println!("{}", s);
   ```

   ```she
   hello rust
   ```

###### 替换(Replace)

替换的方法有三种：

1. **`replace(str, replace_str)`**

   该方法可适用于`String`和`&str`类型。`replace`方法接受两个参数，**str**：要被替换的字符串，**replace_str**：新的字符串。该方法会替换**所有**匹配到的字符串。**该方法是返回一个新的字符串，而不是操作原来的字符串。**

   ```rust
   let s = "I like rust. Learning rust is my favorite";
   let s1 = s.replace("rust", "RUST");
   println!("s1: {}", s1);
   ```

   ```shell
   s1: I like RUST. Learning RUST is my favorite
   ```

2. **`replacen(str, replace_str, n)`**

   该方法可适用于`String`和`&str`类型。`replace`方法接受三个参数，**str**：要被替换的字符串，**replace_str**：新的字符串，**n**：替换的个数。**该方法是返回一个新的字符串，而不是操作原来的字符串。**

   ```rust
   let s = "I like rust. Learning rust is my favorite";
   let s1 = s.replacen("rust", "RUST", 1);
   println!("s1: {}", s1);
   ```

   ```shell
   s1: I like RUST. Learning rust is my favorite
   ```

3. **`replace_range(start..end, str)`**

   该方法只适用于`String`类型。`replace_range`接受两个参数，**start..end**：要替换字符串字符串的范围(Range)，**str**：新字符串。该方法是直接操作原来的字符串，**不会返回新的字符串**，需要使用`mut`关键字修饰。

   ```rust
   let mut s = String::from("abcdef");
   s.replace_range(1..3, "BC");
   println!("s: {}", s);
   ```

   ```shell
   s: aBCdef
   ```

###### 删除(Delete)

与字符删除相关的方法有4个：

1. **`pop()`**：删除并返回字符串的最后一个字符

   该方法是**直接操作原字符串**。其返回值是一个`option`类型，如果字符串为空，则返回**`None`**。

   ```rust
   let mut s = String::from("rust pop 中文！");
   let p1 = s.pop();
   let p2 = s.pop();
   dbg!(p1);
   dbg!(p2);
   dbg!(s);
   ```

   ```shell
   [src\main.rs:327] p1 = Some(
       '！',
   )
   [src\main.rs:328] p2 = Some(
       '文',
   )
   [src\main.rs:329] s = "rust pop 中"
   ```

2. **`remove(index)`**：删除并返回字符串中指定位置的字符

   该方法是**直接操作原字符串**。其返回值是字符串指定位置的字符。**index**：表示该字符**起始**索引位置。 `remove`方法是按**字节**来处理字符串的，如index是不合法的字符边界，则会报错。

   ```rust
   let mut s = String::from("测试hello rust");
   // 直接删除第二个汉字
   dbg!(s.remove(3));
   
   dbg!(s.remove(4));
   dbg!(s);
   ```

   ```shell
   [src\main.rs:335] s.remove(3) = '试'
   [src\main.rs:337] s.remove(4) = 'e'
   [src\main.rs:338] s = "测hllo rust"
   ```

   由于汉字为3个字节，所以第二个汉字要从索引`3`开始。

   `index`一定要符合字符边界(字母或数字占1个字节，汉字占3个字节，大部分emoji占4个字节)，**如出现不合法字符边界，则会报错**

   ```rust
   let mut s = String::from("测试hello rust");
   dbg!(s.remove(1));
   ```

   ```shell
   thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside '测' (bytes 0..3) of `测试hello rust`', /rustc/7737e0b5c4103216d6fd8cf941b7ab9bdbaace7c\library\alloc\src\string.rs:1259:24
   note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
   ```

   **注意：**使用`rmove`方法后，原字符串的字符索引也会发生相应变化，如之后需使用原字符要注意。

3. **`truncate(index)`**：删除字符串中**从指定位置到结尾的全部字符**

   该方法是**直接操作原字符串**。无返回值。 `truncate`方法是按**字节**来处理字符串的，如index是不合法的字符边界，则会报错。

   ```rust
   let mut s = String::from("测试hello rust");
   s.truncate(3);
   dbg!(s);
   ```

   ```shell
   [src\main.rs:335] s = "测"
   ```

   ```rust
   let mut s = String::from("测试hello rust");
   s.truncate(1);
   dbg!(s);
   ```

   ```shell
   thread 'main' panicked at 'assertion failed: self.is_char_boundary(new_len)', /rustc/7737e0b5c4103216d6fd8cf941b7ab9bdbaace7c\library\alloc\src\string.rs:1202:13
   note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
   ```

   出现不合法字符边界，则会报错。

4. **`clear()`**：清空字符串

   该方法是**直接操作原字符串**。调用后，删除字符串中的所有字符，相当于`truncate`方法参数为0。

   ```rust
   let mut s = String::from("测试hello rust");
   s.clear();
   dbg!(s);
   ```

   ```shell
   [src\main.rs:335] s = ""
   ```

##### 字符串的访问

rust中，字符串的访问要注意两点：

1. 由于字符串是UTF-8编码的字节序列，是可变长度编码，所以不能直接使用索引来访问字符。
2. 字符串操作分为按**字节**处理和按**字符**处理两种方式。使用`bytes()`方法是按字节处理，返回字节迭代的迭代器，使用`chars()`方法是按字符处理，返回字符迭代的迭代器。

###### 字符串的长度

+ **`len()`**：以字节为单位的长度
+ **`chars().count()`**：以字符为单位的长度

```rust
let s = "测试hello rust";
dbg!(s.len());
dbg!(s.chars().count());
```

```shell
[src\main.rs:334] s.len() = 16
[src\main.rs:335] s.chars().count() = 12
```

###### 访问字符串元素

由于rust的字符串是UTF-8编码，所以不允许直接使用索引来访问单个字符元素，只能使用迭代器来访问。

+ **`bytes().nth(index)`**：使用字节迭代器，通过index(索引)访问元素，其返回的是`option`类型。
+ **`chars().nth(index)`**：使用字符迭代器，通过index(索引)访问元素，其返回的是`option`类型。

```rust
let s = "测试hello rust";
dbg!(s.bytes().nth(1));
dbg!(s.chars().nth(1));
```

```shell
[src\main.rs:334] s.bytes().nth(1) = Some(
    181,
)
[src\main.rs:335] s.chars().nth(1) = Some(
    '试',
)
```

###### 遍历字符串

+ 按字节遍历

  ```rust
  let s = "测试hello";
  let mut vec = vec![];
  for i in s.bytes() {
      vec.push(i);
  }
  println!("vec: {:?}", vec);
  ```

  ```shell
  vec: [230, 181, 139, 232, 175, 149, 104, 101, 108, 108, 111]
  ```

+ 按字符遍历

  ```rust
  let s = "测试hello";
  let mut vec = vec![];
  for i in s.chars() {
      vec.push(i);
  }
  println!("vec: {:?}", vec);
  ```

  ```shell
  vec: ['测', '试', 'h', 'e', 'l', 'l', 'o']
  ```

##### 切割`String`

切割`String必须沿着字符边界进行`，如切割不合法字符边界，则会报错。

使用**`&字符串名[start..end]`**切割字符串(也是字符串切片)。

```rust
let str = String::from("测试hello");

let selstr = &str[6..8];
dbg!(selstr);
```

```shell
[src\main.rs:336] selstr = "he"
```

如切割不合法字符边界，则会报错。

```rust
let str = String::from("测试hello");

let selstr = &str[2..8];
dbg!(selstr);
```

```shell
thread 'main' panicked at 'byte index 2 is not a char boundary; it is inside '测' (bytes 0..3) of `测试hello`', src\main.rs:335:19
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

##### 元组((n1, n2, ..))

元组是由多种类型组合到一起形成的，因此它是复合类型，元组的长度和元组中元素的顺序都是固定的。

###### 创建一个元组

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
println!("tup: {:?}", tup)
```

```shell
tup: (500, 6.4, 1)
```

变量`tup`被绑定一个元组值`(500, 6.4, 1)`，该元组的类型是`(i32, f64, u8)`

###### 用模式匹配解构元组

使用模式匹配或者 `.` 操作符来获取元组中的值。

```rust
// 全部元素匹配
let tup = (500, 6.4, 1);
let (x, y, z) = tup;
println!("x: {}, y: {}, z: {}", x, y, z);

// 部分元素匹配
let (a, .., b) = tup;
println!("a: {}, b: {}", a, b);

let (c, ..) = tup;
println!("c: {}", c);

let (.., d) = tup;
println!("d: {}", d);
```

```shell
x: 500, y: 6.4, z: 1
a: 500, b: 1
c: 500
d: 1
```

创建一个元组后将其绑到`tup`上，接着使用`let (x ,y, z) = tup;`进行模式匹配。因为元组是`(n1, n2, n3)`形式的，因此需用一样的`(x, y, z)`形式进行匹配。元组中对应的元素会绑定到变量`x`，`y`，`z`上。

解构：用同样的形式把一个复杂对象中的值匹配出来。

###### 访问元组中某个特定元素

模式匹配是把全部或部分元素获取出来，会显得繁琐。访问元组中某个特定元素可用：**`元组名.索引`**，索引从0开始。

```rust
let tup = (500, 6.4, 1);
dbg!(tup.0);
dbg!(tup.2);
```

```shell
[src\main.rs:345] tup.0 = 500
[src\main.rs:346] tup.2 = 1
```

###### 元组的常用示例

函数返回值的解构

```rust
fn main() {
    let s = String::from("hello");

    let (s1, len) = calculate_length(s);
    println!("s1: {:?}, len: {}", s1, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len();
    (s, length)
}
```

```shell
s1: "hello", len: 5
```

##### 结构体(struct)

结构体是由多种类型组合而成

结构体一般由三个部分组成：

+ 通过关键字**`struct`**定义
+ 一个清晰明确的结构体 `名称` (首字母必须**大写**)
+ 内部可以有几个有含义的名称的 `字段`

###### 具有命名字段的结构体(Named-Field Struct)

与元组不同，结构体可以为内部每个字段起一个有含义的名称。结构体更加灵活更加强大。

+ 创建结构体

  如：使用结构体定义网站用户信息：

  ```rust
  struct User {
      active: bool,
      username: String,
      email: String,
      sign_in_count: u64,
  }
  ```

  该结构体名称是`User`，拥有`4`字段，每个字段都有对应的字段名及类型声明。

+ 创建结构体实例

  注意：

  + 初始化实例时，**每个字段**都需要进行初始化
  + 初始化时的字段顺序**不需要**和结构体定义时的顺序一致

  如：创建一个`User`结构体的实例：

  ```rust
  let user1 = User {
      email: String::from("someone@example.com"),
      username: String::from("someone"),
      active: true,
      sign_in_count: 1,
  };
  ```

+ 打印输出结构体实例

  需在文件开头添加**`#[derive(Debug)]`**：Debug输出格式，Debug是一个trait，derive增加派生Debug trait

  ```rust
  #[allow(dead_code)]
  
  #[derive(Debug)]
  
  struct User {
      ...
  }
  
  fn main() {
      let user1 = User {
          ...
      };
      println!("user1: {:#?}", user1);
  }
  ```

  ```shell
  user1: User {
      active: true,
      username: "someone",
      email: "someone@example.com",
      sign_in_count: 1,
  }
  ```

+ 访问和修改结构体字段

  通过**`实例名.字段`**即可访问或修改结构体实例内部的字段值，修改时结构体实例声明必须使用关键字`mut`修饰。rust不支持将某个结构体某个字段标记为可变。

  ```rust
  let mut user1 = User {
      email: String::from("someone@example.com"),
      username: String::from("someone"),
      active: true,
      sign_in_count: 1,
  };
  println!("user1.email: {}", user1.email);
  user1.email = String::from("anotheremail@example.com");
  ```

+ 工厂函数创建结构体实例

  ```rust
  #[allow(dead_code)]
  
  #[derive(Debug)]
  fn main() {
      let email = String::from("someone@example.com");
      let username = String::from("someone");
      let user1 = build_user(email, username);
      println!("user1: {:?}", user1);
  }
  
  fn build_user(email: String, username: String) -> User {
      User {
          email: email,   // 可简化为 email,
          username,
          active: true,
          sign_in_count: 1,
      }
  }
  ```

  ```shell
  user1: User {
      active: true,
      username: "someone",
      email: "someone@example.com",
      sign_in_count: 1,
  }
  ```

+ 以已有的结构体实例，创建新的结构体实例

  如：例如根据已有的 `user1` 实例来构建 `user2`

  + 常规语法：

    ```rust
    let user2 = User{
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
    ```

  + **结构更新语法**

    ```rust
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
    println!("user2: {:#?}", user2);
    ```

    ```shell
    user2: User {
        active: true,
        username: "someone",
        email: "another@example.com",
        sign_in_count: 1,
    }
    ```

    `..`语法表示凡是没有显示声明的字段，全部从`user1`中自动获取。`..user1`必须在**结构体尾**部使用。

    结构体更新语法跟赋值语句 `=` 非常相像。`user1`的部分字段所有权被转移到`user2`中：`username`字段发生了所有权转移，作为结果，`user1`无法再使用。

    ```rust
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
    println!("user1: {:?}", user1);
    ```

    ```shell
    error[E0382]: borrow of partially moved value: `user1`
       --> src\main.rs:380:29
        |
    375 |       let user2 = User {
        |  _________________-
    376 | |         email: String::from("another@example.com"),
    377 | |         ..user1
    378 | |     };
        | |_____- value partially moved here
    379 |       // println!("user2: {:#?}", user2);
    380 |       println!("user1: {:?}", user1);
        |                               ^^^^^ value borrowed here after partial move
        |
        = note: partial move occurs because `user1.username` has type `String`, which does not implement the `Copy` trait
        = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)
    ```

    有三个字段进行赋值，但只有`username`发生转移，这与`Copy`特征有关，实现了`Copy`特征的类型无需所有权转移，直接在赋值时进行数据拷贝。其中`bool`和`u64`类型实现了`Copy`特征，因此`active`和`sign_in_count`字段赋值给`user2`时，仅仅是拷贝，而不是所有权转移。

    **注意**：`username`所有权被转移给了`user2`，导致了`user1`无法再被使用，但不代表`user1`内部其他字段不能被继续使用。如：

    ```rust
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
    dbg!(user1.active);
    dbg!(user1.sign_in_count);
    ```

    ```shell
    [src\main.rs:379] user1.active = true
    [src\main.rs:380] user1.sign_in_count = 1
    ```

+ **结构体的内存表现**

  ![struct](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/4RB6RXhpZgAATU0AKgAAAAgABAE7AAIAAAAEbGluAIdpAAQAAAABAAAISpydAAEAAAAIAAAQauocAAcAAAgMAAAAPgAAAAAc6gAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAB6hwABwAACAwAAAhcAAAAABzqAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABsAGkAbgAAAP/hClxodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvADw/eHBhY2tldCBiZWdpbj0n77u/JyBpZD0nVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkJz8+DQo8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIj48cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6ZGM9Imh0dHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIi8+PHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9InV1aWQ6ZmFmNWJkZDUtYmEzZC0xMWRhLWFkMzEtZDMzZDc1MTgyZjFiIiB4bWxuczpkYz0iaHR0cDovL3B1cmwub3JnL2RjL2VsZW1lbnRzLzEuMS8iPjxkYzpjcmVhdG9yPjxyZGY6U2VxIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+PHJkZjpsaT5saW48L3JkZjpsaT48L3JkZjpTZXE+DQoJCQk8L2RjOmNyZWF0b3I+PC9yZGY6RGVzY3JpcHRpb24+PC9yZGY6UkRGPjwveDp4bXBtZXRhPg0KICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICA8P3hwYWNrZXQgZW5kPSd3Jz8+/9sAQwAHBQUGBQQHBgUGCAcHCAoRCwoJCQoVDxAMERgVGhkYFRgXGx4nIRsdJR0XGCIuIiUoKSssKxogLzMvKjInKisq/9sAQwEHCAgKCQoUCwsUKhwYHCoqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioq/8AAEQgBtAL3AwEiAAIRAQMRAf/EAB8AAAEFAQEBAQEBAAAAAAAAAAABAgMEBQYHCAkKC//EALUQAAIBAwMCBAMFBQQEAAABfQECAwAEEQUSITFBBhNRYQcicRQygZGhCCNCscEVUtHwJDNicoIJChYXGBkaJSYnKCkqNDU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6g4SFhoeI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NoyaTNcYVDCDH5AJBcyZyH46Yru18P6MkgkTSbFXU5DLbICD9cVkt4B0j7RI0Mt7bwSsXe1guWSIknJO0dPwNAGXc6nY6P8AE27vFuI0tP7LaTUnDZEbI42FsdDgt+VdxG6yxq6HKsAQfUGuc1nwZZXvh670zS4obI3jq08gXJkAYE5PU5AxXSIoRFVeijAoAWiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigCtqMttBplzNfhWto4maUMuQVA54rzf4Q+KLbUZNU063gu4IHuXubGKSBhHHbnGMOeOTk47Zrv/ABBbaheaJcW+kPbJcyLtBuVLJg9QQPaq3g7RLnw54TstKvrlbmW2QqXRcKBkkKPYDj8KANuiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAgvr2HTrCa8u22QwIXdsZwB1qSGVJ4EliO5JFDKfUEZFYfjmQReBNZZun2SQfpVO/wDGWleF7SzsbkTT3CWqyPDbR72jjCgF29BQB1dFeZaz4o8S3vizw9/YllDb2N3K4gluZ/kulMRYEqvI45r0ay+1/Y4/7R8n7Tj5/Izsz7Z5oAnpqSJJny3Vtpwdpzg+lZer+JtK0W4ht9QuhHPOCUjCljj1IHQe9cb4D1+DTPDeoahqs5b7frNz9njjBd5MPtAUDr92gD0iiucn8TyX+gveeF7M6lMspheEyCJomHXdnpisj4b6j4kvvDtvda21l9lBlDP5jNNkOw5P3eMY/CgDuqKwLLxrot/qMVnbzvuuGZYJGQqkxXqEY9a36ACiuU1zxje6LdTIdAkngjICzC8iXf8ARSc10ljdrfWEF0q7RKgfbuB257ZHFAE9FcH440+SLxFol1pmpXenXl9drau8UmUK7WPKHg9K19J1+6ttSm0bxKYkvIYTPHcp8sdxEOC2OxHGR70AdLRWBYeM9J1O8W0s5JDNKjPbiRCgnC9dhPWuUsNZ8aXvxEu7dbG0sYzYo4t7ucuFG8jcNnc0AelUVg6n4w0vRpzb38rNNFGJJ/JjLrCvqx7D61twTxXNvHPbuskUihkdTkMD0NABLLHBE0k8ixooyzOcAfjVTTta07VzKNMu47nyThzGcgH61xfxis9VuvC9rNpFvHcJaXS3FyksuxGjUEkMP4gfSuv8N3NrfeHLG8sLQWcFxCsiwiMJtyOmKANOiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAz9e0lNd0G80ySVoVuojGZFHK571xmreDLtdeuL3+ybTXobpIgyzzeU8TIuOvQqeuK9DooA4/VPDer6zYaRcJLZ6XqemXBliWNTJEqlSm3/AL5auh0XTW0nSYbOW5kupEyXmk6uxJJPtyelX6KAOU1vwddan4gk1Gy1h7Fbi2FrcKkQZigJPysfuk5qjF8MoLAxf2Lqtxp628jtAEQP5auoDgZ9SM59Sa7migDl9A8DWvhnUZJ9IvJ44LhP9Kt5DvE8n/PQk9G9fWqOkeA7uzMttfau0umq1x9nt4l2ECUknce5G44rtqKAOFHwvtpbe3+3avd3FzYqq6fOAE+ybehVRwTxyT1rtbaOSK1jjnl86VVAeTbt3n1x2qWoIb62uLqe2gmV5rcgTIDymRkZ/CgDLuvBvh69upbi80qC4llbc7S5bJ+hOBWra2kFjax21nEkMMYwkaDAUVNRQByvifwrqniK/t3j1pbK2tZUnhVLUPIki99xPT2pB4CtJ4rxtUv7vULy7t2tmuZWwY0bqEUcCurooA4g/DWKV4Ly51i7l1W0CizvQoX7Oo/hVBxg9/WrOueEdS1HXEvrDVxZ+bZ/Y7thFl3XOcp/dPX86z9d8WatceLbTRvD80NpE921pNczReZmTyvMwo9gMfjW34k8SS+FNFtGkt5NV1C4kSCOGBdpmc9SB2GMmgChdfDq1vLy6E2pXR0+9KG6shjExVQoy/XGAMitvw5oJ8Oae1hFeSXFojf6NHKBmBP7me4HbNRaB4nTWru6sbiyuNOv7UK0ltPjOxujAjgjg1W8TeLpNC1TT9NsdLn1S8vCzeVAQCiKOWJPvgUAL448P3XiLQUt7GRPNhnSfyZTiOcKc7G9jW3pz3D6dA15bLaT7BvhRwwQ+gI7VS8PeIIPEFnLJHDLbT28phuLeYYeJxzg/gQc1j6t43msfE8ukabod3qn2a3E1zLAQBFk8Lz1OMnFAHXUVS0fVrbXNIg1GxLGGdcgMMFT0II7EEYrlZviM0WqamkWhXtzpumyCOa/iAKg4+YgdSB7UAdvRUC3sD6eL5JAbcxeaH7FcZz+VcdpHxI+3LbT3+h3tjp95cGG3vZMFGycISOoDetAHcUVxnirxP4n0Fpbm10O0l06KVE86W6wzBiBkKPc12SnKgnuKAFooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooA5bx1ceIIbPTo/Cq5upr1I5GIyqIQclvYVzMGq+I/D1zDb61e3d5Z2GpEXV2IctLC8WVyB2DnHHpXp9QLfWr372STobqNBI8QPzKpOAf0oA821Px14lkTU59F0yf5bm2jsoJoCDLG5IZ89hn8sVS0bUPH9wLKTxHcyWEE/wBosmMUWWVgGKzn8VwP/r167RQB5V4H1u20fVba11C7OqXWonyk1GK5aTzD1AeM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)

  结构体Data包含两个字段一个Vec和一个元组，结构体在栈上的布局是将它的各个成员相邻排列，这里Vec占用三个usize，而元组将占用另外两个usize，注意这里忽略了内存对齐和填充，如nums成员内有元素，它们将被分配在堆上。

  这也侧面印证了：**把结构体中具有所有权的字段转移出去后，将无法再访问该字段，但是可以正常访问其它的字段**。

###### 元组结构体(Tuple-Like Struct)

元组结构体像元组和结构体的混合体：字段没有名字，只有类型。

+ 创建元组结构体及实例

  ```rust
  #[allow(dead_code)]
  #[derive(Debug)]
  struct Color(i32, i32, i32);
  
  #[derive(Debug)]
  struct Data(String ,Vec<u8>);
  
  fn main() {
      let black = Color(0, 0, 0);
      let data = Data(String::from("data"), vec![0, 0, 0]);
  
      println!("black: {:?}", black);
      println!("data: {:?}", data);
  }
  ```

  ```rust
  black: Color(0, 0, 0)
  data: Data("data", [0, 0, 0])
  ```

+ Color、Data结构体在内存中的排列(忽略内存对齐)

  ![Tuple-Like Struct](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/4RDSRXhpZgAATU0AKgAAAAgABAE7AAIAAAAEbGluAIdpAAQAAAABAAAISpydAAEAAAAIAAAQwuocAAcAAAgMAAAAPgAAAAAc6gAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFkAMAAgAAABQAABCYkAQAAgAAABQAABCskpEAAgAAAAM3MAAAkpIAAgAAAAM3MAAA6hwABwAACAwAAAiMAAAAABzqAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAyMjowNzoxMyAxNjo1OTo0MwAyMDIyOjA3OjEzIDE2OjU5OjQzAAAAbABpAG4AAAD/4QsWaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49J++7vycgaWQ9J1c1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCc/Pg0KPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyI+PHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj48cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0idXVpZDpmYWY1YmRkNS1iYTNkLTExZGEtYWQzMS1kMzNkNzUxODJmMWIiIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIvPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIj48eG1wOkNyZWF0ZURhdGU+MjAyMi0wNy0xM1QxNjo1OTo0My43MDQ8L3htcDpDcmVhdGVEYXRlPjwvcmRmOkRlc2NyaXB0aW9uPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6ZGM9Imh0dHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIj48ZGM6Y3JlYXRvcj48cmRmOlNlcSB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPjxyZGY6bGk+bGluPC9yZGY6bGk+PC9yZGY6U2VxPg0KCQkJPC9kYzpjcmVhdG9yPjwvcmRmOkRlc2NyaXB0aW9uPjwvcmRmOlJERj48L3g6eG1wbWV0YT4NCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgPD94cGFja2V0IGVuZD0ndyc/Pv/bAEMABwUFBgUEBwYFBggHBwgKEQsKCQkKFQ8QDBEY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IfdQpkSJ7jbzXqxVSclQT7iloA8/13RL/UdH8Rz6HazzX9/LGixXg8kFUx9088HHfFMvx4s1/QDZz+Fxps1kYp4C99HKs7IeUwAMZHc16HRQBxtpb6x4k16y1LW9MbSLHTQXitppEeSWUjBY7SQFA6c5rb8M643iHSTfeQIUMzogDbtyq2AenetZlDqVYZBGCKrabptppGnx2WnQiC3i+4gJOPzoA53xdY6nHrGj67pFk2ovp7uktojqrOjgAldxAyMdyKqQnUNP/tfxn4itFtpo7YrbWIkDGKJecMwyNzH0zXb1Xv7C21Oxls76PzbeYbXQkjI/CgBml3b3+k2t3LF5TzxLIUznbkZxXL6rHreieNJ9W0rR31e3vrZYmSKVEaKRehJcj5T7ZNdjHGsMSxxjaiAKo9AKdQBxlhBrXhjSpr1tIk1rVtSuDNcxWs0caxZ6Dc5GQBxWdo//AAkGk6Hrl7regySXmq3bGCxtGWVlDLgb2ztA9TmvRKKAOF8IyS2GuW2gS2UDz2Omobm86uhYkiMHuK7qoktLeO5kuI4I1nlAEkiqAz46ZPepaACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiikDA5wQccHB6UALRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAVHcXEVrbSXFw4SKJS7sewHU1JXnnjTxpFL4d161gsrkWkMT20mpFkWJZSMbQC25vwFAHe2d5BqFnFd2cglgmUPG4BG4HvzUrusalnYKo6knAFcBpnilPD/haysNOtf7RGn6fHNdzmYRRxLtyPmIOWPoBUfinVL7xf8OH1XwxqVtb6dNas8wltzLIcfwghgAfwNAHogIIBByD0IoridI1H/hGfC9vc6prl3q80kEXlWbJCrgsMKFVFU89MsTV7QvG0OqeJLnQL62Wx1SCISmATiX5T7gDBHcUAdRRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUEZBFAGF4T1S51axu57tg228ljjwMYRTgVV8JyH7d4gkkkyg1BgNx4XAFcZD4K8S2F7JBp0FxFM1+Z11Iah+5WMtkr5OeTjjp+Nad34a8QC0uYksxdQvqb3EtstyI/tMZAx8x6DPUUAeipIki7o3Vl9VORWZquuwWOj3t5ayRXL2aFnjWQcY7HHSuJ07wx4u0mHUbeE25t9VjYrFBJtXT3xwFz1B9R3qpb6C0ek3emWnhG707VbmyNs1wHBglPdi4OM98nmgD0my1W1vI7cLPGJpohIIt4LAEZ6Vdrgfh54UvvCUkmn3+nwzhUDLq+8NJJ/sNnnj8q76gAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAoooo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oooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKCQqkscAckntQBieKtYl0rSljsRv1C8cW9onq5/i+gHNWdA0ePQtFgsY23sg3SyHrI55Zj9TWJoKt4j8RXHiKcZtLctbacp6EA/PJ9SeB7V1tABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFcn4iDeHNdg8TwD/RXAttTQD/AJZk/LJj1Unn2JrrKiuraG8tJba5QSQzIUdT0IPUUASKwZQykEEZBB60tct4SuZdNubnwvfuWlsBvtJGPM1sfu/Ur90/h611NABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFAHD+MdbbQvGWhy29pNfXVzDPBDbRHBkYhSAT0Ucck9BVG7+I+py6XBbafpBj1x55YbiFUa6W1EeN77V2tJ1GAME5r0B7S3luormSCN54QRHKyAsgPXB6jNZGreENM1SZblBLp98khlS9sWEcoYgAnOCGyAAQwI4oA5y38aa7BoljHqlnbW2qX1xJHDLeobWIRIM+a8ZYspx/BnP0qtafFmI+GddvbuKJ7vSUaSIIGjjvY+iyRhsnbnjOT+tdVbeD9PETprEtzrxchidWZZ1UjoVTART7qoqr468Ow6t4VuRa6bBcX1vF/ouIl3rjHyqccZAxxQBl+B/G2qazdWVpryac8uo2RvbeTTmJVFBAKOCTyMjkHHWu9rN0fR9M0+M3On6TbafNdKHm8q3WN2JGfmwOT9a0qACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiijpQAVzPi67mumtvDunOVutSJErr1hgH32/HoPrTfC2sSHwpdaxrExETXE8wZj92IMQAPyp3hG0mujc+ItRQrd6kQYkbrDAPuL+PU/WgDoLO0hsLGG0tUEcMKBEUdgKmoooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooA5zxdp1w0NvrWkx79S0tjKiDjzo/44j9R+oFbOmajb6tpdvf2T74LhA6H+n1HSrXXrXJ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1Ifh/K+4SeKdaKycy4aAbj2/5ZcV2VFHsafYftZ9zjh4BnLAv4r1w5G2T54fmHYf6qk/4V9Kdm7xVrfB+Y74enYD91wK7Kij2MOwe2n3Mvw9oFt4b0w2VnLNMrStK0k5UuzMcknaAP0rUoorUyCiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooA//Z)

  元组结构在只需要一个整体名称，但又不关心里面字段名称时非常有用。如一个3D点的坐标表示（x, y, z），`strcut Point(i32, i32, i32);`

+ 使用 **`. `**号按下标访问元组结构体中的字段

  ```rust
  println!("black.0: {}", black.0);
  println!("data.0: {:?}", data.0);
  ```

  ```shell
  black.0: 0
  data.0: "data"
  ```

+ 当元组结构体只有一个字段的时候，称为 New Type 模式

  ```rust
  #[allow(dead_code)]
  #[derive(Debug)]
  struct Integer(u32);
  
  let i = Integer(10);
  ```

###### 单元结构体(Unit-Like Struct)

没有任何字段的结构体，类似于单元类型。

当需要定义一个类型，但不关心该类型的内容，只关心它的行为时，就可以使用 单元结构体。

```rust
#[allow(unused)]
struct AlwaysEqual; // <=> struct AlwaysEqual{}

let subject = AlwaysEqual;

// 不关心 AlwaysEqual 的字段数据，只关心它的行为，因此将它声明为单元结构体，然后再为它实现某个特征
impl SomeTrait for AlwaysEqual {}
```

单元结构体和 new type 模式类似，也相当于定义了一个新的类型。

单元结构体一般用于特定场景，标准库源码中RangeFull就是一个单元结构体。

###### 结构体数据的所有权

在之前的 `User` 结构体的定义中，有一处细节：我们使用了自身拥有所有权的 `String` 类型而不是基于引用的 `&str` 字符串切片类型。这是一个有意而为之的选择：因为我们想要这个结构体拥有它所有的数据，而不是从其它地方借用数据。

如要让 `User` 结构体从其它对象借用数据，就需要引入**生命周期(lifetimes)**。作用是生命周期能确保结构体的作用范围要比它所借用的数据的作用范围要小。

总之，如果你想在结构体中使用一个引用，就必须加上生命周期，否则就会报错：

```rust
#[allow(dead_code)]
#[derive(Debug)]

struct User {
    name: &str,
    active: bool,
}

fn main() {
    let user = User {
        name: "someone",
        active: true,
    }
}
```

编译器会报错它需要生命周期标识符：

```shell
error[E0106]: missing lifetime specifier
   --> src\main.rs:430:11
    |
430 |     name: &str,
    |           ^ expected named lifetime parameter
    |
help: consider introducing a named lifetime parameter
    |
429 ~ struct User<'a> {
430 ~     name: &'a str,
```

之后会在生命周期中解决这个问题以便在结构体中存储引用。

###### 使用`#[derive(Debug)]`打印结构体的信息

在之前的代码中使用`#[derive(Debug)]`对结构体进行标记，这样才能使用`println!("{:?}", s);`打印输出。

如不加就会报错：

```rust
struct Ponit {
    x: i32,
    y: i32,
}

fn main() {
    let ponit = Ponit {
        x: 0,
        y: 0,
    };

    println!("point: {:?}", ponit);
}
```

```shell
error[E0277]: `Ponit` doesn't implement `Debug`
   --> src\main.rs:437:29
    |
437 |     println!("point: {:?}", ponit);
    |                             ^^^^^ `Ponit` cannot be formatted using `{:?}`
    |
    = help: the trait `Debug` is not implemented for `Ponit`
    = note: add `#[derive(Debug)]` to `Ponit` or manually `impl Debug for Ponit`
    = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)
```

报错的原因是没有结构体没有实现`Debug`特征。

**在rust中默认不会实现Debug，但可通过两种方式可实现：**

+ 手动实现

+ 使用`derive`派生实现

  ```rust
  #[derive(Debug)]
  struct Ponit {
      x: i32,
      y: i32,
  }
  ```

##### 枚举(enum)

###### 定义枚举

枚举通过列举可能的成员来定义一个枚举类型。如：扑克牌花色

枚举里的字段称为：**变体**

```rust
#[allow(unused)]
#[derive(Debug)]
enum PokerSuit {
    Clubs,
    Spades,
    Diamonds,
    Hearts,
}
```

**枚举类型是一个类型，它会包含所有可能的枚举成员**

###### 枚举值

创建`PokerSuit`枚举类型的一个成员实例：

通过**`::`**操作符来访问枚举类型的具体成员

```rust
let heart = PokerSuit::Hearts;
```

**枚举值是该类型中的具体某个成员的实例。**

###### 枚举的使用

+ 使用实例：将枚举传入函数

  ```rust
  fn main() {
      let heart = PokerSuit::Hearts;
  
      print_suit(heart);
  }
  
  fn print_suit(card: PokerSuit) {
      println!("{:?}", card);
  }
  ```

  ```shell
  Hearts
  ```

+ 将数据附加到枚举的变体中

  + 与结构体配合将数据附加到变体

    ```rust
    #[allow(dead_code)]
    #[derive(Debug)]
    struct PokerCard {
        suit: PokerSuit,
        value: u8,
    }
    
    fn main() {
        let c = PokerCard {
            suit: PokerSuit::Clubs,
            value: 1,
        };
        println!("{:?}", c);
    }
    ```

  + 推荐：直接将数据信息附加到变体

    ```rust
    #[allow(dead_code)]
    #[allow(unused)]
    #[derive(Debug)]
    enum PokerCard {
        Clubs(u8),
        Spades(u8),
        Diamonds(u8),
        Hearts(u8),
    }
    
    fn main() {
        let c = PokerCard::Clubs(1);
        println!("{:?}", c);
    }
    ```

    ```shell
    Clubs(1)
    ```

  + 任何类型的数据都可以附加到变体如：字符串、数值、结构体或另一个枚举。

    ```rust
    #[allow(dead_code)]
    #[allow(unused)]
    #[derive(Debug)]
    enum Message {
        Quit,
        Move {x: i32, y: i32},
        Write(String),
        ChangeColor(i32, i32, i32),
    }
    
    fn main() {
        let m1 = Message::Quit;
        let m2 = Message::Move{x: 1, y: 1};
        let m3 = Message::ChangeColor(255, 255, 255);
        println!("{:?}, {:?}, {:?}", m1, m2, m3);
    }
    ```

    ```shell
    Quit, Move { x: 1, y: 1 }, ChangeColor(255, 255, 255)
    ```

    Message枚举类型包含四个不同的变体：

    + `Quit`：没有任何关联数据
    + `Move`：包含一个匿名结构体
    + `Write`：包含一个`String`字符串
    + `ChangeColor`：包含三个`i32`

    也可用结构体来定义这些信息：

    ```rust
    struct QuitMessage; // 单元结构体
    struct MoveMessage {
        x: i32,
        y: i32,
    };
    struct WriteMessage(String); // 元组结构体
    struct ChangeColorMessage(i32, i32, i32); // 元组结构体
    ```

    由于每个结构体都有自己的类型，无法在需要同一类型的地方使用。

    如：假设实现函数来处理信息的状态使用枚举就只需实现一个方法

    ```rust
    fn get_message(message: Message) {...}
    ```

    而使用几个不同结构体则需要几个针对不同结构体的方法。

    ```rust
    fn get_quit_message(quitMessage: QuitMessage) {...};
    
    fn get_move_message(moveMessage: QuitMessage) {...};
    
    fn get_write_message(writeMessage: QuitMessage) {...};
    
    fn get_changecolor_message(changeColorMessage: QuitMessage) {...};
    ```

    代码规范角度来看，枚举的实现更简洁，代码内聚性更强，不像结构体的分散。

###### 枚举在内存中的布局

在内存中，枚举类型中的每个变体的存储都是以一个**整数标记**开始的，rust编译器会选择能够存储该枚举类型的最大变体的最短整型。

+ 无变体值枚举

  ```rust
  enum HTTPStatus {
      Ok,
      NotFound,
  }
  ```

  ![enum1](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/4RDSRXhpZgAATU0AKgAAAAgABAE7AAIAAAAEbGluAIdpAAQAAAABAAAISpydAAEAAAAIAAAQwuocAAcAAAgMAAAAPgAAAAAc6gAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFkAMAAgAAABQAABCYkAQAAgAAABQAABCskpEAAgAAAAMzOAAAkpIAAgAAAAMzOAAA6hwABwAACAwAAAiMAAAAABzqAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAyMjowNzoxNCAxNTo1ODowMQAyMDIyOjA3OjE0IDE1OjU4OjAxAAAAbABpAG4AAAD/4QsWaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49J++7vycgaWQ9J1c1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCc/Pg0KPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyI+PHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj48cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0idXVpZDpmYWY1YmRkNS1iYTNkLTExZGEtYWQzMS1kMzNkNzUxODJmMWIiIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIvPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIj48eG1wOkNyZWF0ZURhdGU+MjAyMi0wNy0xNFQxNTo1ODowMS4zODA8L3htcDpDcmVhdGVEYXRlPjwvcmRmOkRlc2NyaXB0aW9uPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6ZGM9Imh0dHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIj48ZGM6Y3JlYXRvcj48cmRmOlNlcSB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPjxyZGY6bGk+bGluPC9yZGY6bGk+PC9yZGY6U2VxPg0KCQkJPC9kYzpjcmVhdG9yPjwvcmRmOkRlc2NyaXB0aW9uPjwvcmRmOlJERj48L3g6eG1wbWV0YT4NCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgPD94cGFja2V0IGVuZD0ndyc/Pv/bAEMABwUFBgUEBwYFBggHBwgKEQsKCQkKFQ8QDBEY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FG4Z3AGxTlvU8121FL2FPsHt6nc5nw/4SuNI1yfVtQ1Z9RupoBASYBGAM5zwea6aiitklFWRk227sKKKKYgooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigArl5tK8ZNM5h8SWaRliVVrAEgdhnNdRRQByn9k+Nf+hms//BeP8aQ6R42IOPE9mD6/2eP8a6yigDk/7I8a/wDQz2f/AILx/jR/ZHjX/oaLT/wXr/jXWUUAcn/Y/jX/AKGi0/8ABev+NH9j+Nf+hptP/Bev+NdZRQByR0bxtuGPFVqB3H9nLz+tH9jeNf8AoarX/wAF6/411tFAHJf2N41/6Gq1/wDBcv8AjR/Y3jX/AKGu2/8ABctdbRQByX9i+Nf+hrtv/BclJ/YvjbnPiy29v+JclddRQByX9i+Nf+hst/8AwXJR/YvjX/obLf8A8FyV1tFAHJf2J41/6G2D/wAFyUn9ieNf+htg/wDBclddRQByP9ieNf8Aobof/BdHR/YfjX/oboP/AAXR111FAHI/2H41/wChvh/8F0dH9h+NP+hvh/8ABdHXXUUAcj/YXjT/AKG6L/wXx0f2F4z/AOhui/8ABfHXXUUAcj/YXjP/AKG6L/wXpSDQfGfOfF0f/gAldfRQByP9g+Mv+huj/wDABKP7B8Y/9DbH/wCAKV11FAHIf2B4x/6G2P8A8AUpf7A8Yf8AQ2p/4BJXXUUAch/YHjD/AKG1P/AJaP8AhH/F/wD0Nq/+AS119FAHHp4e8YKihvFykgYJ+xrzS/8ACP8Ai/8A6Gxf/ANa6+igDkP+Ee8Xf9DYv/gItH/CPeLv+hsH/gItdfRQByH/AAj3i3/obB/4CCj/AIR3xZ/0Ng/8BR/jXX0UAce3hzxaykDxbjPcWo/xpf8AhHfFn/Q1/wDksP8AGuvooA5D/hHPFf8A0Nf/AJLD/Gj/AIRzxX/0NZ/8Bh/jXX0UAch/wjnir/oaz/4Dj/Gj/hG/FX/Q1n/wH/8Ar119FAHIf8I34p/6Gtv+/H/16a/hnxSyMo8WMpIwCIOn612NFAHIf8I14o/6Gpv+/H/16P8AhGvFH/Q1N/35/wDr119FAHH/APCM+J/+hqf/AL8//Xo/4RnxP/0NT/8Afr/69dhRQBx//CM+Jv8AoapP+/X/ANej/hGPE3/Q1Sf9+j/jXYUUAcc3hfxOUIXxXIGxwfKPH60Dwv4lCgf8JVKcDqYz/jXY0UAcf/wi/iX/AKGmX/v2f8aP+EX8Sf8AQ0y/98H/ABrsKKAOP/4RfxJ/0NM3/fB/xo/4RbxH/wBDTN/3wf8AGuwooA4//hFvEf8A0NM3/fJ/xpknhPxI+3b4snUBgThTyPTrXZ0UAcf/AMIr4i/6Gmf8m/xpP+EV8Rf9DTP+Tf412NFAHH/8Ip4h/wChpuPyb/Gk/wCEU8Q/9DTcf+Pf412NFAHHf8Ip4g/6Gm4/8e/xo/4RPxB/0NNx/wCPf/FV2NFAHG/8Il4h3EnxVc4xwPm/+Kpf+ET1/wD6Gm5/Nv8A4quxooA47/hEtf8A+hpuvzb/AOKo/wCES17/AKGm6/76b/4quxooA47/AIRLXv8Aoabr/vp//iqP+ER13/oabr/vp/8A4quxooA4z/hENf8AMJPiq724GBufrzn+L6Uv/CIa7/0NN3/32/8A8VXZUUAcd/wiGuf9DTef99v/APFUn/CIa5/0NN5/32//AMVXZUUAcb/wh+uf9DTef9/H/wDiqP8AhD9b/wChpvf+/j//ABVdlRQBxf8Awhuu+YSfFd7twML5j9ec/wAX0/Knf8IdrX/Q033/AH9f/wCKrsqKAON/4Q7Wv+hpvv8Av6//AMVR/wAIdrX/AENN9/39f/4quyooA43/AIQ3Wf8Aoab/AP7+v/8AFUDwbrWBnxVf/wDf1/8A4quyooA43/hDNZ/6GnUP+/z/APxVH/CGax/0NOof9/n/APiq7KigDjf+EM1j/oatQ/7/AD//ABVNbwXrJX5fFeoA5HPnP0zz/FXaUUAcafBmrk5/4SrUfwmf/Gj/AIQvV/8AoatS/wC/7/412VFAHM6V4VvbK/Se91/ULtE5ETXDhSQQeeeR7V01FFABRRRQAUUUUAFFY3ifVJtI02K6gKj/AEmNHLDjaWwaI/F2gzW19cRanbvFYf8AHyyt/q/rQBs0VzelePNG1X7Vtaa1FtEJnN1EY8xnowz1FWtJ8WaVrN21tZzOJhH5oSWMoXT+8M9R70AbVFcxp/xC8Panrkml2l07TJKYfMMZEbSDqofoT7V09ABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAY/inw7F4p0N9LuZ3gikdWZo+pAOcVzviD4cQ3cTtorRW37iOM2rJ+7l2MGG7H4j8a7qigDzy28I3d7Z3Gnt4ftdEhucefcpdec5I5G0c8Z7HFWpfh3sn/tcazdPrkakC9KDBTH+r8vOAtdzRQB5V4E8Bpc6DaXDatKbQ3rXkloI1/16uf4+oHHTFeq1BaWVtYxNHZwrCjOXKoMAsepqegAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigD/2Q==)

  每个变体在不指定整数标记值时，默认是从**0**开始，依次递增。

  `HTTPStatus`枚举中`HTTPStatus::NotFound`变体是整数标记值`1`为最大的变体，所以该枚举中每个变体需1个字节来存储。

+ 给变体值指定整数标记值

  ```rust
  enum HTTPStatus {
      Ok = 200,
      NotFound = 404,
  }
  ```

  ![enum2](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/4RDSRXhpZgAATU0AKgAAAAgABAE7AAIAAAAEbGluAIdpAAQAAAABAAAISpydAAEAAAAIAAAQwuocAAcAAAgMAAAAPgAAAAAc6gAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFkAMAAgAAABQAABCYkAQAAgAAABQAABCskpEAAgAAAAM5NgAAkpIAAgAAAAM5NgAA6hwABwAACAwAAAiMAAAAABzqAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAyMjowNzoxNCAxNjoxMzoyNgAyMDIyOjA3OjE0IDE2OjEzOjI2AAAAbABpAG4AAAD/4QsWaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49J++7vycgaWQ9J1c1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCc/Pg0KPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyI+PHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj48cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0idXVpZDpmYWY1YmRkNS1iYTNkLTExZGEtYWQzMS1kMzNkNzUxODJmMWIiIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIvPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIj48eG1wOkNyZWF0ZURhdGU+MjAyMi0wNy0xNFQxNjoxMzoyNi45NTk8L3htcDpDcmVhdGVEYXRlPjwvcmRmOkRlc2NyaXB0aW9uPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6ZGM9Imh0dHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIj48ZGM6Y3JlYXRvcj48cmRmOlNlcSB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPjxyZGY6bGk+bGluPC9yZGY6bGk+PC9yZGY6U2VxPg0KCQkJPC9kYzpjcmVhdG9yPjwvcmRmOkRlc2NyaXB0aW9uPjwvcmRmOlJERj48L3g6eG1wbWV0YT4NCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgPD94cGFja2V0IGVuZD0ndyc/Pv/bAEMABwUFBgUEBwYFBggHBwgKEQsKCQkKFQ8QDBEY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Gcf/ABNAHY0Vxx8Y67/0J91/4Ef/AGNH/CY67/0KFz/4Ef8A2FAHY0Vx3/CY67/0KFz/AOBH/wBhSf8ACZa7nH/CIXH/AIEH/wCIoA7KiuN/4THXf+hRuP8AwIP/AMRR/wAJlrv/AEKM/wD4EH/43QB2VFcb/wAJlrv/AEKM/wD4EH/43TU8aa667h4QnHs1ww/9p0AdpRXG/wDCZa7/ANClN/4EN/8AG6P+Ey13/oUpf/Ahv/jVAHZUVxb+NNdTH/FIzHJA4uH4/wDIVL/wmWu/9CnJ/wCBD/8AxqgDs6K41fGWubhu8KSgZ5IuHOP/ACFXYRP5sKPtK7lB2sORntQA6iiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAJwpJ6Cq2nahb6rp8V7ZsXhmXcpIxVhvun6V5Xotr4003S7LUHMsNtbXvk/2UkILSQM5BkbIyDyCAOgoA9VzzjvRXjWlte6rqya/q/iW1sDDdl7iOW6kjkt1UkeR5WQmPVjzVu5PjLVLiS40mHUBqiXJlEkzmKyECn5UTtIWGOeevagD1qivF9d1bWvEfjmEPb+ILPTrOwBkt7BGWWO5bO0so5ZeOuCK9T8MvqcnhmwbXk2agYV89cc7vfHegDVooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigClNoul3F8L2402zlul6TvArOP+BEZq706UUUAY8OiyweMLnWI5l8m6tUhkiI53IThgfoa2KKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAP/Z)

  这里为`HTTPStatus`枚举为的两个变体的整数标记分别赋值200、404。所以该枚举中`HTTPStatus::NotFound`变体是整数标记值`404`为最大的变体，所以该枚举中每个变体需2个字节来存储。

+ 附加不同数据类型的枚举

  ```rust
  enum Data {
      Empty,
      Number(i32),
      Array(Vec<i32>),
  }
  
  let num = Data::Number(25);
  let arr = Data::Array(vec![1, 2, 3]);
  ```

  ![enum3](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/4RDSRXhpZgAATU0AKgAAAAgABAE7AAIAAAAEbGluAIdpAAQAAAABAAAISpydAAEAAAAIAAAQwuocAAcAAAgMAAAAPgAAAAAc6gAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFkAMAAgAAABQAABCYkAQAAgAAABQAABCskpEAAgAAAAMwNQAAkpIAAgAAAAMwNQAA6hwABwAACAwAAAiMAAAAABzqAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAyMjowNzoxNCAxNjozMTozNgAyMDIyOjA3OjE0IDE2OjMxOjM2AAAAbABpAG4AAAD/4QsWaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49J++7vycgaWQ9J1c1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCc/Pg0KPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyI+PHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj48cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0idXVpZDpmYWY1YmRkNS1iYTNkLTExZGEtYWQzMS1kMzNkNzUxODJmMWIiIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIvPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIj48eG1wOkNyZWF0ZURhdGU+MjAyMi0wNy0xNFQxNjozMTozNi4wNDc8L3htcDpDcmVhdGVEYXRlPjwvcmRmOkRlc2NyaXB0aW9uPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6ZGM9Imh0dHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIj48ZGM6Y3JlYXRvcj48cmRmOlNlcSB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPjxyZGY6bGk+bGluPC9yZGY6bGk+PC9yZGY6U2VxPg0KCQkJPC9kYzpjcmVhdG9yPjwvcmRmOkRlc2NyaXB0aW9uPjwvcmRmOlJERj48L3g6eG1wbWV0YT4NCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgPD94cGFja2V0IGVuZD0ndyc/Pv/bAEMABwUFBgUEBwYFBggHBwgKEQsKCQkKFQ8QDBEY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tpPHMmAcowOMjPPpU9ABRRVHTdZsNWi32FzHLywKg/MNrFTx16g0AXqKKpLqkLa6+lBW8+O3FwzcbdpYqB9eDQBdoorkX8f29ul7Ne6fPBawtMltcbgy3LRZ3L/sn5TjPXH4UAddRWPoWv8A9rb4LuzksL+JVeS2kYNhW6MrDhgf5itigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigArm/GumXOp6fZx2UHmypeRMSMfIoPJ5rpKKAPK7Kw8d2Wntplp9rEUSybzKYI0+8Svkunz5P+3TbrSpfFmn3XiPUtKmmlguUjtbGQiRo442+cjBwWJz37V6nPGZreSNXKF1Khh1XPevOvD3jWO1+KEvgWLa1lZ2gEdwR80twCS4J6ev4qaAK/iO/wBUnmsrR7pfCWntEzoZGZA5BwAXRl2nHO3PNeg6JdRXei2ssF012vlhftDoVMhHBbBAPJq68aSDEiKw/wBoZpwAAwBgegoAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAGTuYreSQclVJ/SvJtAPj3UNAluY5r7zb6CScyyzRlPmYGMQc5U7NwIIAzXrbAMpB5BGDVXS9Oh0nTILC13eTAu1AxyQM8D8KAOPtbXxNC6Np8N6tkHjBg1WdZp85Idw25towQcbj7Ad80+Dde1PR5IfF0javJcaaz+U7Ax210owoUd8g9e5GfSvTaKAPL9Rv9St/C2nWljaN4ZsZJUinu5UKkAR5O7YQYwW+XfkH6Zq1oNpda3eWlm2vX9/p1jbPvvrS5kiWS4Lggb85kCoQBksD3JNeisqupVwGB6gjNCoqLhFCj0AxQBUu9Mt9R0o6fqIN1C6hZPM/jx3OPpXnum6Xq+neHL7R/D2iyadPDPKXu4tkbSIbjIWInqTGTgnABx+HS/EPxovgXwuNVNv9pka4jhSHPLZPzfkoY/XFdFZXcN/YwXdq4khnjWSN16MpGQaAMLwtb6pDcXJuVvYdOKIIINRnE1wr87iWDNweONx/CsXxTYeJbXxdNqnh77YwuLFIEFssDL5iMxAk83k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FFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABXE+IXVPiNYZJB/suXb/dz5qfe9q7asjWfC+j+IJI5NWs1nkiUqj7ipCk5IyD0OKzqRc4OKNKclCakzIMiFSv3gDkqG5ZvVf9kelL5gb+JJPMPPOBOfb0xUp+HPhY5xpgU9BiV/lHoOaQ/DfwsQw/sxRn7uJX+X6c1w/U5d/6/r+tzt+tx7f1/X9bEZkU/M0gYE7S/wDeP9wj096xPGciHwPrq7xxZS7gDyDtOAvqtb5+G/hYl/8AiWKNwwP3r8frSN8NfCb7hJpCOjJs2GR8Y796awkk73B4qLVrGP4XdR4R0c5BIs4gPUNsHJ9VrV3LyowwJ3bQfvt/eX2HpUifDbwrHHsTTNqgBVAnk+VR2HzdKG+HHhg9NOI54xPJ8o9B81L6pLuH1uPYj3h+rJL5hxnPE59D6YpPNj2hjLnLY3E8lv7p/wBn3qRvht4XIYDTtuSMYmk+X6fNSt8N/C53Y03G4ADE0nH0+aj6pLv/AF/X9bj+tx7f1/X9bHHWTAfGTVcMu46TbD5jmPPmS/8Ajv8AWusDptBTOAdygn5lPHzn1Wnn4ZeEvN81dIVXKhGYTSZZRyAfm6ZNPPw58MHP/EuIJPXz5On9373Sm8I29yY4pJbEe5WIAKybjuCZ4mbn5wewHpQHRmB3rL5h4YnAuG9G9AKd/wAK28LkH/iXfxZXE0nyj0HzdKU/Dfwud/8AxLsbyOk0nA9B81L6pLv/AF/X9b3r63Ht/X9f1tbm/HLofAOs5csPs7DOfmLY6e6CvSbfP2aLdjOwZx9K5v8A4Vv4VLsX0pXDAAI0rkAemN1dQBgADoK6qFJ0k7nLXqqo00FFFFdBzhRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQB//9k=)

  首先看`Array`变体的内存布局，首先是一个整数标记，这里为`2`，然后是三个`usize`存储`Vec`，编译器还将添加一些填充去满足内存对齐，在64位系统上，这个变体总共需32字节;

  `Number`变体的内存布局，它存储一个`i32`，占4个字节，它也需要一个整数标记值`1`，占用一个字节，因为所有变体都具有相同大小，编译器会为其填充直到32字节;
  `Empty'`变体，它只需要一个字节来存储整数标记值`0`，但是编译器也会为它填充直到32字节;

+ **优化**附加不同数据类型的枚举

  枚举的大小由它的最大变体决定，减少内存占用的一个明显技巧就是降低枚举最大变体大小。

  所以可对上一段代码进行优化

  ```rust
  enum Data {
      Empty,
      Number(i32),
      Array(Box<Vec<i32>>),
  }
  
  let num = Data::Number(25);
  let arr = Data::Array(Box::new(vec![1, 2, 3]));
  ```

  ![enum4](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/4RDSRXhpZgAATU0AKgAAAAgABAE7AAIAAAAEbGluAIdpAAQAAAABAAAISpydAAEAAAAIAAAQwuocAAcAAAgMAAAAPgAAAAAc6gAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFkAMAAgAAABQAABCYkAQAAgAAABQAABCskpEAAgAAAAMwNAAAkpIAAgAAAAMwNAAA6hwABwAACAwAAAiMAAAAABzqAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAyMjowNzoxNCAxNjo1MjowNgAyMDIyOjA3OjE0IDE2OjUyOjA2AAAAbABpAG4AAAD/4QsWaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49J++7vycgaWQ9J1c1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCc/Pg0KPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyI+PHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj48cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0idXVpZDpmYWY1YmRkNS1iYTNkLTExZGEtYWQzMS1kMzNkNzUxODJmMWIiIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIvPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIj48eG1wOkNyZWF0ZURhdGU+MjAyMi0wNy0xNFQxNjo1MjowNi4wNDE8L3htcDpDcmVhdGVEYXRlPjwvcmRmOkRlc2NyaXB0aW9uPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6ZGM9Imh0dHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIj48ZGM6Y3JlYXRvcj48cmRmOlNlcSB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPjxyZGY6bGk+bGluPC9yZGY6bGk+PC9yZGY6U2VxPg0KCQkJPC9kYzpjcmVhdG9yPjwvcmRmOkRlc2NyaXB0aW9uPjwvcmRmOlJERj48L3g6eG1wbWV0YT4NCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgPD94cGFja2V0IGVuZD0ndyc/Pv/bAEMABwUFBgUEBwYFBggHBwgKEQsKCQkKFQ8QDBEY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YelA8DalwT4t1DP/AFzSj6jV8g9vE0QPmAGRj5h7f7f/ANajngDH94e/+39fas4eB9Txz4sv+u7/AFadaD4H1M9fFl/ydzDyk5PrR9RqB9YiaRPHtjqemPU/7VGQF/u7RjLfwD/a9SazT4H1ItuPi3UDnO4eUnzH1NL/AMIPqXB/4SzUMj1jTn60/qNXuv6/r+tw+sRNEDb/AHk2j6mIH+eaFXbxjbjnCn7g/wBn3NZo8D6moG3xbqGVOV/dpwaP+EH1LZtXxZqC45UiNOD60vqFT+v6/r7rH1iP9f1/X56I68gc9QOn0Ho1KSDnPH8JP/sn196zv+EH1LBA8WagMj/nmnX1+tH/AAg+pZz/AMJZf9Mf6pP85o+oVe6D6xE0G4z2wNpP93/Y9x70pBGRyMcEL1X/AGV9RWb/AMIPqfbxbqAIGFPlJwKQeBtTXp4t1D5T8n7tPlHej6jU/r+v6/I9vE0znBGNxPUKfvey/wBaByDwH3nBxwJD6D0xWa3gfUirKPFmoLn7uI0+X6Uv/CD6kQQfFl/yMH90nSn9Rq9/6/r+t7n1iJm+MPni0Ykht2rQAseN3Xg+wr0MdBXGp4AkkvrSfUtfvL2O1mWVYXRQrFQcZx9a7LpXfhaMqUWpHPVmpvQKKKK6zEKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAopGdUxvYLk4GTjJpaACiiigAooooAKKKKACiikd1jRndgqqMlicACgBaKRHWRFeNgysMqynII9aWgAoqvf6haaXZvdahOsECYy7ep4A9z7VOjiSNXTlWAI4xxQAtFFVtQ1G00uza71CdYIFIDSP0GTgZ/E0AWaKAQygqcg8gjvRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFZ95r+j6fc/Z7/VbK1mxu8ua4RGx64JrQrzq5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AJKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigArl/HKfuNEkwpEetWhJPbMoXj866iuF+Md3eWHw4nu9LVWvYby0aDcMgOJ0xSbsrjSu7FK21u587UbvSYxPq2v3zW9ipOVit4f3fmt/sBvMb33D1rtdB0O28P6UtnalpCWMk0z/emkblnb3JrzTwnovi3w5bLJHe6K1zJboredaSOY4lAwikSjnOSR1JJJroRqHjraP9P0DAGdxsJuevyn97w3tWH1ml3N/q9Tsd5RXB/wBo+OFB36hoShTl2OnTfuz2Rv3vU0HUfHA3CW/0KMoP3oGnzExk9B/ruc0fWaXcPq1Xsd5RXBnUfHO5g19oIZR8ypYSseem3998x5rK0nxZ441HW9XsDdaABpcqRvItlKQ4MauWH74cjdjFH1in3F9XqdjpfFWsalY6xa29vdjTrRojIbprUzLJIG/1bYB2jGTnj61nL461X+2R/wASqWTTkuGjudlu+63iBwsmTw+44OFBwvvQ1/43IUNf+H2GMkNYS4Zew/133/8AZoa/8cKo/wBP0FQMk50+YAHnCN+94Pt/kn1ml3H9Wq9hug+NNY1XxhcWrWmbA3sttGot3BWNAf3pk+6cspXb15rv6820xfGWjW1xEmo6KsbXEk8gfT5iYHkYts4l6En369au/wBo+OVLeZfaCNvEirYSkpnpt/ffN/Sj6zS7/wBf1/WjD6vV7f1/X9ao7yivLvEnivxx4f0Zr9rvQJtssMJRbKXDGSRUyp87kjfkj2rWXUfHG1c3+gZ27yDYy8p6/wCu+9/s0fWKfcX1ep2Ev/Ef/CPa1rElvZyXl3dXccUUKKx6RKSzbQSFHsDT3+IlxB9miuNAuhcNClxcKCAsEbOV3Hdg++MZrH1PTfFeq7He/wBDtriN/OS8t7GVJFbBUAnzSNxHG0g0/T7bxhp1pNHNqej3TSHNxLeWE0jAnpG37zAz6AY5o+s0u4/q1Xseky3MMFo1zPIsUKJvZ3OAo9Sa8p1PQ38S217qmjaX5NhK+IEKYkvJGbAlc9RGuche+Ksa5p/i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/NU/VJ9x/W4dij4mGPCurYDR+XYzDn70AKHhv72f0qv4LBHgbR+Fw1nEhGeJDtGIz6Y9a05PhhpU8TRXGp65JGyFGVtUm+bIwc/N+lFv8L9JtbZbe31PXIo0QRoqarMAqD+HG7pVfVZ2t/X9f1toL61C9/6/r+t9SwH+YliWOdpJ6k/3W55QetJkM3OWwc4HXPqPVBTT8OLD+HWNeHG3P9rT5C/3R81J/wAK20/5cavruFOB/wATWfhf7o+ap+qT7j+tw7EhTcAFHmbzvCjgSH++pxwB6UfD4ltL1Ri3mZ1S4/eYxv8Am9O1Qj4aaaBj+1tdK7skf2rN09PvV0OiaHaeH7BrSwMzI0jSs08rSuzN1JZiSa6KFCVOV3/X9f11vhXrxqRsv6/r+vLRooorsOMKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigD//2Q==)

  在这个例子中，相比于直接把`Vec`存储在`Array`变体中，如果选择只存储`Vec`的指针，这个变体需要的最大内存直接降低一半，`Box`是指向堆上数据的指针，`Box`在栈上的部分只需1个`usize`来存储堆上数据的地址，在64系统上就是8个字节。
  一个被“装箱”的`Vec`的内存布局，在函数的栈帧上，需要分配一个`usize`去存储它所指向的数据的内存地址，在堆上，需分配3个`usize`去表示`Vec`，如果`Vec`内有值，这些值也将保存在堆上，并指向具体值的指针将存储在`Vec`的指针字段中。

###### 最常见的枚举`Option<T>`

`Option<T>`用于处理空值。其他编程语言中，使用`null`关键值来表明一个变量当前值为空(非零值)，即不存在。

对`null`操作时，如调用一个方法，就会直接抛出**null 异常**，导致程序的崩溃。

rust抛弃 `null`，而改为使用 `Option` 枚举变量来处理空值。

+ 定义如下：

  ```rust
  enum Option<T> {
      Some(T),
      None,
  } 
  ```

+ `Option<T>`枚举包含两个变体分别表示：

  + 含有值：`Some(T)`

    表示该变体的数据类型是`T`（泛型参数），即`Some`可包含任何类型的数据。

  + 没有值：`None`

+ `Option<T>`在内存中布局

  ![Option](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/4RDSRXhpZgAATU0AKgAAAAgABAE7AAIAAAAEbGluAIdpAAQAAAABAAAISpydAAEAAAAIAAAQwuocAAcAAAgMAAAAPgAAAAAc6gAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFkAMAAgAAABQAABCYkAQAAgAAABQAABCskpEAAgAAAAMxNwAAkpIAAgAAAAMxNwAA6hwABwAACAwAAAiMAAAAABzqAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAyMjowNzoyMyAxNjowNjo0MAAyMDIyOjA3OjIzIDE2OjA2OjQwAAAAbABpAG4AAAD/4QsWaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49J++7vycgaWQ9J1c1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCc/Pg0KPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyI+PHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj48cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0idXVpZDpmYWY1YmRkNS1iYTNkLTExZGEtYWQzMS1kMzNkNzUxODJmMWIiIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIvPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIj48eG1wOkNyZWF0ZURhdGU+MjAyMi0wNy0yM1QxNjowNjo0MC4xNjY8L3htcDpDcmVhdGVEYXRlPjwvcmRmOkRlc2NyaXB0aW9uPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6ZGM9Imh0dHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIj48ZGM6Y3JlYXRvcj48cmRmOlNlcSB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPjxyZGY6bGk+bGluPC9yZGY6bGk+PC9yZGY6U2VxPg0KCQkJPC9kYzpjcmVhdG9yPjwvcmRmOkRlc2NyaXB0aW9uPjwvcmRmOlJERj48L3g6eG1wbWV0YT4NCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgPD94cGFja2V0IGVuZD0ndyc/Pv/bAEMABwUFBgUEBwYFBggHBwgKEQsKCQkKFQ8QDBEYFRoZGBUYFxseJyEbHSUdFxgiLiIlKCkrLCsaIC8zLyoyJyorKv/bAEMBBwgICgkKFAsLFCocGBwqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKv/AABEIAWsCzgMBIgACEQEDEQH/xAAfAAABBQEBAQEBAQAAAAAAAAAAAQIDBAUGBwgJCgv/xAC1EAACAQMDAgQDBQUEBAAAAX0BAgMABBEFEiExQQYTUWEHInEUMoGRoQgjQrHBFVLR8CQzYnKCCQoWFxgZGiUmJygpKjQ1Njc4OTpDREVGR0hJSlNUVVZXWFlaY2RlZmdoaWpzdHV2d3h5eoOEhYaHiImKkpOUlZaXmJmaoqOkpaanqKmqsrO0tba3uLm6wsPExcbHyMnK0tPU1dbX2Nna4eLj5OXm5+jp6vHy8/T19vf4+fr/xAAfAQADAQEBAQEBAQEBAAAAAAAAAQIDBAUGBwgJCgv/xAC1EQACAQIEBAMEBwUEBAABAncAAQIDEQQFITEGEkFRB2FxEyIygQgUQpGhscEJIzNS8BVictEKFiQ04SXxFxgZGiYnKCkqNTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqCg4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2dri4+Tl5ufo6ery8/T19vf4+fr/2gAMAwEAAhEDEQA/APpGiiigAooooAKKKKACiisjXNfGkSW9vBaTX15dEiK3hwCQOrEnoB60Aa9FZOla/DqKzpcRNZXVs4SaCZhlSeRg9CDVNvHGjr4mOiecXnEe/cilhnONvA4NAHRUVRi1myaFHnmW1LsVVLkiNiR6Anmrnmx7gu9dx6DPJoAdRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABWHrGkahNq1tqej3EEVzDG0TJcoWR1Jz25BrcooA8v8R+Eb261/S4ZL6KW6upJZ5zMrCKRlUBVwpzgA8VvW/hbVtLvLGbTbm2MiWwt7iaROT82cgd/TmuveGOSRHdFZkOVYjlfpT6AOIl8D3r6lJePcWd47o8a/a42bylZiQVx35o0z4dnTdRhv8A+05p7qKdXRpGOFjxgoB7129FABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUE4HNV7C+h1Kxju7UkxSjKkjGRQBYooooAKKKKACiqWp6xp+jQLNqd0lujttXd1Y+gA5J+lS2OoWupWq3NhOk8LcBlPf09jQBYoqA3tst99jaZRceX5nlk87c4z+dSRSxzxiSGRZEboyNkH8aAH0UVn2mtWt7e3cFvuZLQ7ZZyMR7u6g9yO9AGhRWf8A8JBowm8o6vY+ZnGz7Smc/TNXkdJUDxsrowyGU5BoAdRWNrPiSHSb+009IJLq/vQxggjwNwXGSWPAHIrVt3lkt43uIvJlZQWjDbtp9M96AJKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAK4PxpqviOPxTZWHh8T+Slo88wt41dmcnCAhv4cg5rvKqPpsD6xHqXzCdITDweCpOeR9aAOS/tPxgtsbR9Mka6e6Be4AXykhPYepHf8ab4CfxNbpbadq9pJHBb25SVpIlQK4IACEH5gRnk13VFAHEePL/VE1Gw0+x1KLSbeaN5HupWKBmGAE3g8dc+9VLeXxdezC803U0utPgeKOPZGP9K+ULIxJ6ANz+ddtqE2nq1vb6l5JNxJshSVQ29sE4Ge+AaZo2oQanpouLSIxQ73RVwB91iMjHbigDkZda8arZ2q2eiyPPFAVnafaBLNuUcYPC43HNaHhK58ZS3Q/wCEtt7aOKSDKrbj/VuGxyfcYP4V1tFAHL69BPB4qsdUbS5tTtordo1SEBmhkLA7tpI6jjPbFcpqj+I9O1aAabDNZvq1zJd3FvaKryRxoiqoAPBJJBNep1UuNNgudStb59wntQ4Qg44bGQfXoKAPPl0WW18Radq+q6LcX19dWeJFVywjm3gnvhRgk/hU7W3iMXjeZDqNpbpE4toNN2qhl8xvmfnoRtOPrXotFAHnklj4/vfM+0agtojubcLBGp2q0fMufZ+nsa0NH0m+jht9A8QaNbT2UQ8yO8tXIRmH99Tzk5z3rs6KAOY8P+Hra1v9Ya40uBFe9LwM0SnKbF5H61F4W0vUbTWL24RH0/RpDiDT5TuYPnlx/dB/uiusooA4fxxp0V14o8OXLpJbvDOT9vj3HygMfu8D+/0yewruB0oIB6jP1ooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigDjfGvhrV9f1KxuNLuEtm05TNbu5+UzE7TuHptz+dZ83gi/t5mS2j+07bWOG0uGujH9lYD5mC9yTzW/4l8UyaJeWtnDaAvdA4ubhtkEf+83r7cVNB4o06CaOw1PUrT+0dqmVYA3lqT055AznjJoA5S78N+LtNsbrUYtXnvL1hOnlq/CRE5Uop4Ljn86b4Y1G30BorWxnvdRn1C5jR2uLdokj4O7rj5sAk16J9ttd4T7TDuYgBfMGST0rB1jxfpVvb3Udpcw3F/bj5IyCQHLBM5xjgsM4PFAHSUVzOp+I77QtNkaTTLjUXtYQ81wCsUbk/3Tz/Kor3XdettPTU7nTI7K2t3Vp4xcLMZIjwSMAbSuc+/NAHV0UiOJI1dTlWGQaWgAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAoori/iNqraRBpU1nFdSai12ogFspYlAQZAR3G3PFAHaUVFazi6s4pwjoJEDbZFwwz2I7GpaAMPWvD91q0zGHWJ7WGRNkkHlJKjD1ww4PuKzbf4f21pbzWltfzpZXIT7RCUVmk2qF++eRworrqKAOEtfhNo9reSXi3d490zrIkryZ8sqxPyjp0O36VBofgmbT/wDQbrRbF4nVori/84l5kPOQn8LE4JOetehUUAYSeH7mXwzc6NqOoNcJIDHHPtw6p2z6ketWdY0h9V0kact20ELgJOVQEyJjleemfWtSigDI8NxX8WmyjUwyyfaJfLRjnZHuIQf984rXoooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKyNS8PQ6prVhqFxc3A+wtvjhR8IWwRkj6GteigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiua8c6pqOl6TZjSJEiuLu9S23um/AZWJwPX5axd/jAFceILQhRt/48Rgt/d69axqVo03aRtCjKaujv6K8/z4yQLnxBa7U4P+g87/7p56e9Ju8YqCG1+1wp+bFkCd3tzytZ/WqZp9VqHoNFea6nqXjHS9JvdQ/tuzmNvC0vlfYxiQqpPynPTjmvQ7KY3FhbzPjdJGrHHTJGa2p1Y1PhMalOVP4ieiiitDMKKKKACiiigAooooAKKKKACiiigAqO4uYLWEy3U0cMY6vI4UD8TUlcd4pW0HijTptft5ZtLihZowsTSIJ89WVc9un40AddFLHPEskMiyIwyrI2QfxpRIhlMYdTIBkrnkD1xXldx4g1Tw5fRxaTafY7PVLiW4jSeFn8iJVUfcXldznPtmp7MPD4xt9U1u51QS39jC0duhIAYyNlCB1AyOvbNAHp9FecLrer21wbe3Z9Ks4InMMTWrTtcSCR125IOBwPTrUn9v8Ajm9DiPTbexWR2tozIrMRIU3K59FBGKAOxuPEGmWr3SzXIH2MKZyFLCPPTOBU9jqtjqSk2N1FPgAkK3IHuOorhYtUGieF77Sda0u4066ltpWa6Y+bHcSEYJ3juSehFYNhqmsxyWmo2zWN3ONMKwixhb90crky/wB7A6D1BoA9jqK6uoLK1kubuRYoYl3O7HgCuEfxVrFlY3Bim/tRlWP7PK1oYjNKSd0IXA7Dr2zUM114l142OqXGlrc6ZGxabSkYxypIDxuLcPj06ZoA9AjvIJLRLnzAkUihlZ/l4PTrUoIZQVIIPQiuBvbzTddW9N1b3GlpNDGUubiFjsly6428jI29fes2C6g8P+Db/RZlutXtrYI/2iydw0u9zlSecYxzjsaAPUQc9KK8u8Lzxy6Dq0WqaZqejQyOLu1gj3MY0wApRhzuyMkH1q3LJ4jGj2dzOJ119naOyWMcTRA5H2gfdHHX07UAejUVFaG4NnEbwItwUHmCP7obHOPapaACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAqG6u7extzPdzJDECAXc4GScD9amrl/HehX/AIj0mDTdPfyUeYSSy5A27AWX6guFzQBuWWpw313e28IbdZyiKQkcFiobj8CKuV52PCetJ9in1BJr5nE015BZ3fkL57FdrZyCQqjHWnQ+HfGcMgvJtXeVrZ4jBaq42uM/OHOOflOPqM0AehUV5t4EmaPW1m1PWbq61O6RknsjC4EL53HfyQAMYBGMivSaAOR+IX/Hlovb/ibRfX7knT3px65zt4wSew9G/wBs+tN+IQY2WjbNuf7Wi+92+SSnKT94H1w0n839/SvMxnxo9LB/AwYleuU2DaTjJjH90+ufWldwvByhUYIX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UUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAcVqY/4ugxx/zCI87T83+uk6VqHkHIzuwCF4Dey+h9ai13wndanr8eradrM2mzrbi3by4Y3BUMzZ+YHB+Y9Kqf8Ihr+4/8VfcYJwB9ig4Hr9zr715WIwtSpUconXTqxjGzNHdkEt82TsJxgN/sex96MDByf9k+v+4f9n3rN/4RDxBnJ8X3A42Y+xQfd/7460g8IeIRtx4wuRtBX/jzg4X0+5XP9Sqmnt4EGr5GveGsgcapg+o/cS8L/s13VchZ+C9QXVbC81TxDcXqWMxmigNvEi52MvVVB6Ma6+vTwtKVKnaRy1ZKUroKKKK6jIKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAP/9k=)

  ###### **todo**

+ `Option<T>`枚举的使用

  `Option<T>`枚举由于包含`prelude`(属于rust标准库，rust会将最常用的类型、函数等提前引入其中，而不用再手动引入)之中，不需将其显示引入作用域。`Some(T)`和`None`同样，无需使用`Option::`前缀，直接使用即可。

  ```rust
  let some_number = Some(5);
  let some_string = Some("abc");
  
  // Option<T>需声明具体类型
  let absent_number:Option<i32> = None;
  ```

  当一个`Some`值时，就可知存在一个值，而这个值保存在`Some`中，当有个 `None` 值时，在某种意义上，它跟空值具有相同的意义：并没有一个有效的值。

  **注意：使用`None`时，需声明`Option<T>`具体类型，**编译器只通过`None`值无法推断出`Some`成员保存的值的类型。

+ `Option<T>`要比空值好的原因

  因为`Option<T>`和`T`(任何类型)是不同的类型。如：

  ```rust
  let x: i8 = 5;
  let y: Option<i8> = Some(5);
  let sum = x + y;
  ```

  ```shell
  error[E0277]: cannot add `Option<i8>` to `i8`
     --> src\main.rs:482:17
      |
  482 |     let sum = x + y;
      |                 ^ no implementation for `i8 + Option<i8>`
      |
      = help: the trait `Add<Option<i8>>` is not implemented for `i8`
  ```

  错误的原因是由于`x`与`y`的类型不同，rust不知如何将`Option<i8>`与`i8`相加。

  当在rust中有一个`i8`类型的值时，编译器确保它总是一个有效的值，使用时无需做**空值检查**。

  当使用`Option<i8>`时需确定是否有值，编译器会确保我们在使用值之前处理了空值的情况。

  在对`Option<T>`进行`T`的运算之前需将其转换为`T`。这能帮我们捕获到空值最常见的问题之一：期望某值不为空但实际上为空的情况。

  当一个可能为空的值时，必须显示的声明`Option<T>`，接着必须有明确处理值为空的情况。只要一个值不是`Option<T>`类型，就可以安全认定它的值不为空，这是rust一个设计决策，以提高代码的安全性。

+ 取出`Some`变体中的`T`值

  ```rust
  let x: Option<u32> = Some(2);
  
  println!("{}", x.unwrap());
  ```

  ```shell
  2
  ```

  [`Option<T>`更多的相关方法]([Option in std::option - Rust (rust-lang.org)](https://doc.rust-lang.org/std/option/enum.Option.html))

#### 流程控制

rust程序 是从上而下顺序执行的，在此过程中，我们可以通过循环、分支等流程控制方式，更好的实现相应的功能。

##### 使用`if`实现分支控制

`if else` **表达式**根据条件执行不同的代码分支。

如：若 `condition` 的值为 `true`，则执行 `A` 代码，否则执行 `B` 代码。

```rust
if condition == true {
    // A...
} else {
    // B...
}
```

+ **使用`if`语句块进行赋值**

  ```rust
  let condition = true;
  let num = if condition {
      5
  } else {
       6
  };
  
  println!("num: {}", num);
  ```

  ```shell
  num: 5
  ```

  + `if`语句块是表达式，使用`if`表达式的返回值给`num`赋值。

  + 用`if`赋值需确保每个分支返回的类型是一样。如返回类型不一致就会报错。

    ```rust
    let condition = true;
    let num = if condition {
        5
    } else {
         "six"
    };
    
    println!("num: {}", num);
    ```

    ```shell
    error[E0308]: `if` and `else` have incompatible types
       --> src\main.rs:484:10
        |
    481 |       let num = if condition {
        |  _______________-
    482 | |         5
        | |         - expected because of this
    483 | |     } else {
    484 | |          "six"
        | |          ^^^^^ expected integer, found `&str`
    485 | |     };
        | |_____- `if` and `else` have incompatible types
    ```

    + 但有个特殊情况：当配合循环中的`continue`、`break`使用时

      ```rust
      let mut v = 0;
      for i in 1..10 {
          v = if i == 9 {
              continue
          } else {
              i
          }
      }
      println!("{}", v);

##### 使用`else if`来处理多重条件

将 `else if` 与 `if`、`else` 组合在一起实现更复杂的条件分支判断：

```rust
let n = 6;
if n % 4 == 0 {
    println!("num is divisible by 4");
} else if n % 3 == 0 {
    println!("num is divisible by 3");
} else {
    println!("num is not divisible by 4 or 3");
}
```

```shell
num is divisible by 3
```

注意：就算有多个分支能匹配，也只有**第一个匹配**的分支会被执行！

#### 循环控制

在 rust 语言中有三种循环方式：**`for`**、`while` 和 `loop`。

##### `for`循环

###### `for`循环是rust的大杀器

语法如下：

```rust
for 元素 in 集合 {
    // ...
}
```

如：输出一个1~3的序列

```rust
for i in 1..=3 {
    println!("i:{}", i);
}
```

```shell
i:1
i:2
i:3
```

###### 使用时所有权转移问题

使用`for`时往往使用**集合的引用形式**，除非你在之后的代码中不继续使用该集合。不使用引用，所有权就会被转移(move)到`for`语句块中，后续无法再使用该集合。

```rust
for item in &container {
    // ...
}
```

对于实现了`copy`特征的数组(如 [i32,;10])，`for item in arr`不会发生所有权转移，只是直接对其进行拷贝，在此后的仍可使用`arr`。

###### 在循环中，修改该元素，需使用`mut`关键字

``` rust
for item in &mut collection {
    // ...
}
```

###### 总结：

| 使用方法                          | 等价使用方式                                    | 所有权     |
| --------------------------------- | ----------------------------------------------- | ---------- |
| for item in collection            | for item in IntoIterator::into_iter(collection) | 转移所有权 |
| for item in &collection           | for item in collection.iter()                   | 不可变借用 |
| for item in collection.iter_mut() | for item in collection.iter_mut()               | 可变借用   |

###### 在循环中**获取元素的索引**

```rust
let a = [4, 3, 2, 1];

// `.iter()`把数组a变成一个迭代器
for (i, v) in a.iter().enumerate() {
    println!("第{}个元素是{}", i + 1, v);
}
```

```shell
第1个元素是4
第2个元素是3
第3个元素是2
第4个元素是1
```

###### 循环控制某个过程n次

用 `_` 来替代 `i` 用于 `for` 循环中，在 Rust 中 `_` 的含义是忽略该值或者类型的意思，如果不使用 `_`，那么编译器会给你一个 `变量未使用的` 的警告。

```rust
for _ in 0..10 {
    // ...
}
```

###### **两种循环方式优劣对比**

1. 循环索引，然后通过索引下标去访问集合

   ```rust
   let collection = [1, 2, 3, 4];
   for i in o..collection.len() {
       let item = collection[i];
       // ...
   }
   ```

2. 直接循环集合中的元素

   ```rust
   let collection = [1, 2, 3, 4];
   for item in collection {
       // ...
   }
   ```

优劣如下:

+ 性能：
  + 第一种使用`collection[index]`索引访问，会因为边界检查(Bounds Checking)导致运行时的性能损耗。——rust会检查并确认 `index` 是否落在集合内。
  + 第二种直接迭代的方式就不会触发这种检查，因为编译器会在编译时就完成分析并证明这种访问是合法的
+ 安全：
  + 第一种对 `collection` 的索引访问是非连续的，存在一定可能性在两次访问之间，`collection` 发生了变化，导致脏数据产生。
  + 第二种直接迭代的方式是连续访问，因此不存在这种风险

总结：推荐第二种方式。

###### `continue`：跳过当次的循环，开始下次循环

```rust
for i in 1..4 {
    if i == 2 {
        continue;
    }
    println!("{}", i);
}
```

```shell
1
3
```

###### break：直接结束整个循环

```rust
for i in 1..4 {
    if i == 2 {
        break;
    }
    println!("{}", i);
}
```

```shell
1
```

##### `while循环`

只需要一个条件来循环，，当该条件为 `true` 时，继续循环，条件为 `false`，结束整个循环。

```rust
let mut n = 0;

while n <= 3 {
    println!("{}", n);
    n += 1;
}

println!("end");
```

```shell
0
1
2
3
end
```

**`while` vs `for`**

通过对数组中的元素的访问进行来进行对比

+ `while`

  ```rust
  let a = [1, 2, 3];
  let mut i = 0;
  while i < a.len() {
      println!("{}", a[i]);
      i += 1;
  }
  ```

   `i` 在某一时刻会到达到数组长度值，不过循环在其尝试从数组获取第六个值（会越界）之前停止，但这个过程很容易出错；如果索引长度不正确会导致程序 **panic**。这使程序更慢，因为编译器增加了运行时代码来对每次循环的每个元素进行条件检查。

+ `for`

  ```rust
  let a = [1, 2, 3];
  
  for i in a {
      println!("{}", i);
  }
  ```

  `for` 并不会使用索引去访问数组，因此更安全也更简洁，同时避免 `运行时的边界检查`，性能更高。

##### `loop`循环

`loop`循环虽然可以适用于所有循环场景，但在大多数场景下，`for`和`while`才是最优选择。

`loop`循环是一个简单的无限循环，但可在内部实现逻辑，通过`break`关键字来控制循环的结束。

```rust
// 一个无限循环
loop {
    println("again!");
}
```

该循环会不停的在终端打印输出，直到使用 `Ctrl-C` 结束程序：

```shell
again!
again!
^Cagain!
```

**`loop`循环 与 `break`关键字组合实现在某个条件下结束循环**

```rust
let mut count = 0;
let result = loop {
    count += 1;
    if count == 10 {
        break count * 2;
    }
};
println!("result: {}", result);
```

```shell
result: 20
```

`loop`循环 与 `break`关键字组合实现`while`循环

```rust
let mut n = 0;
loop {
    if n > 3 {
        break
    }
    println!("n: {}", n);
    n += 1;
};
```

```shell
n: 0
n: 1
n: 2
n: 3
```

在这种循环场景中，`while`循环要简洁的多。

**注意：**

+ `break`可以单独使用，也可带一个返回值。类似`return`
+ `loop`循环是一个表达式，可返回一个值。

#### 模式匹配

rust 中，模式匹配最常用的就是 `match` 和 `if let`

**模式匹配需类型相同**

##### `match`匹配

###### `match`的通用形式

```rust
match target {
    模式1 => 表达式1,
    模式2 => {
        语句1;
        语句2;
        表达式2
    },
    _ => 表达式3
}
```

模式匹配：将模式与 `target` 进行匹配

**注意：**

+ `match` 的匹配必须要穷举出所有可能，可使用 **`_`** 来代表未列出的所有可能性。(类似于 `switch` 中的 `default`)
+ `match` 的每一个分支都必须是一个表达式，且所有分支的表达式最终**返回值的类型必须相同**。
+ **X | Y**，类似逻辑运算符 `或`，代表该分支可以匹配 `X` 也可以匹配 `Y`，只要满足一个即可。

###### `match`的使用

`match` 允许将一个值与一系列的模式相比较，并根据相匹配的模式执行对应的代码

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Penny");
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

`match`后是一个表达式，表达式返回值可以是任意类型，只要于后面的分支中的模式匹配起来即可。这与`if`后的表达式必须是一个布尔值不同。

`match`的分支

一个分支有两个部分：

+ 模式
+ 针对该模式的处理代码：

  通过**`=> `**将模式和将要运行的代码分开

不同分支间使用**`,`**分隔。

当`match`表达式执行时，它的目标值会按顺序依次与每个分支的模式相比较，如模式匹配了这个值，则执行模式之后的代码。如模式不匹配这个值，将继续执行下个分支。

每个分支相关联的代码是一个表达式，而表达式的结果值将作为整个 `match` 表达式的返回值。

如分支的处理代码有多行时，需用** `{}`** 包裹，同时最后一行代码需是一个表达式。

###### 使用`match`表达式赋值

`match` 本身也是**一个表达式**，因此可以用它来赋值。

```rust
enum IpAddr {
    Ipv4,
    Ipv6,
}

fn main() {
    let ip = IpAddr::Ipv6;
    let ip_str = match ip {
        IpAddr::Ipv4 => "127.0.0.1",
        _ => "::1",
    };
    println!("{}", ip_str);
}
```

```shell
::1
```

###### 模式绑定

模式匹配一个重要功能是从模式中取出绑定的值。

```rust
enum Action {
    Say(String),
    MoveTo(i32, i32),
    ChangeColorRGB(u16, u16, u16),
}

fn main() {
    let actions = [
        Action::Say("Hello Rust".to_string()),
        Action::MoveTo(1,2),
        Action::ChangeColorRGB(255, 255, 0),
    ];
    for action in actions {
        match action {
            Action::Say(s) => {
                println!("{}", s);
            },
            Action::MoveTo(x, y) => {
                println!("move to ({}, {})", x, y);
            }
            Action::ChangeColorRGB(r, g, _) => {
                println!("rgb ({}, {}, 0)", r, g);
            }
        }
    }
}
```

```shell
Hello Rust
move to (1, 2)
rgb (255, 255, 0)
```

###### 穷尽匹配

`match` 的匹配必须穷尽所有情况

```rust
enum Direction {
    East,
    West,
    North,
    South,
}
fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        }
    }    
}
```

如不处理`Direction::West`的情况，会报错：

```shell
error[E0004]: non-exhaustive patterns: `West` not covered
  --> src/main.rs:37:11
   |
37 |     match dire {
   |           ^^^^ pattern `West` not covered
   |
note: `Direction` defined here
  --> src/main.rs:11:5
   |
9  | enum Direction {
   |      ---------
10 |     East,
11 |     West,
   |     ^^^^ not covered
   = note: the matched value is of type `Direction`
help: ensure that all possible cases are being handled by adding a match arm with a wildcard pattern or an explicit pattern as shown
   |
41 ~         }
42 +         West => todo!()
   |
```

###### **`_`**通配符

如需匹配除已列出的模式外的模式时，可以使用**特殊模式** `_` 替代

```rust
fn main() {
    let dire = Direction::West;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
        _ => ();
    }    
}
```

将 `_` 其放置于其他分支后，`_` 将会匹配所有遗漏的值。`()` 表示返回**单元类型**与所有分支返回值的类型相同，所以当匹配到 `_` 后，什么也不会发生。

##### `if let` 与 `while let`匹配

当**只有一个**模式的值需要被处理，其他值直接忽略的场景，可用`if let`的方式来实现。

+ `if let`

  ```rust
  let v = Some(3u8);
  if let Some(3) = v {
      println!("three");
  }
  ```

  ```shell
  three
  ```

+ `while let`

  ```rust
  let a = Some(5);
  while let Some(n) = a {
      println!("{}", n);
      break
  }
  ```

  ```shell
  5
  ```

**`if let`与`if`配合使用**

```rust
let a = Some(5);

if a.unwrap() == 5 {
    println!("a is 5");
} else if let Some(6) = a {
    println!("a is 6");
}
```

```shell
a is 5
```

**不将`if let`作为一个固定的组合**

`if let Some(6) = a`，只是一个普通的**`if`表达式**，而`let Some(6) = a`是一个**特殊布尔表达式**

**`let Some(6) = a`**

这个语句与赋值表达式很像，但有许多不同

+ 普通`let`语句格式是，`let 变量名 = 表达式`，这个是`if 匹配模式 = 变量名`

+ 普通`let`语句是进行赋值操作，而这个是一种判别等价，且带有赋值功能语句

+ 普通`let`语句，返回值恒为`()`，而该表达式返回布尔值，代表是否匹配成功。

+ 该表达式**不能单独**作为布尔表达式

  ```rust
  println!("! {:?}", let Some(42) = a)
  let b: bool = let Some(42) = a
  ```

##### `matches!`宏

`matches!`：rust标准库中提供了一个可以将一个表达式跟模式进行匹配，然后返回匹配的结果`true` or `false`。

**`matches!(expr, pattern)`**

+ expr：指条件判断的入参
+ pattern：期待为true的匹配模式

```rust
let foo = 'f';
assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));// true

let bar = Some(4);
assert!(matches!(bar, Some(x) if x > 2));// true
```

**matches!的原理**

```rust
let foo = 'f';

let t = match foo {
    'A'..='Z' | 'a'..='z' => true,
    _ => false
};
println!("t: {}", t);

let bar = Some(4);

let t1 = match bar {
    Some(x) if x > 2 => true,
    _ => false
};
println!("t1: {}", t1);
```

```she
t: true
t1: true
```

###### todo推导一个类似该`matches!`宏的自定义`matche_map`函数。

##### 变量覆盖

`match`与`if let`都可以在匹配时覆盖掉老值，绑定新值：

```rust
let age = Some(30);
println!("before age: {:?}", age);
if let Some(age) = age {
    println!("age: {:?}", age); 
}
println!("after age: {:?}", age);
```

```shell
before age: Some(30)
age: 30
after age: Some(30)
```

在`if let`中，`=` 右边的`Some(i32)`通过**解构**，给左边的`age`赋值，该`age`脱离`if`作用域后，被释放。

对于`match`也是如此：

```rust
let age = Some(30);
println!("before age: {:?}", age);
match age {
    // Some(age) = age
    Some(age) => println!("age: {:?}", age),
    _ => ()
}
println!("after age: {:?}", age);
```

```shell
before age: Some(30)
age: 30
after age: Some(30)
```

##### 解构`Option`

通过`match`来使用`Option`枚举类型

+ 匹配`Option<T>`

  使用 `Option<T>`，是为了从 `Some` 中取出其内部的 `T` 值以及处理没有值的情况。

  编写一个函数，它获取一个 `Option<i32>`，如果其中含有一个值，将其加一；如果其中没有值，则函数返回 `None` 值：

  ```rust
  fn plus_one(x: Option<i32>) -> Option<i32> {
      match x {
          None => None,
          Some(i) => Some(i + 1)
      }
  }
  
  let a = Some(5);
  let b = plus_one(a);
  let none = plus_one(None);
  println!("b: {:?}， none: {:?}", b, none);
  ```

  ```shell
  b: Some(6)， none: None
  ```

  `plus_one` 接受一个 `Option<i32>` 类型的参数，同时返回一个 `Option<i32>` 类型的值(这种形式的函数在标准库内随处所见)，在该函数的内部处理中，如果传入的是一个 `None` ，则返回一个 `None` 且不做任何处理；如果传入的是一个 `Some(i32)`，则通过模式绑定，把其中的值绑定到变量 `i` 上，然后返回 `i+1` 的值，同时用 `Some` 进行包裹。

##### 模式适用场景

###### 模式

模式是 Rust 中的特殊语法，它用来匹配类型中的结构和数据，它往往和 `match` 表达式联用，以实现强大的模式匹配能力。模式一般由以下内容组合而成：

+ 字面值
+ 解构的数组、枚举、结构体或者元组
+ 变量
+ 通配符
+ 占位符

###### 所有可能用到模式的地方

+ `match`分支

  `match` 的每个分支就是一个**模式**，因为 `match` 匹配是**穷尽式**的，有时需要一个特殊模式`_`。

  ```rust
  match VALUE {
      PATTERN => EXPRESSION,
      PATTERN => EXPRESSION,
      PATTERN => EXPRESSION,
  }
  ```

  ```rust
  match VALUE {
      PATTERN => EXPRESSION,
      PATTERN => EXPRESSION,
      _ => EXPRESSION,
  }
  ```

+ `if let`分支

  `if let` 往往用于匹配一个模式，而忽略剩下的所有模式。

  ```rust
  if let PATTERN = SOME_VALUE {
  	...
  }
  ```

+ `while let`条件循环

  一个与 `if let` 类似的结构是 `while let` 条件循环，它允许只要模式匹配就一直进行 `while` 循环。

  ```rust
  let mut stack = vec![1, 2, 3];
  
  while let Some(top) = stack.pop() {
      println!("{}", top);
  }
  ```

  ```shell
  3
  2
  1
  ```

  `pop`方法取出动态数组的最后一个元素并返回`Some(value)`，如果动态数组为空，则返回`None`，`while`循环停止。

+ `for`循环

  使用`enumerate`方法产生一个迭代器，该迭代器每次迭代会返回一个`(索引, 值)`形式的**元组**，然后与`(index, value)`匹配。

  ```rust
  let v = vec!['a', 'b', 'c'];
  
  for (index, value) in v.iter().enumerate() {
      println!("{} is at index {}", value, index);
  }
  ```

  ```shell
  a is at index 0
  b is at index 1
  c is at index 2
  ```

+ `let`语句

  ```rust
  let PATTERN = EXPRESSION;
  ```

  ```rust
  let x = 5;
  ```

  `x` 也是一种模式绑定，代表将**匹配的值绑定到变量 x 上**。在 rust 中,**变量名也是一种模式**

  ```rust
  let (x, y, z) = (1, 2, 3);
  ```

  将一个元组与模式进行匹配(**模式和值的类型必需相同！**)，然后把 `1, 2, 3` 分别绑定到 `x, y, z` 上。对于元组来说，元素个数也是类型的一部分！

+ 函数参数

  ```rust
  fn foo(x: i32) {
      // ...
  }
  ```

  函数参数也是模式，其中 `x` 就是一个模式，还可在参数中匹配元组：

  ```rust
  fn print_code(&(x, y): &(i32, i32)) {
      println!("({}, {})", x, y);
  }
  let point = (3, 5);
  print_code(&point);
  ```

  ```shell
  (3, 5)
  ```

+ **`if` 和 `if let`**

  ```rust
  let Some(x) = some_option_value
  ```

  右边的值可能不为`Some`，而是`none`，这时就不能进行匹配，也就是缺少`None`的匹配。

  + **不可驳模式匹配**

    类似 `let` 和 `for`、`match` 都必须要求完全覆盖匹配，才能通过编译( 不可驳模式匹配 )。

  + **可驳模式匹配**

    对于某些可能的值，无法进行匹配的模式，像`let if`这种可驳模式匹配，可以这样用。

    ```rust
    if let Some(x) = some_option_value {
        println!("{}", x);
    }
    ```

##### 全模式列表（总结）

###### 匹配字面值

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    _ => println!("other"),
}
```

```shell
one
```

###### 匹配命名变量

```rust
let x = Some(5);
let y = 10;

match x {
    Some(50) => println!("50"),
    Some(y) => println!("y:{:?}", y),
    _ => println!("x: {:?}", x),
}
println!("x: {:?}, y = {:?}", x, y);
```

```shell
y:5
x: Some(5), y = 10
```

该匹配模式要注意变量覆盖的问题

###### 单分支多模式

在 `match` 表达式中，可以使用 **`|`** 语法匹配多个模式，它代表 **或**的意思。

```rust
let x = 1;
match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("other"),
}
```

```she
one or two
```

###### 通过序列`..=`匹配值的范围

序列语法也可用于匹配模式，`..=` 语法允许匹配一个闭区间序列内的值。

```rust
let x = 1;
match x {
    1..=5 => println!("one through five"),
    _ => println!("other"),
}
```

```shell
one through five
```

注意：序列语法要包含**`=`**，不然会报错。

只使用`..`目前还是实验性`exclusive range pattern syntax is experimental`。rust version：1.62.1

使用序列语法只能用于数字或字符类型，由于它们可以连续的，同时编译器在编译器可以检查该序列是否为空，**字符**和**数字值**是rust中**仅有**的可以用于判断是**否为空**的类型。

```rust
let x = 'c';
match x {
    'a'..='j' => println!("a ~ j"),
    'k'..='z' => println!("k ~ z"),
    _ => println!("other"),
}
```

```shell
a ~ j
```

**在结构体中使用序列匹配**

```rust
enum Message {
    Hello { id: i32 }
}

let msg = Message::Hello{ id: 5 };

match msg {
    Message::Hello { id: 0..=5 } => {
        println!("hello");
    },
    _ => (),
}
```

```shell
hello
```

###### 解构并分解值

可使用模式来解构结构体、枚举、元组和引用。

+ **解构结构体**

  ```rust
  struct Point {
      x: i32,
      y: i32,
  }
  
  let p = Point { x: 0, y: 7 };
  
  let Point { x: a, y: b } = p;
  assert_eq!(0, a);
  assert_eq!(7, b);
  ```

  变量`a`和`b`分别匹配结构体中的`x`和`y`字段，这展示了**模式中的变量名不必与结构体中的字段名一致**。

  一般变量名与字段名一致为了便于理解，即可简写为：

  ```rust
  let p = Point {x: 0, y: 7};
  // let Point { x: x, y: y} = p;
  let Point { x, y } = p;
  assert_eq!(0, x);
  assert_eq!(7, y);
  ```

  变量`x`和`y`，与结构体`p`中的`x`和`y`字段相匹配。其结果是变量`x`和`y`包含结构体`p`中的值。

  可使用字面值作为结构体模式的**一部分**进行解构，而不是为所有的字段创建变量。这可为一些字段为特定值的同时创建其他字段的变量。

  ```rust
  match p {
      Point { x, y: 0} => println!("x: {}", x),
      Point { x: 0, y } => println!("y: {}", y),
      Point { x, y } => println!("x: {}, y: {}", x, y),
  }
  ```

  ```shell
  y: 7
  ```

  `p`因为其`x: 0`而匹配第二分支，因此打印出`y: 7`

+ **解构枚举**

  ```rust
  enum Message {
      Quit,
      Move { x: i32, y: i32 },
      Write(String),
      ChangeColor(i32, i32, i32),
  }
  
  let msg = Message::ChangeColor(0, 160, 255);
  
  match msg {
      Message::Quit => println!("Quit"),
      Message::Move { x, y } => println!("x: {}, y: {}", x, y),
      Message::Write(text) => println!("text: {:?}", text),
      Message::ChangeColor(r, g, b) => println!("r: {}, g: {}, b: {}", r, g, b),
  }
  ```

  ```shell
  r: 0, g: 160, b: 255
  ```

  + 模式匹配需类型相同，匹配`Message::Move{1,2}`必须用`Message::{x,y}`这样的同类模式。
  + 对于`Message::Quit`这种没有任何数据枚举成员值，不能进一步解构其值，只能匹配其字面值`Message::Quit`。

+ **解构嵌套的结构体和枚举**

  匹配多级的结构体或枚举。

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
  
  let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));
  
  match msg {
      Message::ChangeColor(Color::Rgb(r, g, b)) => println!("r: {}, g: {}, b: {}", r, g, b),
      Message::ChangeColor(Color::Hsv(h, s, v)) => println!("h: {}, s: {}, v: {}", h, s, v),
      _ => (),
  } 
  ```

  ```shell
  h: 0, s: 160, v: 255
  ```

  `match`第一个分支的模式匹配一个`Message::ChangeColor`枚举成员，该枚举成员又包含一个`Color::Rgb`的枚举成员，最终绑定了3个内部的`i32`值。

+ **解构结构体和元组**

  可用复杂方式来混合、匹配和嵌套解构模式。

  ```rust
  struct Point {
      x: i32,
      y: i32,
  }
  
  let ((feet, inches), Point {x, y}) = ((3, 10), Point {x: 3, y: -10});
  println!("feet: {}, inches: {}, x: {}, y: {}", feet, inches, x, y);
  ```

  ```shell
  feet: 3, inches: 10, x: 3, y: -10
  ```

+ **忽略模式中的值**

  可使用**`_`模式**或**`..`**忽略所剩部分的值。

  + **使用`_`忽略整个值**

    `_`模式作为`match`表达式的通配符外，还又其他作用。

    ```rust
    fn foo(_: i32, y: i32) {
        println!("y: {}", y);
    }
    foo(3, 4);
    ```

    ```shell
    y: 4
    ```

    `foo`会完全忽略作为第一个参数传递的值`3`，并打印输出`y: 4`

    大部分情况当你不再需要特定函数参数时，最好修改签名不再包含无用的参数。在一些情况下忽略函数参数会变得特别有用，比如实现特征时，当你需要特定类型签名但是函数实现并不需要某个参数时。此时编译器就**不会警告说存在未使用的函数参数**，就跟使用命名参数一样。

  + **使用嵌套的`_`忽略部分值**

    + 在一个模式内部使用 `_` 忽略部分值。

      ```rust
      let mut a = Some(5);
      let b = Some(10);
      
      match (a, b) {
          (Some(_), Some(_)) => {
              println!("Can't overwrite an existing customized value");
          }
          _ => {
              a = b;
          }
      }
      println!("a: {:?}", a);
      ```

      ```shell
      Can't overwrite an existing customized value
      a: Some(5)
      ```

      第一个匹配分支，只关心元组中两个元素的类型，对`Some`中的值，直接进行忽略。

    + 还可在一个模式中的多处使用下划线来忽略特定值

      ```rust
      let n = (1, 2, 3, 4, 5);
      
      match n {
          (first, _, third, _, fifth) => {
              println!("first: {}, third: {}, fifth: {}", first, third, fifth);
          },
      }
      ```

      ```shell
      first: 1, third: 3, fifth: 5
      ```

      模式匹配需类型相同，匹配 `n` 元组的模式，也必须有五个值（**元组中元素的数量也属于元组类型的一部分**）。

  + **使用`_`开头忽略未使用的变量**

    当创建了变量而未被使用时，rust通常会给警告。但有时创建一个不会被使用的变量是有用的，可用`_`作为变量名的开头。

    ```rust
    let _x = 1;
    let y = 2;
    ```

    ```shell
    warning: unused variable: `y`
     --> src/main.rs:3:9
      |
    3 |     let y = 2;
      |         ^ help: if this is intentional, prefix it with an underscore: `_y`
      |
      = note: `#[warn(unused_variables)]` on by default
    ```

    这里警告说未使用变量`y`，至于`x`则没有警告。

    **注意**：只使用`_`与使用以`_`开头的变量名的区别是：`_x`仍会将值绑定到变量，而`_`则完全不会绑定。

    + **`_x`仍会将值绑定到变量**

      ```rust
      let s = Some(String::from("hello"));
      
      if let Some(_s) = s {
          println!("a string");
      }
      println!("{:?}", s);
      ```

      ```shell
      error[E0382]: borrow of partially moved value: `s`
       --> src/main.rs:7:22
        |
      4 |     if let Some(_s) = s {
        |                 -- value partially moved here
      ...
      7 |     println!("{:?}", s);
        |                      ^ value borrowed here after partial move
      ```

      `s`是一个拥有所有权的动态字符串，由于`s`的值被转移给`_s`，再使用`s`会报错。

    + **只使用`_`则不会绑定**

      ```rust
      let s = Some(String::from("hello"));
      
      if let Some(_) = s {
          println!("a string");
      }
      println!("{:?}", s);
      ```

      ```shell
      a string
      Some("hello")
      ```

      `s`没被移动进`_`。

  + **用`..`忽略剩余值**

    对于有多个部分的值，可以使用`..` 语法来只使用部分值而忽略其它值，不用再为每一个被忽略的值都单独列出下划线。

    `..` 模式会忽略模式中剩余的任何没有显式匹配的值部分。

    ```rust
    struct Point { 
        x: i32,
        y: i32,
        z: i32,
    }
    
    let origin = Point { x: 0, y: 1, z: 2 };
    
    match origin {
        Point { x, .. } => println!("x: {}", x),
    }
    
    let n = (1, 2, 3, 4);
    
    match n {
        (first, .., last) => println!("{}, {}", first, last),
    }
    ```

    ```shell
    x: 0
    1, 4
    ```

    使用`..`要简洁的多。

    但要注意使用`..`必须是无歧义的。如期望匹配和忽略的值是不明确的，Rust 会报错。

    ```rust
    let n = (1, 2, 3, 4);
    
    match n {
        (.., second, ..) => println!("{}", second),
    }
    ```

    ```shell
    error: `..` can only be used once per tuple pattern
     --> src/main.rs:5:22
      |
    5 |         (.., second, ..) => println!("{}", second),
      |          --          ^^ can only be used once per tuple pattern
      |          |
      |          previously used here
    ```

    Rust 无法判断，`second` 应该匹配 `n` 中的第几个元素，这里使用两个 `..` 模式，是有很大歧义的！所以报错。

+ **匹配守卫提供的额外条件**

  **匹配守卫**（*match guard*）是一个位于 `match` 分支模式之后的额外 `if` 条件，它能为分支模式提供更进一步的匹配条件。

  ```rust
  let num = Some(4);
  
  match num {
      Some(x) if x < 5 => println!("less than five: {}", x),
      Some(x) => println!("{}", x),
      None => (),
  }
  ```

  ```shell
  less than five: 4
  ```

  当`num`与模式中第一个分支匹配时，`Some(4)`可与`Some(x)`匹配，接着匹配模式守卫`x < 5`，匹配后第一分支被选择。

  如果匹配守卫为假，rust就会继续前往下一分支。

  **使用匹配守卫解决模式中变量覆盖问题**

  

  ```rust
  let x = Some(5);
  let y = 10;
  
  match x {
      Some(50) => println!("50"),
      Some(n) if n == y => println!("n: {}", n),
      _ => println!("x: {:?}, y: {}", x, y),
  }
  ```

  ```shell
  x: Some(5), y: 10
  ```

  第二个匹配分支中的模式，不会引入一个新变量`y`来覆盖外部变量`y`，这意味着可以在匹配守卫中使用外部变量 `y`。

  相比指定会覆盖外部变量 `y` 的模式 `Some(y)`，这里指定为 `Some(n)`。此新建变量 `n` 并没有覆盖任何值，因为 `match` 外部没有变量 `n`。

  匹配守卫 `if n == y` 并不是一个模式所以没有引入新变量。这个 `y` **是** 外部变量 `y` 而不是新的覆盖变量 `y`，这样就可以通过比较 `n` 和 `y` 来表达寻找一个与外部 `y` 相同的值的概念了。

  **在匹配守卫中使用 或 运算符 `|` 来指定多个模式**

  **同时匹配**守卫的条件会作用于**所有**的模式。

  ```rust
  let x = 4;
  let y = false;
  
  match x {
      4 | 5 | 6 if y => println!("yes"),
      _ => println!("no"),
  }
  ```

  ```shell
  no
  ```

  匹配守卫与 `|` 的优先级，在满足4 | 5 | 6 后才会判断`y`是否为`true`;

  匹配守卫作用于多个模式时的优先级规则:

  ```rust
  (4 | 5 | 6) if y => ...
  ```

###### `@`绑定

`@`（读作 at）运算符允许为一个字段绑定另外一个变量。

```rust
enum Message {
    Hello { id: i32 }
}

let msg = Message::Hello{ id: 5 };

match msg {
    Message::Hello { id: id_var @ 0..=5 } => {
        println!("id_var: {}", id_var);
    },
    _ => (),
}
```

```shell
id_var: 5
```

`Message::Hello `的`id`字段的值匹配`0..=5`范围，同时将其值绑定到`id_var`变量中以便**此分支中**可以使用。这里也可以将`id_var`命名为`id`与字段同名。

###### `@`前绑定后解构（rust 1.56 新增）

使用`@`可在绑定新变量同时对目标进行解构。

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
} 

let p @ Point { x: px, y: py } = Point { x: 10, y: 20 };
println!("x: {}, y: {}", px, py);
println!("{:?}", p);

let point = Point { x: 10, y: 5 };
if let p @ Point { x: 10, y } = point {
    println!("y is {} in {:?}", y, p);
} else {
    println!("x is not 10");
}
```

```shell
x: 10, y: 20
Point { x: 10, y: 20 }
y is 5 in Point { x: 10, y: 5 }
```

###### `@`新特性（rust 1.53新增）

```rust
match 1 {
    num @ ( 1 | 2 ) => {
        println!("{}", num);
    }
    _ => {}
}
```

如`num` 没有绑定(`@`)到所有模式上，则会报错

```rust
num @  1 | 2 
```

这样只绑定模式`1`，编译不通过报错。

#### 方法 Method

rust 的方法往往跟结构体、枚举、特征(Trait)一起使用。

##### 使用`impl`定义方法

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    // new 是Circle 的关联函数，因为它的第一个参数不是self，且new并不是关键字
    // 这种方法常用于初始化当前结构体的实例
    fn new (x: f64, y: f64, radius: f64) -> Circle {
        Circle {
            x,
            y,
            radius,
        }
    }

    // Circle 的方法，&self表示借用当前的Circle结构体
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

其他语言所有定义都在`class`中，rust的对象定义和方法是分离的，这样给予使用者极高的灵活度。

![impl](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/4RDSRXhpZgAATU0AKgAAAAgABAE7AAIAAAAEbGluAIdpAAQAAAABAAAISpydAAEAAAAIAAAQwuocAAcAAAgMAAAAPgAAAAAc6gAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFkAMAAgAAABQAABCYkAQAAgAAABQAABCskpEAAgAAAAM0MgAAkpIAAgAAAAM0MgAA6hwABwAACAwAAAiMAAAAABzqAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAyMjowODowMyAyMjowNzozMQAyMDIyOjA4OjAzIDIyOjA3OjMxAAAAbABpAG4AAAD/4QsWaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49J++7vycgaWQ9J1c1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCc/Pg0KPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyI+PHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj48cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0idXVpZDpmYWY1YmRkNS1iYTNkLTExZGEtYWQzMS1kMzNkNzUxODJmMWIiIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIvPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIj48eG1wOkNyZWF0ZURhdGU+MjAyMi0wOC0wM1QyMjowNzozMS40MTY8L3htcDpDcmVhdGVEYXRlPjwvcmRmOkRlc2NyaXB0aW9uPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6ZGM9Imh0dHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIj48ZGM6Y3JlYXRvcj48cmRmOlNlcSB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPjxyZGY6bGk+bGluPC9yZGY6bGk+PC9yZGY6U2VxPg0KCQkJPC9kYzpjcmVhdG9yPjwvcmRmOkRlc2NyaXB0aW9uPjwvcmRmOlJERj48L3g6eG1wbWV0YT4NCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgPD94cGFja2V0IGVuZD0ndyc/Pv/bAEMABwUFBgUEBwYFBggHBwgKEQsKCQkKFQ8QDBEY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uVis/BenaOrOPtM8l0G3DuVCKDn6mgDG0Lxp4l1bxKkOvanY+GvLmKnS7izO6df9mVjg/hXqwOelecaz8OfEWpLJZp4tEumysSYr6xS4kTPZZDyMdq7jQ9L/ALF0O0077RJc/Z4wnmyn5mx60AX655v+SkD/ALBg/wDRjV0Nc83/ACUgf9gwf+jGoA6GiiigAooooAKKKKACiiigDn9N/wCSga9/16WX8566Cuf03/koGvf9ell/OeugoAKKKKACiiigAooooAK57xxrlz4d8MtqVpGkhjuIVkV1yNjSBW/HBroaiubaC8t2gu4UmibG5JFDKec9DQB49d6l428Xat4lsNG1STTdO0udpI7uIYkZlQFYRjHGeT9cV6d4S1Ztc8I6bqMjBpLi3VnI7tjn9a1Y4Iot3lRom85bauNx96bbWsFnAsFpCkMS/dSNQoH4CgDifipcynT9G0hLmW2h1bUo7a4kifY3l4JIB7ZwBWX4k8Lr8PPB99qPgy91KzEaIJLfz2mjALjdJh9xBAz0xXpF1Y2t95X2y3in8mQSR+YgbYw6MM9DUzosiFHUMrDBUjIIoA848U/E2xstP0i20DWLe6vr24gWSaLbKsUZYBmfHC56c461i+IfDWpah4uvptUuZPFumIQx0uz1E289lnkfu1IDcepBNenQeGNCtbaW3ttHsYoZjmSNLdQrnOckY55rN8QfDrwt4ouFuNZ0mOWcADzY3aJyB0BKkEj60Actruj+D/GvgvVNZt9O869s7GS2H2lGE1syKcKVbow9f1rf8JatpNjovhzTYlVbq/sg8axIOiqMlsdOvWtrR/C+jaBpL6ZpNhFb2kmfMjAzvyMHJPJ/GqnhzwJ4c8KTSTaFpkdvNJkNIWLtjOcAsSQPagDoaKKKACiiigAooooAKKKKACue0H/kavFP/X5B/wCk0VdDXPaD/wAjV4p/6/IP/SaKgDoaKKKACiiigAooooA5++/5KBpP/XncfzSugrn77/koGk/9edx/NK6CgAooooAKKKKACiiigAooooAKKKKACiiigAooooAbLGs0LxyDKupVh7GuG0P4ZR6H4pj1CLV7ufT7cu9rYTMWWF26kH0ru6KACiiigArjPGXw9Hia9W/sNWu9KvvL8iSSFiVliPVSv9a7OigCrplhHpel21jBkx28YjUnqQBVqiigAooooAK5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FAHN/wBk+Kv+hli/8AUo/snxV/0MsX/gCldJRQBzf9k+Kv8AoZYv/AFKP7J8Vf8AQyxf+AKV0lFAHN/2T4q/6GWL/wAAUo/snxV/0MsX/gCldJRQBzf9k+Kv+hli/wDAFKP7J8Vf9DLF/wCAKV0lFAHN/wBk+Kv+hli/8AUo/snxV/0MsX/gCldJRQBzf9k+Kv8AoZYv/AFKt6Botzpct/cX999tub6ZZZHEQQDaioAAPZa2aKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAP/9k=)

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
}

let rect = Rectangle { width: 30, height: 50 };

println!("rect area is {}", rect.area());
```

```shell
rect area is 1500
```

该例子定义一个`Rectangle`结构体，并在其上定义一个`area`方法，用于计算该矩形的面积。

`impl Rectangle`：为`Rectangle`实现方法(`impl`: *implementation*(实现))，表明`impl`语句块中的一切都是与`Rectangle`相关联的。

##### `self`、`&self` 和 `&mut self`

在`area`的签名中，使用`&self`替代`rectangle: &Rectangle`，`&self`其实是`self: &Self`的简写（注意大小写）。

在一个`impl`块内，`Self`指代被实现方法的结构体类型(`struct Rectangle`)，`self`指代此类型的实例，即`self`指代的是`Rectangle`结构体实例(`rect`)，这样的写法更加简洁，且便于理解。

当`self`用作函数的第一个参数时，它等价于`self: Self`；`&self`参数等价于`self: &Self`；`&mut self`参数等价于`self: &mut Self`

**`self`的所有权问题：**

+ `self`表示`Rectangle`的所有权转移到该方法中，
+ `&self`表示该方法对`Rectangle`的不可变借用
+ `&mut self`表示可变借用

`self`的使用跟函数参数一样，要严格遵守 Rust 的所有权规则。

使用方法代替函数的好处：

+ 不用在函数签名中重复书写`self`对应的类型
+ 代码的组织性和内聚性更强，对于代码维护和阅读有巨大好处。

###### rust的自动引用和解引用的功能

如果在一个**对象的指针**上调用方法，这是需先解引用指针。如：`object`是一个指针，要调用`something`方法需要`(*object).something()`

rust有一个**自动引用和解引用**的功能，方法调用时rust中少有这种行为的地方。

当使用 `object.something()` 调用方法时，rust 会自动为 `object` 添加 `&`、`&mut` 或 `*` 以便使 `object` 与方法签名匹配。

```rust
p1.distance(&p2);
(&p1).distance(&p2);
```

以上两行代码是等价的。第一行要简洁的多，这种自动引用的行为之所以有效，是因为**方法有一个明确的接收者**— **`self` 的类型**（例如：p1）。

在给出**接收者**和**方法名**的前提下，rust 可以明确地计算出方法是仅仅读取（`&self`），做出修改（`&mut self`）或者是获取所有权（`self`）。事实上，rust 对方法接收者的隐式借用让所有权在实践中更友好。

##### 方法名允许于结构体字段名相同

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

let rect1 = Rectangle {
    width: 30,
    height: 50,
};

if rect1.width() {
    println!("width is {}", rect1.width);
}
```

当使用`rect1.width()`时，调用的是它的方法，使用`rect1.width`，则是访问它的字段。

###### 实现 `getter` 访问器

一般来说，方法跟字段同名，往往适用于实现 `getter` 访问器。

```rust
pub struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    pub fn new(width: u32, height: u32) -> Self {
        Rectangle { width, height }
    }
    pub fn width(&self) -> u32 {
        return self.width;
    }
}

let rect1 = Rectangle::new(30, 50);

println!("{}", rect1.width());
```

```shell
30
```

用这种方式，可以把`Rectangle`的字段设置为私有属性，自需把它的`new`和`width`方法设置为公开可见。当用户访问字段rect1.width字段时，就会报错。

注意在此例，`Self` 指代的就是被实现方法的结构体 `Rectangle`。

##### 带有多个参数的方法

```rust
struct Rectangle {
    w: u32,
    h: u32,
}

impl Rectangle {
    fn new(w: u32, h: u32) -> Self {
        Rectangle { w, h }
    }
    fn area(&self) -> u32 {
        self.w * self.h
    }
    fn hold(&self, other: &Rectangle) -> bool {
        self.w > other.w && self.h > other.h
    }
}

let rect1 = Rectangle::new(30, 50);
let rect2 = Rectangle::new(10, 40);

println!("Can rect1 hold rect2？{}", rect1.hold(&rect2));
```

##### 关联函数

关联函数指定义在`impl`中且没有使用`self`的函数。由于没有`self`，不能用`.`的形式调用，**需用`::`的形式调用**。它是一个函数不是方法，但又在`impl`中，于结构体紧密关联，因此称为关联函数。

如：`String::from`，用于创建一个动态字符串

```rust
impl Rectangle {
    fn new(w: u32, h: u32) -> Self {
        Rectangle { w, h }
    }
}
```

###### 构造器 `new`

rust中有一个约定俗成的规则，使用 **`new`** 来作为构造器的名称，出于设计上的考虑，Rust 特地没有用 `new` 作为关键字。

因为是函数，所以不能用 `.` 的方式来调用，我们需要用 `::` 来调用，例如 `let sq = Rectangle::new(3, 3);`。这个方法位于结构体的命名空间中：`::` 语法用于关联函数和模块创建的命名空间。

##### 多个`impl`定义

rust 允许为一个结构体定义多个 `impl` 块，目的是提供更多的灵活性和代码组织性。

例如当方法多了后，可以把相关的方法组织在同一个 `impl` 块中，那么就可以形成多个 `impl` 块，各自完成一块儿目标：

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.w * self.h
    }
}

impl Rectangle {
    fn hold(&self, other: &Rectangle) -> bool {
        self.w > other.w && self.h > other.h
    }
}
```

##### 枚举实现方法

枚举类型之所以强大，不仅在于它好用、可以同一化类型，还在于可像结构体一样，为枚举实现方法：

```rust
#[derive(Debug)]
#[allow(unused)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        match self {
            Message::Quit => println!("Quit"),
            Message::Move { x, y } => println!("x: {}, y: {}", x, y),
            Message::Write(str) => println!("str: {:?}", str),
            Message::ChangeColor(r, g, b) => println!("r: {}, g: {}, b: {}", r, g, b),
        }
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

```shell
str: "hello"
```

#### 泛型和特征

##### 泛型 `Generics`

###### 多态

指为不同数据类型的实体提供统一的接口。

泛型也是一种多态。泛型的目的是为编程提供便利，减少代码的臃肿，同时丰富语言的表达能力。

###### 泛型详解

在rust中，出于惯例一般用**`T`**(type)表示泛型参数。

**使用泛型参数，需在使用前对其进行声明**。

```rust
fn largest<T>(list: &[T]) -> T {}
```

该泛型函数的作用是从列表中找出最大值，其中列表中的元素类型为**`T`**。首先`largest<T>`对泛型参数`T`进行了声明，然后才在函数参数中进行使用该泛型参数`list: &[T]`。

**注意：**泛型参数命名需**大写开头**

可理解为：函数 `largest` 有泛型类型 `T`，它有个参数 `list`，其类型是元素为 `T` 的数组切片，最后，该函数返回值的类型也是 `T`。

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item
        }
    }
    largest
}

let n_list = vec![1, 3, 6, 2, 7, 8];
let result = largest(&n_list);
println!("largest is {}", result);

let c_list = ['y', 'm', 'a', 'q'];
let result1 = largest(&c_list);
println!("largest is {}", result1);
```

运行之后会报错：

```shell
error[E0369]: binary operation `>` cannot be applied to type `T`
// `>`操作符不能用于类型`T`
 --> src/main.rs:6:21
  |
6 |             if item > largest {
  |                ---- ^ ------- T
  |                |
  |                T
  |
help: consider restricting type parameter `T`
// 考虑对T进行类型限制
  |
2 |     fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
  |                 ++++++++++++++++++++++
```

由于`T`可以是任何类型，但不是所有类型都是能进行比较的，因此编译器建议给`T`添加一个类型限制：使用`std::cmp::PartialOrd`特征(Trait)对`T`进行限制。该特征就是让**类型实现可比较的功能**。

由于`std::cmp::PartialOrd` 位于 prelude 中所以并不需要手动将其引入作用域，直接使用即可。

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T {}
```

但还是会报错：

```shell
error[E0508]: cannot move out of type `[T]`, a non-copy slice
 --> src/main.rs:3:27
  |
3 |         let mut largest = list[0];
  |                           ^^^^^^^
  |                           |
  |                           cannot move out of here
  |                           move occurs because `list[_]` has type `T`, which does not implement the `Copy` trait
  |                           help: consider borrowing here: `&list[0]`

error[E0507]: cannot move out of a shared reference
 --> src/main.rs:5:22
  |
5 |         for &item in list.iter() {
  |             -----    ^^^^^^^^^^^
  |             ||
  |             |data moved here
  |             |move occurs because `item` has type `T`, which does not implement the `Copy` trait
  |             help: consider removing the `&`: `item`
```

由于像`i32`和`char`这种实现了`copy trait`(已知大小并存储在栈上)的类型，但`largest`函数使用的是泛型，参数`list`就可能没有实现`copy trait`。因此可能不能移动`list[0]`的值。

可通过在签名添加(+)**`Copy`**来限制只能用于实现了`copy trait`的类型。

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {}
```

```shell
largest is 8
largest is y
```

例如：实现一个`add`泛型函数

```rust
fn add<T> (a: T, b: T) {
    a + b
}

println!("1 + 3 = {}", add(1, 3));
```

这样运行会报错

```shell
error[E0369]: cannot add `T` to `T`
 --> src/main.rs:3:11
  |
3 |         a + b
  |         - ^ - T
  |         |
  |         T
  |
help: consider restricting type parameter `T`
  |
2 |     fn add<T: std::ops::Add> (a: T, b: T) -> T {
  |             +++++++++++++++
```

同样的，不是所有`T`类型都能进行相加操作，因此需用`std::ops::Add<Output = T>` 对 `T` 进行限制：

```rust
fn add<T: std::ops::Add<Output =T>> (a: T, b: T) -> T {
    a + b
}
```

```shell
1 + 3 = 4
```

###### 结构体中使用泛型

结构体中的字段类型也可以用泛型来定义

```rust
struct Point<T> {
    x: T,
    y: T,
}

let integer = Point { x: 5, y: 10 };
let float = Point { x: 1.0, y: 4.0 };

println!("integer x: {}, y: {}", integer.x, integer.y);
println!("float x: {:?}, y: {:?}", float.x, float.y);
```

```shell
integer x: 5, y: 10
float x: 1.0, y: 4.0
```

要注意两点：

+ **提前声明**，与泛型函数定义类型，在使用泛型参数之前必需先声明`Point<T>`，就可以在结构体的字段类型中使用`T`来代替具体类型。

+ `x`和`y`是**相同**的类型

  如使用不同类型则会报错：

  ```rust
  let p = Point { x: 1, y: 1.1 };
  ```

  ```shell
  error[E0308]: mismatched types// 类型不匹配
   --> src/main.rs:7:30
    |
  7 |     let p = Point { x: 1, y: 1.1 };
    |                              ^^^ expected integer, found floating-point number
  ```

  当`x`被赋值`1`时，变量`p`的`T`类型就被确定为整数类型，`y`必须为整数类型，因此报错。

**在结构体中使用多个泛型参数**

如想让`x`和`y`即能类型不相同，需使用不同的泛型参数

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

let p = Point { x: 1, y: 1.1 };
println!("x: {}, y: {:?}", p.x, p.y);
```

```shell
x: 1, y: 1.1
```

**注意**：所有的泛型参数都需提前声明。当泛型参数个数过多时需考虑拆分这个结构体，减少泛型参数的个数和代码复杂度。

###### 枚举中使用泛型

`Option<T>`是一个拥有泛型`T`的枚举类型，用于确定值的存在。

```rust
enum Option<T> {
    Some(T),
    None,
}
```

`Result<T, E>`关注的主要是值的正确性。

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

如函数正常运行，则最后返回一个 `Ok(T)`，`T` 是函数具体的返回值类型，如果函数异常运行，则返回一个 `Err(E)`，`E` 是错误类型。

例如打开一个文件：如果成功打开文件，则返回 Ok(std::fs::File)，因此 T 对应的是 std::fs::File 类型；而当打开文件时出现问题时，返回 Er

r(std::io::Error)，E 对应的就是 std::io::Error 类型

```rust
use std::fs::File;

let f = File::open("hello.txt");
match f {
    Ok(file) => println!("file: {:?}", file),
    Err(err) => panic!("{:?}", err),
}
```

```shell
file: File { fd: 3, path: "/mnt/e/Linux/rustTest/hello/src/hello.txt", read: true, write: false }
```

###### 方法中使用泛型

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

let p = Point { x: 2, y: 4 };
println!("p.x: {}", p.x());
```

```shell
p.x: 2
```

使用泛型参数之前，依然需提前声明：`impl<T>`，只有提前声明了，才能在`Point<T>`中使用`T`，这样rust就确定`Point`的尖括号中的类型是泛型而不是具体类型。需注意这里的`Point<T>`不是泛型声明，而是一个完整的结构体类型。

**在该结构体的方法中定义额外的泛型参数**，就跟泛型函数一样：

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y
        }
    }
}

let p1 = Point{ x: 5, y: 16.5 };
let p2 = Point{ x: 1.5, y: 4 };

let p3 = p1.mixup(p2);

println!("p3.x: {}, p3.y: {}", p3.x, p3.y);
```

```rust
p3.x: 5, p3.y: 
```

`<T, U>`是定义在结构体`Point`上的泛型参数，是结构体泛型；

`<V, W>`是单独定义在方法`mixup`上的泛型参数，是函数泛型。

###### 为具体的泛型类型实现方法

对于 `Point<T>` 类型，不仅能定义基于 `T` 的方法，还能针对特定的具体类型，进行方法定义：

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl Point<f32> {
    fn distance(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

let p = Point{ x: 1.2, y: 4.6 };
println!("distance: {}", p.distance());
```

```shell
distance: 4.753946
```

这意味着`Point<f32>`类型会有一个方法`distance`，其他`T`不是`f32`类型的`Point<T>`实例则没有定义此方法。

这样就能针对特定的泛型类型实现某个特定的方法，对于其它泛型类型则**没有定义**该方法。

###### `const`泛型（rust v.1.5.1引入）

之前的泛型中都是相对于**类型**，针对类型实现的泛型，所有的泛型都是为了抽象不同的类型。

**值使用泛型**

如：`[i32; 2]` 和 `[i32; 3]` 是不同的数组类型，无法用同一个函数调用。

1. 让`display_arr`能打印任意长度的`i32`数组

   ```rust
   fn display_arr(arr: &[i32]) {
       println!("{:?}", arr);
   }
   
   let arr1: [i32; 3] = [1, 2, 3];
   let arr2: [i32; 2] = [1, 3];
   
   display_arr(&arr1);
   display_arr(&arr2);
   ```

2. 将 `i32` 改成所有类型的数组

   ```rust
   fn display_arr<T: std::fmt::Debug>(arr: &[T]) {
       println!("{:?}", arr);
   }
   
   let arr1: [i32; 3] = [1, 2, 3];
   let arr2: [i32; 2] = [1, 3];
   
   display_arr(&arr1);
   display_arr(&arr2);
   ```

   需对`T`加一个限制`std::fmt::Debug`，表明`T`可用在 `println!("{:?}", arr)` 中，因为 `{:?}` 形式的格式化输出需要 `arr` 实现该特征。

3. 使用针对值的泛型（**`const`泛型**），处理数组长度的问题

   ```rust
   fn display_arr<T: std::fmt::Debug, const N: usize>(arr: &[T; N]) {
       println!("{:?}", arr);
   }
   
   let arr1: [i32; 3] = [1, 2, 3];
   let arr2: [i32; 2] = [1, 3];
   
   display_arr(&arr1);
   display_arr(&arr2);
   ```

   ```shell
   [1, 2, 3]
   [1, 3]
   ```

   定义了一个类型为`[T;N]`的数组，其中`T`是基于类型的泛型参数；`N`是`const`泛型，定义语法是`const N: uszie`，表示`const`泛型`N`，它基于的值的类型是`unsize`。

   **注意：**`const`泛型参数命名需**大写开头**

   ###### `const`泛型表达式

   const 参数不能使用复杂的通用表达式，仅能由以下形式的 const 参数实例化：

   + 一个独立的`const`参数

     ```rust
     fn foo<const N: usize>() {
         println!("N: {}", N);
     }
     
     fn bar<const M: usize>() {
         // `M` is a const parameter
         // 注意这里使用的::
         foo::<M>();
     }
     // 注意这里使用的::
     bar::<10>();
     ```
     
     ```shell
     N: 10
     ```
     
   + 一个文字（即一个整数、布尔或字符）

     ```rust
     fn print_num<const N: usize>() {
         println!("N: {}", N);
     }   
     print_num::<10>();
     
     fn print_char<const C: char>() {
         println!("C: {:?}", C);
     }
     print_char::<'z'>();
     
     fn print_bool<const B: bool>() {
         println!("B: {}", B);
     }
     print_bool::<true>();
     ```
     
     ```shell
     N: 10
     C: 'z'
     B: true
     ```
     
   + 一个**具体的`const`表达式**（用**`{}`**括起来），不涉及通用参数
   
     ```rust
     fn foo<const N: usize>() {
         println!("N: {}", N);
     }
     
     const A: usize = 10;
     foo::<{ A }>();
     foo::<{ A + 1 }>();
     foo::<{ 20 * 100 + 1 }>();
     ```
     
     ```shell
     N: 10
     N: 11
     N: 2001
     ```
     
   

###### 泛型的性能

在rust中泛型是零成本的抽象，在使用时不用担心性能上的问题。rust在编译期为泛型对应的多个类型，生成各自的代码，因此损失了编译速度和增大了最终生成文件的大小。

具体来说：

Rust 通过在编译时进行泛型代码的 **单态化**(*monomorphization*)来保证效率。单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。

编译器所做的工作与创建泛型函数的步骤相反，编译器寻找所有泛型代码被调用的位置并针对具体类型生成代码。

如：`Option<T>`

```rust
let int =Some(5);
let float = Some(5.0);
```

当编译这段代码时，它会进行单态化。编译器会读取传递给 `Option<T>` 的值并发现有两种类型 `Option<T>`分别为 `i32`与  `f64`。进而将泛型定义`Option<T>`展开为`Option_i32`与`Option_f64`，接着泛型定义将被替换为这两个具体的定义。

编译器生成的单态化版本的代码：

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

由于单态化的存在，使用泛型时没有**运行时**开销；当代码运行，它的执行效率就跟好像手写每个具体定义的重复代码一样。这个单态化过程正是 rust 泛型在运行时极其高效的原因。

##### 特征 Trait

> Trait可以作用在一个预定义类型上，给它**添加**某些**特性**，并无需修改该类型本身的所有声明和原始定义的代码，就可为该类型添加功能(function)的特性，使得代码上写法更加灵活。

###### 定义特征

如果不同类型具有相同的行为，那么就可以把这些行为抽象出来，然后为这些类型实现该特征。

**定义特征**是把一些方法组合在一起，目的是定义一个实现某些功能所必需的行为的集合。

如：分别计算矩形与圆形的面积，我们可以总结这个行为就是共享的，可用特征来定义：

```rust
// 定义一个特征
pub trait Geometry {
    fn area(&self) -> f64;
}
```

+ 使用**`trait`**关键字来声明一个特征
+ `Geometry`是特征名，首字母需**大写**
+ 大括号中定义了该特征的所有方法，**只定义特征方法签名，而不进行实现**，此时方法签名结尾是**`;`**。

###### 为类型实现特征

每一个实现这个特征的类型都需具体实现该特征的相应方法。编译器也会确保任何实现`Geometry`特征的类型都拥有与这个签名定义一致的`area`方法。

```rust
#[derive(Debug)]
struct Rectangle {
    width: f64,
    height: f64,
}

struct Circle {
    radius: f64,
}

// 分别为Rectangle 与 Circle实现Geometry特征
impl Geometry for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.height
    }
}

impl Geometry for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

`impl Geometry for Rectangle`：为`Rectangle`类型实现`Geometry`特征。`impl`的花括号中实现该特征的具体方法。

###### 在类型实例上调用特征的方法：

```rust
// 使用对应特性
let rectangle = Rectangle { width: 1.5, height: 2.5 };
println!("rectangel area is {:?}", rectangle.area());

let circle = Circle { radius: 2.5 };
println!("circle area is {:?}", circle.area());
```

```shell
rectangel area is 3.75
circle area is 19.634954084936208
```

###### 特征定义与实现的位置(孤儿规则)

特征实现与定义的位置，有一条重要原则：**如要为类型`A`实现特征`T`，那么`A`或`T`至少有一个是在当前作用域中定义的。**

例如上面代码

+ 可为`Rectangle`类型实现标准库中的`Display`特征。这是由于`Rectangle`类型定义在当前的作用域中。
+ 假设当前代码段中存在`String`类型，即可为其实现`Geometry`特征，这是由于`Geometry`定义在当前作用域中。
+ 无法在当前作用域中，为`String`类型实现`Display`特征，因为它们俩都定义在标准库中，其定义所在的位置都不在当前作用域。

该规则被称为**孤儿规则**，可确保其它人编写的代码不会破坏你的代码，也确保了不会莫名其妙就破坏了的其它代码。

###### 默认实现

在特征中定义具有**默认实现**的方法，这样其它类型无需再实现该方法，或者也可以选择重载该方法。

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("Read more...")
    }
}

pub struct Post {
    pub title: String,
    pub author: String,
    pub content: String,
}

pub struct Weibo {
    pub username: String,
    pub content: String,
}

impl Summary for Post {}

impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}

let post = Post{ title: "post".to_string(), author: "jack".to_string(), content: "rust".to_string() };
println!("{}", post.summarize());

let weibo = Weibo { username: "jack".to_string(), content: "rust".to_string() };
println!("{}", weibo.summarize());
```

```shell
Read more...
jack发表了微博rust
```

`Post`选择了默认实现，而`Weibo`重载了该方法。

默认实现**中**允许调用该特征中的其他方法，无论这些方法是否默认实现。特征可以提供很多有用的功能而只需实现指定的部分内容。

如：定义`Summary`特征，使其具有一个待实现的`summarize_author`方法，然后定义一个默认实现的`summarize`方法，并在此方法中调用`summarize_author`方法。

为了使用`Summary`，只需实现`summarize_author`方法即可：

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        format!("Read more from {}...", self.summarize_author())
    }
    fn summarize_author(&self) -> String;
}

pub struct Post {
    pub title: String,
    pub author: String,
    pub content: String,
}

impl Summary for Post {
    fn summarize_author(&self) -> String {
        format!("@{}", self.author)
    }
}

let post = Post{ title: "post".to_string(), author: "jack".to_string(), content: "rust".to_string() };
println!("{}", post.summarize());
```

```shell
Read more from @jack...
```

`post.summarize()`会先调用 `Summary` 特征默认实现的 `summarize` 方法，通过该方法进而调用 `Post` 为 `Summary` 实现的 `summarize_author` 方法。

###### 使用特征作为函数参数

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;
}

pub struct Post {
    pub author: String,
    pub content: String,
}

impl Summary for Post {
    fn summarize_author(&self) -> String {
        format!("@{}", self.author)
    }
}

pub fn notify(item: &impl Summary) {
    println!("Breaking news ! {}", item.summarize_author());
}

let post = Post{ author: "jack".to_string(), content: "rust".to_string() };
notify(&post);
```

```shell
Breaking news ! @jack
```

可使用任何**实现`Summary`特征**的类型作为该函数的参数，同时在函数体内，还可调用该特征的方法。

###### 特征约束（trait bound)

`impl Trait`实际上只是一个语法糖：

完整的书写形式：

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

`T: Summary`被称为**特征约束**。

在简单的场景下`impl Trait`足够使用，但对于复杂的场景，特征约束可以拥有更大的灵活性和语法表现能力。

如：一个函数接受两个`impl Trait`的参数

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;
}

pub struct Post {
    pub author: String,
    pub content: String,
}

pub struct Weibo {
    pub username: String,
    pub content: String,
}

impl Summary for Post {
    fn summarize_author(&self) -> String {
        format!("@{}-post", self.author)
    }
}

impl Summary for Weibo {
    fn summarize_author(&self) -> String {
        format!("@{}-weibo", self.username)
    }
}

pub fn notify(item1: &impl Summary, item2: &impl Summary) {
    println!("Breaking news ! {}", item1.summarize_author());

    println!("Breaking news ! {}", item2.summarize_author());
}

let post = Post{ author: "jack".to_string(), content: "rust".to_string() };
let weibo = Weibo { username: "nick".to_string(), content: "rust".to_string() };
notify(&post, &weibo);
```

```shell
Breaking news ! @jack-post
Breaking news ! @nick-weibo
```

如函数两个参数是不同的类型，只要这两个类型都实现了`Summary`特征即可使用上面方法。

**强制函数的两个参数是同一类型，只能使用特征约束来实现**

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {}
```

泛型类型 `T` 说明了 `item1` 和 `item2` 必须是同样的类型，同时 `T: Summary` 说明了 `T` 必须实现 `Summary` 特征。

###### 多重约束

除了单个约束条件，还可指定多个约束条件。

+ 语法糖形式：

  ```rust
  pub fn notify(item: &(Summary + Display))
  ```

+ 特征约束模式

  ```rust
  pub fn notify<T: Summary + Display>(item: &T) {}
  ```

###### `where`约束

当有多个约束时，函数的签名将变的复杂

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}
```

可通过`where`对其形式做一些改进

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
	where T: Display + Clone,
		  U: Clone + Debug
{}
```

###### 使用特征约束有条件地实现方法或特征

特征约束，可在指定类型 + 指定特定地条件下实现方法

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("the largest num is x = {}", self.x);
        } else {
            println!("the largest num is y = {}", self.y);
        }
    }
}

let p = Pair::new(4, 8);
p.cmp_display();
```

```shell
the largest num is y = 8
```

`cmp_display`方法，要`T`同时实现了`Display + PartialOrd`的`Pair<T>`才拥有此方法。该形式可读性好。每个泛型参数的特征也通过特征约束进行约束。

**也可有条件地实现特征**。

如：标准库为任何实现了`Display`特征的类型实现了`ToString`特征

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

这样可对任何实现了 `Display` 特征的类型调用由 `ToString` 定义的 `to_string` 方法。

如：可以将整型转换为对应的 `String` 值，因为整型实现了 `Display`

```rust
let s = 3.to_string();
```

###### 函数返回中的`impl Trait`

可通过`impl Trait`来说明一个函数返回一个类型，该类型实现了某个特性

```rust
pub struct Weibo {
    username: String,
    content: String,
}

pub trait Summary {
    fn summary_author(&self) -> String;
}

impl Summary for Weibo {
    fn summary_author(&self) -> String {
        format!("{}", self.username)
    }
}

fn return_summarizable() -> impl Summary {
    Weibo {
        username: String::from("jack"),
        content: String::from("rust"),
    }
}

println!("{}", return_summarizable().summary_author());
```

```shell
jack
```

因为`Weibo`实现了`Summary`，所以它可用来作为返回值。对于`returns_summarizable` 的调用者而言，只知道返回的是一个`Summary`特征的对象，而不知返回了一个`Weibo`类型。

这种 `impl Trait` 形式的返回值，在一种场景下非常有用，那就是返回的真实类型非常复杂，不知道该怎么声明时，就可以用 `impl Trait` 的方式简单返回。

如：闭包和迭代器就是很复杂，只有编译器才知道真实类型，就可以用 `impl Iterator` 来告诉调用者，返回了一个迭代器，因为所有迭代器都会实现 `Iterator` 特征。

这种返回值方式**只能有一个具体的类型**，这是一个很大的限制。

如：

```rust
fn return_summarizable(switch: bool) -> impl Summary {
    if switch {
        Weibo {
            username: String::from("jack"), 
            content: String::from("rust"),
        }
    } else {
        Post {
            author: String::from("nick"),
            content: String::from("rusr"),
        }
    }
}
```

以上的代码是无法通过编译，因为它返回的是两个不同的类型`Post`和`Weibo`。

```shel
error[E0308]: `if` and `else` have incompatible types
  --> src/main.rs:34:13
   |
28 |   /         if switch {
29 |   |             Weibo {
   |  _|_____________-
30 | | |                 username: String::from("jack"),
31 | | |                 content: String::from("rust"),
32 | | |             }
   | |_|_____________- expected because of this
33 |   |         } else {
34 | / |             Post {
35 | | |                 author: String::from("nick"),
36 | | |                 content: String::from("rusr"),
37 | | |             }
   | |_|_____________^ expected struct `Weibo`, found struct `Post`
38 |   |         }
   |   |_________- `if` and `else` have incompatible types
```

如要实现返回不同的类型，需要使用**特征对象**

```rust
fn return_summarizable(switch: bool) -> Box<dyn Summary> {
    if switch {
        Box::new(
            Weibo {
                username: String::from("jack"), 
                content: String::from("rust"),
            }
        )
    } else {
        Box::new(
            Post {
                author: String::from("nick"),
                content: String::from("rusr"),
            }
        )
    }
}
```

###### 通过`derive`派生特性

形如：`#[derive(Debug)]`这种是一种特征派生语法，被`derive`标记的对象会自动实现对应的默认特征代码，继承相应的功能。

如：`Debug` 特征，它有一套自动实现的默认代码，当你给一个结构体标记后，就可使用 `println!("{:?}", s)` 的形式打印该结构体的对象。

总之，`derive` 派生出来的是 rust 默认提供的特征，在开发过程中简化了手动实现相应特征的需求，如有特殊的需求，还可以手动重载该实现。

[详细的`derive`列表]([派生特征 trait - Rust语言圣经(Rust Course)](https://course.rs/appendix/derive.html))

###### 调用方法需引入特征

在一些场景中，使用 `as` 关键字做类型转换会有比较大的限制，因为想要在类型转换上拥有完全的控制。例如：处理转换错误，那么需要 `TryInto`

```rust
// use std::convert::TryInto;

let a: i32 = 10;
let b: u16 = 100;

let b_ = b.try_into().unwrap();

if a < b_ {
    println!("a < b");
}
```

```shell
a < b
```

**要使用一个特征的方法，需要引入该特征到当前的作用域中**，用到了 `try_into` 方法，因此需要引入`std::convert::TryInto`的特征，但如果此时进行编译，编译器会警告`std::convert::TryInto`的引入是多余的。

这是rust把最常用的标准库中的特征通过 `std::prelude` 模块提前引入到当前作用域中，其中包括了 `std::convert::TryInto`。

###### 总结例子

+ **为自定义类型实现`+`操作**

  在rust中除了数值类型的加法，`String`也可做加法，这是由于rust为这些类型实现了`std::ops::Add`类型。

  同理，也可为自定义类型实现该特征，即可实现`Point1` + `Point`的操作。

  ```rust
  use std::ops::Add;
  
  // 为Point结构体派生Debug特征，用于格式化输出
  #[derive(Debug)]
  struct Point<T: Add<T, Output = T>> {
      // 限制类型T必须实现Add特征，否则无法进行+操作
      x: T,
      y: T,
  }
  
  // 为Point<T>实现Add特征
  impl<T: Add<T, Output = T>> Add for Point<T> {
      // 关联类型
      type Output = Point<T>;
  
      fn add(self, p: Point<T>) -> Point<T> {
          Point {
              x: self.x + p.x,
              y: self.y + p.y,
          }
      }
  }
  
  fn add<T: Add<T, Output = T>>(a: T, b: T) -> T {
      a + b
  }
  
  let p1 = Point { x: 1.1f32, y: 2.2f32 };
  let p2 = Point { x: 2.3f32, y: 3.5f32 };
  println!("{:?}", add(p1, p2));
  ```

  ```shell
  Point { x: 3.4, y: 5.7 }
  ```

+ **自定义类型的打印输出**

  使用`#derive(Debug)`对自定义类型进行标注，即可实现打印输出的功能。

  ```rust
  #[derive(Debug)]
  struct Point {
      x: i32,
      y: i32,
  }
  
  let p = Point { x: 2, y: 3 };
  println!("{:?}", p);
  ```

  ```shell
  Point { x: 2, y: 3 }
  ```

  在实际开发中，需对自定义类型进行自定义的格式输出，就需为自定义类型实现`std::fmt::Display`特征。

  ```rust
  #![allow(dead_code)]
  
  use std::fmt;
  use std::fmt::{ Display };
  
  #[derive(Debug, PartialEq)]
  enum FileState {
      Open,
      Closed,
  }
  
  #[derive(Debug)]
  struct File {
      name: String,
      data: Vec<u8>,
      state: FileState,
  }
  
  // 为FileState实现Display特性 并重载fmt
  impl Display for FileState {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          match *self {
              FileState::Open => write!(f, "OPEN"),
              FileState::Closed => write!(f, "CLOSED"),
          }
      }
  }
  
  // 为File实现Display特性 并重载fmt
  impl Display for File {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          write!(f, "<{} ({})>", self.name, self.state)
      }
  }
  
  impl File {
      fn new(name: &str) -> File {
          File {
              name: String::from(name),
              data: Vec::new(),
              state: FileState::Closed,
          }
      }
  }
  
  let f = File::new("f.txt");
  println!("{:?}", f);
  println!("{}", f);
  ```

  ```shell
  File { name: "f.txt", data: [], state: Closed }
  <f.txt (CLOSED)>
  ```

##### 特征对象

###### 特征对象定义

假设编写一个UI库时，只知道：

+ UI对象的类型不同
+ 需要一个统一的类型来处理这些对象，无论是作为函数参数还是列表中的一员
+ 需对每一个对象调用`draw`方法

在拥有继承的语言中，可定义一个`Component`的类，该类有一个`draw`方法。其他的类如：`Button`、`SelectBox`会从`Component`派生并因此继承`draw`方法。它们可重载`draw`来定义自己的行为，框架会把这些类型当作是 `Component` 的实例，并在其上调用 `draw`。

**rust没有继承**，需另寻出路，由此引入了一个概念——**特征对象**。

**特征对象**是对具有相同行为的**一组**具体类型的抽象，将共同拥有相同行为的类型集合**抽象为一个类型**。

**特征对象的创建**

+ 通过 **`&` 引用**方式来创建

  **`&dyn Trait`**形式的特征对象

+ 通过**`Box<T>` 智能指针**方式来创建

  **`Box<dyn Trait>`**形式的特征对象

  由于`T`都实现了`Trait`特征，`Box<T>`在调用时隐式地被转换为特征对象`Box<dyn Trait>`

+ 注意：由于'dyn Trait'是**动态大小**类型(无法在编译期确定大小，在运行时才能确定大小)，像切片一样，所以总是使用它的**引用**

+ `dyn`关键字只能用在特征对象的类型声明上，在创建时无需使用 `dyn`

**创建一个UI库**

+ 为UI组件定义一个特征

  ```rust
  // 为UI组件定一个特征
  pub trait Draw {
      fn draw(&self);
  }
  ```

+ 定义UI组件并为UI组件实现`Draw`特征，才可调用`draw`进行渲染。

  这以`Button`和`selectBox`为例 

  ```rust
  // 为Button 和 selectBox 组件实现draw特征
  pub struct Button {
      pub width: u32,
      pub height: u32,
      pub label: String,
  }
  
  impl Draw for Button {
      fn draw(&self) {
          // 绘制Button的代码
          println!("draw Button w: {}, h: {}", self.width, self.height);
      }
  }
  
  pub struct SelectBox {
      pub width: u32,
      pub height: u32,
      pub options: Vec<String>,
  }
  
  impl Draw for SelectBox {
      fn draw(&self) {
          // 绘制SelectBox的代码
          println!("draw SelectBox w: {}, h: {}", self.width, self.height);
      }
  }
  ```

+ 定义一个UI库，并定义将列表中的UI组件渲染在在屏幕上的方法

  ```rust
  // 声明一个UI库
  pub struct Screen {
      // 使用一个动态数组来储存这些UI对象
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

  `Vec<Box<dyn Draw>>`:

  + 由于`Button`和`SelectBox`都实现了`Draw`特征，所以可将`Button`和`SelectBox`抽象为一个类型，填入到数组中。
  + 特征对象指向实现了 `Draw` 特征的类型的实例，也就是`Button`和`SelectBox`，这种映射关系是存储在一张表中，可以在运行时通过特征对象找到具体调用的类型方法。
  + `Box<dyn Draw>`形式的特征对象是通过 `Box::new(x)` 的方式创建的。

  上边代码也可**通过泛型来实现**

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

  在 `Screen` 中使用特征约束让 `T` 实现了 `Draw` 特征，进而可以调用 `draw` 方法。

  **注意**：这种写法限制了 `Screen` 实例的 `Vec<T>` 中的每个元素必须是**相同**类型（ `Button` 或 `SelectBox` ）。

  如只需同质（相同类型）集合，更倾向于这种写法：使用泛型和特征约束。实现更清晰，且性能更好。

  这是由于特征对象，需要在运行时从 `vtable` 表中查找需要调用的方法。

+ 运行渲染UI 组件列表

  + 使用特征对象

    ```rust
    // 运行渲染UI组件库
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };
    
    screen.run();
    ```

    ```shell
    draw SelectBox w: 75, h: 10
    draw Button w: 50, h: 10
    ```

  + 使用泛型和特征约束

    ```rust
    let screen = Screen {
        components: vec![
            Button {
                width: 75,
                height: 10,
                label: String::from("OK"),
            },
            Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            },
        ],
    };
    screen.run();
    ```

    ```shell
    draw Button w: 75, h: 10
    draw Button w: 50, h: 10
    ```

**另一个例子**

```rust
pub trait Draw {
    fn draw(&self) -> String;
}

impl Draw for u8 {
    fn draw(&self) -> String {
        format!("u8: {}", *self)
    }
}

impl Draw for f64 {
    fn draw(&self) -> String {
        format!("f64: {}", *self)
    }
}

// 若T实现了Draw特征，则调用该函数时传入的Box<T> 可被隐式转换成函参数签名中的Box<dyn Draw>
fn draw1(x: Box<dyn Draw>) {
    // 由于实现了Deref特征，Box智能指针会自动解引用为它所包裹的值,然后调用该值对应的类型上定义的draw方法
    println!("{}", x.draw());
}

fn draw2(x: &dyn Draw) {
    println!("{}", x.draw());
}

let x = 1.1f64;
let y = 8u8;

// x,y 的类型T都实现了Draw特征
// 因为 Box<T> 可以在函数调用时隐式地被转换为特征对象 Box<dyn Draw>

// 基于x的值创建一个Box<f64>类型的智能指针
draw1(Box::new(x));
// 基于y的值创建一个Box<u8>类型的智能指针
draw1(Box::new(y));

draw2(&x);
draw2(&y);
```

```shell
f64: 1.1
u8: 8
f64: 1.1
u8: 8
```

这段代码几个重点：

+ `draw1`函数的参数是`Box<dyn Draw>`形式的特征对象，该特征对象是通过`Box::new(x)`的方式创建。
+ `draw2`函数的参数是`&dyn Draw`形式的特征对象，该特征对象是通过`&x`的方式创建。
+ 可以使用特征对象来代表泛型或具体的类型。

###### 鸭子类型(duck typing)

鸭子类型：指只关心值，不关心它实际是什么。例如：当一个东西走起来像鸭子，叫起来像鸭子，那么它就是一只鸭子，就算它实际上不是一只鸭子，也不重要，就当它是鸭子。

在上例中，`Screen`在`run`时，并不需知道各个组件的具体类型是什么，也不检查是`Button`还是`SelectBox`的实例，只要实现了`draw`特征，就能通过 `Box::new` 包装成 `Box<dyn Draw>` 特征对象，然后被渲染。

使用特征对象和rust类型系统来进行类似鸭子类型操作的优势是：无需在**运行时**检查一个值是否实现了特定方法或者担心在调用时因为值没有实现方法而产生错误。如果值没有实现特征对象所需的特征， 那么 rust根本就不会编译这些代码。

```rust
pub trait Draw {
    fn draw(&self) -> String;
}

fn draw1(x: Box<dyn Draw>) {
    println!("{}", x.draw());
}

let z = 5u32;
draw1(Box::new(z));
```

```shell
error[E0277]: the trait bound `u32: Draw` is not satisfied
  --> src/main.rs:43:11
   |
43 |     draw1(Box::new(z));
   |     ----- ^^^^^^^^^^^ the trait `Draw` is not implemented for `u32`
   |     |
   |     required by a bound introduced by this call
   |
   = help: the following other types implement trait `Draw`:
             f64
             u8
   = note: required for the cast to the object type `dyn Draw`
```

由于`u32`没有实现`Draw`特征，rust编译报错，不会让上述代码运行。

**注意 `dyn` 不能单独作为特征对象的定义**

```rust
fn draw3(x: dyn Draw) {
    x.draw();
}
```

```shell
error[E0277]: the size for values of type `(dyn Draw + 'static)` cannot be known at compilation time
  --> src/main.rs:42:14
   |
42 |     fn draw3(x: dyn Draw) {
   |              ^ doesn't have a size known at compile-time
   |
   = help: the trait `Sized` is not implemented for `(dyn Draw + 'static)`
help: function arguments must have a statically known size, borrowed types always have a known size
```

编译器报错的原因是特征对象可以是任意实现了某个特征的类型，编译器在编译期不知道该类型的大小，不同的类型大小是不同的。

而 `&dyn Trait` 和 `Box<dyn Trait>` 在编译期都是**已知大小**，所以可用作特征对象的定义。

###### 特征对象在内存中的表现

![triat](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/4RDSRXhpZgAATU0AKgAAAAgABAE7AAIAAAAEbGluAIdpAAQAAAABAAAISpydAAEAAAAIAAAQwuocAAcAAAgMAAAAPgAAAAAc6gAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFkAMAAgAAABQAABCYkAQAAgAAABQAABCskpEAAgAAAAM0OAAAkpIAAgAAAAM0OAAA6hwABwAACAwAAAiMAAAAABzqAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAyMjowODoxOSAwMjozMzo1NQAyMDIyOjA4OjE5IDAyOjMzOjU1AAAAbABpAG4AAAD/4QsWaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49J++7vycgaWQ9J1c1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCc/Pg0KPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyI+PHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj48cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0idXVpZDpmYWY1YmRkNS1iYTNkLTExZGEtYWQzMS1kMzNkNzUxODJmMWIiIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIvPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIj48eG1wOkNyZWF0ZURhdGU+MjAyMi0wOC0xOVQwMjozMzo1NS40ODI8L3htcDpDcmVhdGVEYXRlPjwvcmRmOkRlc2NyaXB0aW9uPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6ZGM9Imh0dHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIj48ZGM6Y3JlYXRvcj48cmRmOlNlcSB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPjxyZGY6bGk+bGluPC9yZGY6bGk+PC9yZGY6U2VxPg0KCQkJPC9kYzpjcmVhdG9yPjwvcmRmOkRlc2NyaXB0aW9uPjwvcmRmOlJERj48L3g6eG1wbWV0YT4NCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgPD94cGFja2V0IGVuZD0ndyc/Pv/bAEMABwUFBgUEBwYFBggHBwgKEQsKCQkKFQ8QDBEYFRoZGBUYFxseJyEbHSUdFxgiLiIlKCkrLCsaIC8zLyoyJyorKv/bAEMBBwgICgkKFAsLFCocGBwqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKioqKv/AABEIAc4D5AMBIgACEQEDEQH/xAAfAAABBQEBAQEBAQAAAAAAAAAAAQIDBAUGBwgJCgv/xAC1EAACAQMDAgQDBQUEBAAAAX0BAgMABBEFEiExQQYTUWEHInEUMoGRoQgjQrHBFVLR8CQzYnKCCQoWFxgZGiUmJygpKjQ1Njc4OTpDREVGR0hJSlNUVVZXWFlaY2RlZmdoaWpzdHV2d3h5eoOEhYaHiImKkpOUlZaXmJmaoqOkpaanqKmqsrO0tba3uLm6wsPExcbHyMnK0tPU1dbX2Nna4eLj5OXm5+jp6vHy8/T19vf4+fr/xAAfAQADAQEBAQEBAQEBAAAAAAAAAQIDBAUGBwgJCgv/xAC1EQACAQIEBAMEBwUEBAABAncAAQIDEQQFITEGEkFRB2FxEyIygQgUQpGhscEJIzNS8BVictEKFiQ04SXxFxgZGiYnKCkqNTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqCg4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2dri4+Tl5ufo6ery8/T19vf4+fr/2gAMAwEAAhEDEQA/APpGiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigDzjU4/FkvxGsll1Ox0+3eCbyjBCXOwHPzbzjPvitkeKtTm1e4sdH02LU4bBUF1dPdCNnYjPyKFIPHqQK0/EHhWz8RTW0tzPcwSW+4BreUoWVhhlJHY1PpPh3TNEe4bToPKNxt8wbiQdowOD7UAYTfEfT4bi0+1Ws1tZ3k/2eC6lZQHf2XOcZ4zVLxTLrEi67a+HtTV3FvvuIr0ErAhQ4MW3Byfc4rbi8AeHIjcH+zw4nBUiSRnCAnJ2An5efTFXrXw1ptokqwxOTNALd2klZ2ZAMAEk0AZngVNRTw/ZnVdXW9ke2RliEapsGOvHJ+prqKwvD3hKx8OM7201zcSsojElzKXKoOijsAK3aACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAoqmmsabLq76VHf2z6jHF5z2izKZVjyBuKZyBkgZ96fqLXiaZcNpkcct4IyYUkbarPjgE+lAFmiuD0jVvE2n+MrPSNfv7O/wDtdm9zMlvb+X9kIIwM7juB59OldXH4g0qTTY9QW+h+ySv5aTFvlZs4wD9aANGiub0jx1pGvarJZaOt1diJmSS5jt2MKMOql8YzTF+IOg/21FpNzJc2d7O5SKK6tni8wj+6SOaAOnormx4mj0fSZrvxLe2Z23LQxmz3OX54XbjO/wBhmtTRNcsPEOmrf6VMZYGYrypUqQcEEHkGgDQooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACijNNkkSJC0rqijqWOBQA6imRzxTQiWKRHjPR1bIP405XV1yjBh6g5oAWimSzRwoXmkWNR1LHArl/F3xD0PwjZxSXd7bPPMybLfzgHZCcFg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NvDrFRuH+hynf/s/f5+lb3wkW4XwGn210ef7VN5hjBC53nOAelai8gYI5+Yc8Ef3+vDe1UPhb/yJhyQx+2T87cH756+9ell8ryZzYhWSOyooor2DiCiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooA4GTSPHLeIdQigvbWPRryZv3jOTNGhxynoccY/GmaV4CubDw/Hs41Rp/9IZ52ZZod/Kt6/L0r0GigDzmTQ9a0GbXk8N6PaxRz7poLlCA6/IBtQY+9nPXiq/gq8tfD7SrF/bmrXN2AZV+wlBGwHOc7QWPrXp1FAHI6v4k1yazNvonhrUkuLgFEnuPLRYCRwxG49KybDQ/GcUN5HdyRy219avCbaW6MjQvtI37yOdxPIwMV6JRQBwj+E9VVpL238pLy2eGa1BfKuViCMjegPPNJrNtqmqx3mr3un/2dFb6PcRlWlDPIzJkggdAMfrXeVl+Jv+RS1f8A68Zv/RZoA53w+zL4Z08s2cW0aM2Onyj5D/jWmW2gg5XaNpweU/2BzyPes3w++PDunknbttkG4r/qxjow7k+taRIHrF5Y+phB/nmvk5P3nr/X9f1vy+ulov6/r+vmuBjBHTqqHp7J7etAXORgPvPROkh9F44x3pDhcj7u3kqjfcHqnPU9xRgFTnHzDkL0I9B6N60v6/r+v1QByWJPz7ztJxgSn+6fTHrRuy2Sd3O0lv4j/dYZ+6PWgHjJ7/KTjr/sH396Nw5LHG35Ce6/7HuPej+v6/r9W2BIJA5OD9Tn190pdvGFG7PzADjcf7446D0pp25O7+HghTkr7L6r60EAkjAbJ5VOjH/YPoO9H9f1/X+QC4OBgq+87hngSn+97EelUPhcc+Djl95+2T5OMH7561fY59JPMODjgTH09sVQ+Fx/4o4gvvIvJ8+o+c8V6eXfG/6/r+vV8uJ+Ff1/X9ei7KiiivaOEKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKCcdaR22IzEEhRnAGTXjieJ7jU/DerSQa+80159rWWynHlyWwV2VDHwCMAKCD3Oc0AeyUVnR65poszM15GI45vszOW/5ag4K/XNYq+P9Nu9Ma/0uOee1S4hha4eJkjYSSBCysRhsZzQB1dZXilQ/g/WVYZDWE4I/wC2bVlnxHDrXifTLHRdTja0Mc080sDBvNMbKvlZ+rZPfgVUu768hs/Fmi6lM1w0NjJdWszAAtDIjjacd1ZWH0xQByGifC3whc6JYzz6WXlaFWcrPJ8xI6Dnk1dHwl8G/J/xKSdrcEXEnzH+797rXQ+Hhnwzp3AU/ZkOAeAMDkf7VaQGBnpxjJ9PQ/7VfLyqT5nqeqoxtscZ/wAKl8Ggj/iVcKf+e8nDH+H7360H4TeDRkHST6HE8nB/765Hv1rswxGS2V2jaT/cH90/X1ozgnOV2jBxyUHoPWp9pLu/6/r+tbPlXY4w/CXwZ0/sgsduCBcSfN7j5uR79acPhL4N+QjSd5AIX/SJP3n0+btXYkAnaR9VTt7J/Whhk4xu3cEL0b2HoaftJ9/6/r+ujXLHscYvwl8GbV26Xuxwh8+Qbj/31xSD4S+DMH/iVHngEzyDB9/m4HuK7Q5wcnduO0+jH+77fWgnAOck5wT7/wB0/wCz70vaS7v+v6/rdvlXY4w/CbwYS/8AxKSNy4P7+Tj/AGuvSt74RQQWvgGO3s02wxXU6IOegc+vNaw/PnoPX29VrP8Aha27waTjH+mT/j85r0svlJyd2cuISSVjsqKKK9g4wooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigArHPhmx/sO80lTKtvdyySvhvmVnfecHHqa2KKAOZk8CaZJqS3Zub5RHdi9jgWYCJJs5Lhcd+c5JHJxisqT4YQ3d5N/aOqSS2ExzJawwiLzfnDjzCDhuR1Cgn1ru6KAOZuvAumssz6a8mnXTTi5hngx+4kCBDtHTaQoyvQ03VdDjsNG8RalLczXV3dWEqs8pGI0VGIRQBwMknuea6is7xDFJP4Y1SKFC8klnMqKo5YlCAKAOb8Pr/AMU1pwGObdCATwePvexrSJwBg/iw/Vvf0rjtJ8Vw2ejWttPperGSKJVdfsTYdgP6Vebxra7sf2Zq7E9C1i3zn/a+lfMSo1Lu0X/X9f119RTjbc6PO0Zzt2jgsMlB/teuaFIHqmwZ9TGPX3zXOHxtZgZGmawccqTYsTn39RQvjWzKqV0zWBzkA2LcH+9/9aj2NX+X+v6/raxzw7/1/X9d+jC/8BwM4B+6PVf9o+lJjnsOMe2PT2audHjS0A40zV+vT7C//fX19qanji0dnA0rWMqcfNYMA3+19aXsKn8rH7SPc6UnB54/hJ9B/dPv70rHAOcrtG046oP7vuPeubHja03caZq4OPlJsWOB7+ppB43sy2F0zWE28A/YW+Qf1zT9jV7f1/X9dlzx7/1/X9d+kKgZ4xgYIX+H2X1HrWf8LN3/AAhh3qqn7ZP908H5zWWfGlngf8SzWFA4wLJvkHqPc1tfDO2uLbwZH9qtpLZpLiWVUlXDbWckEjtxXoYCnOE3zK39f1/Whz4iSaVmdbRRRXrnGFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFQX1ubzT7i2WRojNEyB0OGXIxkH1oAnrJ0jxPpGuB/wCz7yNnSeWAxMwD7o2Kt8uc4yprhtOg1bxnp2neHNXhvbS10sBNXncmM3ciZVURupU8OT9BVPwS2j+F7rUtJ0nRmn1iPUL0hVjO6OMbnQsx7NkAc85oA9brM0bW49Zk1FYomjFjePaEk53lQMkenWvNdM8X+KIr6z1VoNQv9MVf+JwJLby/IZjwIUxubYevqPWq1pr3iDw7qGq2+nI89xday01vYNYufPhlYHeZei/LnvxjkUAel+HNTutSk1cXe0i11GW3iIXHyKBj+Zrbrya+1vV9L0V7PTB9gvdW167iaec+WIwGJADEYBZQAprr/Af9p2+lz2ev6lFd3qzM6RfaRNJDEfuh2GMnrzigDqqKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKYsUaSM6xqrt95gvJ+pp9FABRRRQBWv9Ns9Us2tdRtormB/vRyLkGq+k6BpWhRsmkWENoG+95a4J+prRooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooA//Z)

在内存中，特征对象是一个**胖指针**，它是由两个普通指针(data pointer 和 vtable pointer)组成，每个特征对象占用两个usize。

+ data pointer：指向值的指针（Vec）
+ vtable pointer：
  + 指向`vtable`(虚表)的指针，这张表能够表示值的类型。
  + `vtable`在**编译时**生成，并由相同类型的**所有对象共享**(`w`于`v`共享)。
  + `vtable`包含了一系列函数指针，指向实现trait特征(Write)必须实现的方法，当调用一个特征对象的方法时，rust会自动使用vtable

> 关于胖指针：一个指向大小不固定的值的指针总是一个胖指针，占用**两**个字节。如：
>
> + str胖指针：指向切片的指针 + 切片长度
> + 特征对象：一个`trait`对象 + 一个指向`trait`方法实现的`vtable`指针

对特征类型的引用称为特征对象。

有多种方法可将具体类型转为实现了`Trait`特征的对象：

+ rust将普通引用装换为实现了`Trait`特征的特征对象

  + 在给变量赋值时发生转换

    ```rust
    use std::io::Write;
    
    let mut buffer::Vec<u>  = vec![];
    let w: &mut dyn Write = &mut buffer;
    ```

  + 在将某种具体类型作为参数传递给接受实现了`Trait`特征的特征对象的函数时发生转换

    ```rust
    use std::io::Write;
    
    let mut buffer::Vec<u8> = vec![];
    writer(&mut buffer);
    
    fn writer(w: &mut dyn Write) {
        // code...
    }
    ```

+ rust可将智能指针(如 Box 和 Rc)装换为实现了`Trait`特征的特征对象

  + `Box`

    ```rust
    let mut buffer: Vec<u8> = vec![];
    let mut w: Box<dyn Write> = Box::new(buffer);
    ```

    `Box<dyn Write>`意味着拥有一个在堆上实现`Write`特征的值

  + `Rc`

    ```rust
    use std::io::Write;
    use std::rc::Rc;
    
    let mut buffer: Vec<u8> = vec![];
    let mut w: Rc<dyn Write> = Rc::new(buffer);
    ```

​	在这些情况下，它也会转为胖指针。无论它是普通引用还是智能指针，在发生转换时，rust知道引用的真实类型，在上面的例子中就是`Vec<u8>`，它只是**通过添加适当的`vtable`地址，将常规指针转化为胖指针**。

###### `Self` 与 `self`

+ `Self`：指代特征或者方法类型的别名
+ `self`：指代当前的实例对象

```rust
trait Draw {
	fn draw(&self) -> Self;
}

#[derive(Clone)]
struct Button;
impl Draw for Button {
    fn draw(&self) -> Self {
        return self.clone()
    }
}

let button = Button;
let btn = button.draw();
```

+ `self`指代的就是当前的实例对象，也就是 `button.draw()` 中的 `button` 实例
+ `Self` 则指代的是 `Button` 类型

###### trait 同时支持静态分发与动态分发

+ 

###### 动态大小类型(DST)与`Sized trait`

+ **动态大小类型(DST: Dynamic Sized Type | unsized types)**

  **类型的大小从确定的时间分类**：

  + 定长类型(`sized`)：类型大小在编译时已知。
  + 不定长类型(`unsized`)：类型大小只有到程序运行时才能动态获知，又称为**DST**。

  **关于动态大小类型(DST)**

  rust中几乎所有类型都是固定大小的类型，如：`Vec`、`String`等，而动态大小类型与之相反：**在编译期是无法得知类型值的大小，只有到了程序运行时，才能动态获知。**

  > 像`Vec`这些类型虽然底层数据可动态变化，像是DST，但实际上，这些底层数据是保存在堆上，在栈中存着的是一个引用类型，该引用可能包含了集合的内存地址、内容数目、分配空间信息，通过这些信息，编译器对该类型值实际大小了如指掌，重要的是：栈上的引用类型是固定大小的，所以这些类型都是**固定大小类型**。

  如在代码中直接使用DST类型，编译器无法在编译器获知类型大小，会报错。

  **最典型动态大小类型：str**

  ```rust
  let s: str = "abc";
  ```

  ```shell
  error[E0277]: the size for values of type `str` cannot be known at compilation time
   --> src/main.rs:2:9
    |
  2 |     let s: str = "abc";
    |         ^ doesn't have a size known at compile-time
    |
    = help: the trait `Sized` is not implemented for `str`
    = note: all local variables must have a statically known size
    = help: unsized locals are gated as an unstable feature
  ```

  编译器不能在编译期`s`的具体大小，所以报错。

  **使用动态大小类型的黄金规则（只能间接使用）**

  + 通过引用，将动态大小类型的值置于`&`之后

    如：`&str`存储在栈上，包含着`str`的地址和长度，大小为 2 * usize。`&str`的大小在编译器已确定。

  + 必须将动态大小类型的值置于某种指针之后，如：`Box<DST>\RC<DST> `
  + 总结：将动态大小固定化就是**使用引用指向动态大小类型的值，并在引用中存储相关的内存位置、长度等信息**

  **rust中常见的DST类型还有**：`trait`，同样trait 对象必须通过`&dyn Trait`或`dyn Box<Trait> | Rc<Trait>`来间接使用，无法单独被使用。

+ **`Sized trait`**

  为了处理DST，rust用`Sized trait`来决定一个类型的大小是否在编译时可知。

  `Sized trait`自动为编译器在编译时就已知该类型的大小和实现。

  + 在使用泛型时，rust如何确保泛型参数时固定大小的类型的

    ```rust
    fn generic<T>(t: T) {
        // --snip--
    }
    
    // 等价于
    fn generic<T: Sized>(t: T) {
        // --snip--
    }
    ```

    rust自动添加的特征约束`T: Sized`，即`T`只能是实现了`Sized Trait`的类型，而**所有在编译时就已知其大小的类型，都自动实现`Sized trait`**

  + 在使用泛型时，如要使用DST，可使用`?Sized Trait`来放宽这个限制

    ```rust
    fn generic<T: ?Sized>(t: &T) {
        / --snip--
    }
    ```

    **注意：**

    + `T: ?Sized`表示`T`可能是`Sized`，也可能不是`Sized`的。
    + 函数参数也变成了`&T`，由于`T`可能是DST，因此需用一个固定大小的指针(引用)来包裹它。

  + 除此之外，还有`!Sized`表示类型必须是编译期不确定大小的。

###### 特征对象的限制

rust没有类继承，特征对象体现出来的是对一个trait类型的指针或引用。在运行时才能确定具体类型，编译期间不知道大小。





**不是所有**特征都能拥有特征对象。能作为特征对象使用的`trait`需同时满足以下规则：

1. `trait`的`Self`类型参数不能被限定为`Sized`(特征对象 不能标记为`Sized`)

2. `trait`中所有方法必须是**对象安全**，该规则要求方法必须满足以下条件之一：

   + 方法受 `Self`：`Sized`约束(被`Sized`约束的方法不能被动态调用，)

   `std::marker::Sized`，所有固定大小的类型都具有`Sized`特征，主要用作类型参数的约束。

   所有类型参数都具有`Sized`的隐式界限，可用特殊语法`?Sized`来删除此绑定。

   

   `T: Sized`的约束要求`T`是一个大小在编译期已知的类型。

   对于rust中大小不固定的类型(值的大小不同)，如字符串切片类型`str`、`dyn`类型、`struct`中最后一个字段

3. 

+ 方法没有任何泛型参数

对象安全对于特征对象是必须的，一旦有了特征对象就不需知道实现该特征的具体类型。

+ 如：特征方法返回了具体的`Self`类型







###### 

**静态分发**(static dispatch)：像泛型这样的类型是在编译器完成处理的，编译器会为每一个泛型参数对应的具体类型生成一份代码。由于在编译期完成的，对于运行期性能完全没有任何影响。

**动态分发**(dynamic dispatch)：直到运行时，才能确定需调用的方法。在使用`dyn`关键字时，需注意"动态"这一特点。

当使用特征对象时，必须使用动态分发。编译器无法知晓所有可能用于特征对象的类型以及调用类型的方法实现。动态分发也阻止编译器有选择的内联方法代码，这会相应的禁用一些优化。











































#### 常量求值

[常量求值 - Rust 参考手册 中文版 (rustwiki.org)](https://rustwiki.org/zh-CN/reference/const_eval.html)

#### 所有权

编程语言处理内存空间三个流派

+ 垃圾回收机制(GC)：由GC来管理内存的分配和释放。eg：java、go
+ 手动管理内存的分配和释放：需在编写代码过程中手动申请和释放内存。eg：c++
+ 通过**所有权**来管理内存，编译器会在编译时根据一系列规则进行检查，对于程序运行期，不会有任何性能上的损失。eg：rust

##### 一段不安全代码

```c
int* foo() {
    int a;
    a = 100;
    char *c = "xyz";
    return &a;
}
```

这段代码虽然可以通过编译，但存在两个内存安全问题：

1. 局部变量`a`在函数结束后返回`a`的地址，但`a`存在栈中，当函数执行结束退栈后，栈上的内存会被系统回收，导致**悬空指针**。
2. 字符串`c`储存于常量区，只有当整个程序结束后系统才会回收这片内存。

##### 内存的四个储存区域

###### 代码区

这块区域在程序运行前就存在，是存放代码的区域，存放的代码是CPU执行的机器指令(机器码)。

代码区有两个特点：

1. 共享性

   一个程序可能被运行多次，但只需将代码保存一次，提高程序的运行效率。

2. 只读性

   避免程序运行中被恶意修改代码，该区域具有只读性。

###### 全局区

这块区域在程序运行前就存在。全局区的数据在程序结束后由系统释放。

**全局区存放的数据**：

1. 全局变量
2. 静态变量
3. 常量
   + 字符串常量
   + const修饰的**全局**常量（const修饰的局部常量不存在全局区）

###### 栈区

栈区的数据在程序结束后是由编译器自动释放，所以不能返回局部变量(常量)的地址。

**栈区存放的数据**

1. 函数的参数
2. 局部变量

###### 堆区

堆区的数据由程序员进行释放或在程序运行结束后由编译器自动释放。

一般在堆区开辟空间需要用到`new`关键字。

**堆区存放的数据**

1. 局部变量

##### 栈(Stack)与堆(Heap)

栈和堆的核心目标就是为程序在运行时提供可供使用的内存空间。

###### 栈

增加数据叫做**入栈**，移出数据则叫做**出栈**。遵循**后进先出，先进后出**。

**入栈**

入栈时操作系统时无需分配新空间，只需要将新数据放入栈顶。

栈中储存的数据都必须占用**已知且固定大小**的内存空间。

###### 堆

堆储存的可以是**大小未知或可能变化**的数据。

**在堆上分配内存**

1. 操作系统在堆的某处找到一个足够大的空间，并标记为已使用，返回该位置地址的**指针**
2. 将该指针推入**栈**中（指针的大小是已知固定的）。
3. 后续可通过栈中的指针，获取数据在堆上的位置，进而访问该数据。

###### 栈与堆的性能区别

+ 写入

  入栈比堆上分配内存更快。

  入栈不需分配新的空间，堆上分配需分配新的空间，以及更多的工作。

+ 读取

  栈数据比堆数据读取更快。

  CPU 高速缓存，使得处理器可以减少对内存的访问，高速缓存和内存的访问速度差异在 10 倍以上。

  栈数据往往可以直接储存在CPU高速缓存，堆数据只能存在内存中。访问堆数据需通过访问栈上的指针来访问。

处理器处理和分配在栈上数据会比在堆上的数据更加高效。

##### 所有权

###### 所有权与栈堆

当调用一个函数时，函数的传参(指向堆上数据的指针或函数局部变量)依次压入栈中，当函数调用结束后，这些值将反顺序依次出栈。

由于堆上的数据缺乏组织，因此跟踪这些数据何时分配和释放是非常重要，否则堆上的数据将产生内存泄漏，导致这些数据无法回收。rust所有权系统能提供的强大保障。

###### 所有权原则

> 1. 每一个值都 **有且只有**一个所有者(变量)
> 2. 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)

**变量作用域**

作用域是一个变量在程序中有效范围

```rust
{					 // s 在这里无效，它尚未声明
    let a = "hello"; // s 从创建开始就有效
   	// 使用 s
}					 // 此作用域已结束，s 不再有效
```

###### 变量绑定背后的数据交互

+ 转移所有权

  + 基本数据类型

    ```rust
    let x = 5;
    let y = x;
    println!("x:{}, y:{}", x, y); // x:5 y:5
    ```

    ```
    x:5, y:5
    ```

    将`5`绑定到变量`x`；再拷贝`x`的值赋给`y`，最终`x`和`y`都等于`5`。

    

    对于基本数据类型，值的大小是固定的，是存在栈中。

    对于自动拷贝，栈上的数据简单，而且拷贝快，复制的内存空间小。这种拷贝速度远比在堆上分配空间快。

  + 复杂数据类型

    ```rust
    fn main() {
        let s1 = String::from("hello");
        let s2 = s1;
        println!("s1:{}, s2:{}", s1, s2);
    }
    ```

    报错：禁止使用无效引用

    ```shell
    error[E0382]: borrow of moved value: `s1`
     --> src\main.rs:4:30
      |
    2 |     let s1 = String::from("hello");
      |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
    3 |     let s2 = s1;
      |              -- value moved here
    4 |     println!("s1:{}, s2:{}", s1, s2);
      |                              ^^ value borrowed here after move
    ```

    **错误原因**：

    对于复杂数据类型，由储存在栈中的**堆指针**(指向真实存储该类型的堆内存)、该类型的**容量**(堆内存分配空间大小)和**长度**(已使用堆内存空间大小)组成。

    **rust解决二次释放(double free)错误**

    二次释放：当两个复杂数据类型的变量指向同一位置。当这两个变量离开作用域，它们会尝试释放相同的内存。这是内存安全性bug之一，两次释放（相同）内存会导致内存污染，它可能会导致潜在的安全漏洞。

    rust通过**所有权的转移**来解决：当`s1`赋值`s2`后，发生所有权转移，所有权从`s1`转移给`s2`后`s1`失效。无需在`s1`离开作用域后`drop`任何东西。这也是为什么`let a = b`称为**变量绑定**。

    

    在rust中对于只拷贝指针、长度和容量而不拷贝数据（其他语言称为**浅拷贝**），但会使"被拷贝"变量失效，这个操作称为**移动(move)**，而不是浅拷贝。

    上面代码可以解读为：`s1`被移动到`s2`，此时`s1`不再指向任何数据，只有`s2`有效，当`s2`离开作用域，它就会释放内存。

    


    另一段代码：
    
    ```rust
    fn main() {
        let x: &str = "hello";
        let y = x;
        println!("x:{}, y:{}", x,y);
    }
    ```
    
    ```shell
    x:hello, y:hello
    ```
    
    这段代码与上一段代码的本质区别在于`s1`持有`String::from("hello")`创建的值的所有权。但`x`只是引用了存储在二进制中的字符串 `"hello"`，并没有持有所有权。
    
    `let y = x` ，仅是对该引用进行了拷贝，此时 `y` 和 `x` 都引用了同一个字符串。关于**"引用与借用"**之后会介绍。

  + **克隆(深拷贝)**

    rust**不会**自动创建数据"深拷贝"。如需深度复制复杂数据类型堆上数据，而不仅是栈上数据，可以使用`clone()`。

    对于始化程序时，或者在某段时间只会执行一次时可以使用`clone()`来简化编程，对于执行较为频繁的代码(热点路径)，使用 `clone()` 会极大的降低程序性能，需要小心使用！

    ```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();
    println!("s1:{}, s2:{}", s1, s2);
    ```

    ```shell
    s1:hello, s2:hello
    ```

  + **拷贝(浅拷贝)**

    浅拷贝只发生在栈上，因此性能很高。

    ```rust
    let x = 5;
    let y = x;
    println!("x = {}, y = {}", x, y);
    ```

    这段代码没有使用`clone()`，但实现类似深拷贝效果，并没有报所有权的错误。

    原因是基本类型在编译时大小是已知，会被存储在栈上，所以拷贝其实际值的速度快。没理由创建`y`后使`x`失效。这里没有深浅拷贝的区别，因此这里调用`clone()`并不会与通常的浅拷贝没有不同（可理解为在栈上做了深拷贝）。

  + `copy`特征

    用在基本类型。如果一个类型拥有 `Copy` 特征，一个旧的变量在被赋值给其他变量后仍然可用。

    任何基本类型的组合可以 `Copy` ，不需要分配内存或某种形式资源的类型是可以 `Copy` 的。

    + 所有整数类型
    + 布尔类型
    + 所有浮点数类型
    + 字符类型
    + 元组，当且仅当其包含的类型也都是 `Copy` 的时候。比如，`(i32, i32)` 是 `Copy` 的，但 `(i32, String)` 就不是。
    + 引用类型

###### 函数传递与返回

将值传递给函数，一样会发生 `移动` 或者 `复制`，就跟 `let` 语句一样，下面的代码展示了所有权、作用域的规则：

```rust
fn main() {
    let s = String::from("hello");	// s 进入作用域
    
    takse_ownership(s);				// 由于s为复杂类型，s 的值移动到函数里之后 s 失效
   	// println!("s:{}", s);         // 再使用 s 就会报无效错误
    
    let x = 5;                      // x 进入作用域
    make_copy(x);                   // 由于x为基本类型是Copy的，所以在后面可继续使用 x
    println!("x:{}", x);			// x 仍可使用
} //  x 先移出了作用域，然后是 s。但因为 s 的值已被移走，所以不会有特殊操作

fn takse_ownership(some_string: String) {// some_string 进入作用域
    println!("{}", some_string);
}// 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn make_copy(integer: i32) {// some_integer 进入作用域
    println!("{}", integer);
}// 这里，some_integer 移出作用域。不会有特殊操作
```

```shell
hello
5
x:5
```

函数返回值也有所有权

```rust
fn main() {
    let s1 = gives_ownership(); // gives_ownership 将返回值 移动到 s1
    println!("s1:{}", s1);

    let s2 = String::from("hello"); // s2 进入作用域

    let s3 = takes_and_gives_back(s2); // s2 被移动到takes_and_gives_back中 a_string 它也将返回值移给 s3
    println!("s3:{}", s3);
} // s3 移出作用域并被丢弃
  // s2 也移出作用域，但已被移走，所以什么也不会发生。
  // s1 移出作用域并被丢弃

// gives_ownership 将返回值移动给调用它的函数
fn gives_ownership() -> String {
    let some_string = String::from("hello"); // some_string 进入作用域
    some_string
}// 返回 some_string 并移出给调用的函数

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String {
    a_string // a_string 进入作用域
}// 返回 a_string 并移出给调用的函数
```

```shell
s1:hello
s3:hello
```

#### 引用与借用

##### 借用

rust通过`借用(Borrowing)`，**获取变量引用**，来解决只能通过转移所有权的方式获取一个值，让程序变复杂的问题。

##### 引用与解引用

###### 引用(符号：&)

常规引用是一个**指针类型**，指向了对象存储的内存地址。

**引用只允许使用值，没有所有权。**

```rust
let x = 5;
// y 是 x 的一个引用
let y = &x;
println!("{}", y); // 5

assert_eq!(5, x);
assert_eq!(5, y);
```

```shell
error[E0277]: can't compare `{integer}` with `&{integer}`
  --> src\main.rs:49:5
   |
49 |     assert_eq!(5, y);
   |     ^^^^^^^^^^^^^^^^ no implementation for `{integer} == &{integer}`
   |
   = help: the trait `PartialEq<&{integer}>` is not implemented for `{integer}`
```

`x`绑定的值`5`的类型为`i32`，`y`是对`x`的一个引用，所以`y`引用的`5`的类型为**指针类型**。虽然打印输出结果显示为`5`，但其实指针类型，不是`i32`。

可以用`assert_eq!(5, y)`断言判断是否是同一类型，结果编译错误为不允许比较整数与引用，因为它们是不同的类型。。

###### 解引用(符号：*)

解出引用所指向的值，一旦解用了，就可以访问所指向的值。

```rust
let x = 5;
let y = &x;
// 用*y来解出引用所指向的值
println!("y:{}", *y);

assert_eq!(5, x);
assert_eq!(5, *y);
```

```shell
5
```

要对`y`的**值**进行断言，要用`*y`来解出引用所指的值(解引用)。解引用后就可以访问`y`所指的`i32`并可以与`5`做比较。

##### 不可变引用

使用引用类型作为函数参数进行传参

+ 无需先通过函数参数转入所有权，再通过函数返回传出所有权
+ 函数参数的类型需要是**指针类型**(&)

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {// s 是对 String 的引用
    s.len()
} // 这里，s 离开了作用域。但因为它并不拥有引用值的所有权，所以什么也不会发生
```

```shell
The length of 'hello' is 5.
```

引用图解：

![图解](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAp8AAAFdCAYAAABFIAYqAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAJxsSURBVHhe7d0FuG1V1f9x61VRXgQJaQFBWkAapEQ6pAUJ6ZTuku5OUbobpBTw0o2kdHdJKN3O//OZ7nnf7fnfC3ftc+4+a+89vs8zn3P22muvXnP+5hhjjvmVFARBEARBEARtIsRnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEARB0DZCfAZBEARBEARtI8RnEARBEAQt8e9//zt99NFH6fPPP28s+T8eeOCBdMwxx6RXXnmlsSQI/kOIzyAIgiAIWuKpp55Ku+++e7ruuusaS/7DJ598kvbbb780zjjjpOuvv76xNAj+Q4jPIAh6mk8//TQ9++yz+W8QBNW49dZb03jjjZd22mmn9NlnnzWWpvTBBx+kDTbYIP3v//5vuuaaaxpLg+A/hPgMgqBnee+999LGG2+cG8+77rqrsTQIghHlwQcfTNNMM03acsst08cff9xY+n/ic/LJJ0/33XdfY2kQ/IcQnx2K+JpXX3019yhPOumktO+++6ZLLrkkx94EQfDlsHTuv//+6Stf+UqIzyBokUcffTTNMMMMaZtttvkv8fn222+nX/3qV+lnP/tZds0HQTMhPjsML/fdd9+dTjjhhLTRRhuleeedN/3whz9M3/rWt9Kyyy6bBWkQBF/O2WefnV2CIT6DoHWK5XPPPff8L7f7Cy+8kOaZZ560wgorpNdff72x9D+IB7333nvT8ccfn+NFTz311PTYY4/lwUvN+Gyw0pAhQ9KZZ56ZjS32N6zBTQXeDOsdcMAB6dhjj0133nnnf4nioB6E+OwwbrjhhrT00ktnwTnVVFOl1VdfPe2www75BT755JPTP//5z8aaQRAMDw3j/PPPn0VnKSE+g6A6t9xySxpzzDHTiSee2FjyH4jLSSaZJG299dbpww8/bCxN6d13381CcplllknTTjttmnDCCdNkk02W1lprrSxACwTm7bffnjbffPMsYn/+85/ndYTJ3HzzzcOM0bZtx2H9UUYZJf3gBz9ISyyxRDrkkEPSSy+91FgrqAMhPjuM8847L4077rjZWjPffPOlyy67LMfWBEHwf2iY/v73v6dddtklN0QsnGuvvXZ2BRZYQ/71r3/luLQQn0HQGmeccUZuj/bZZ5900003Zasjcbnddttlj9wf/vCHoZZKbRUr50wzzZTd8X7D+vnrX/86r3vUUUdl66n1b7zxxixQvb9bbbVVXs+25phjjrTIIosMM470r3/9azbKOB7t5BprrJGWWmqpNP7446e99torjDM1IsRnh/H888/nXpwXUq9STI08ahparowg6GW43ISkeD80QM2FpdOo9mbeeuutbBkJ8RkE1eES1wZ5v37yk5+kX/ziFzkU7Mc//nEaa6yx/mukOxc6i6X1vG+E58svv5yeeeaZ7LmzjT322CN3HLVzyy23XJp11lnT+eefP7TTyDIqvtS6Rtc3G178TmfTd4QsC6lYU2537/jEE0+cjTVBPQjx2YGUmBYv6iqrrJLmmmuutPzyy6e99947XXvttTHoKOhZDj300Nz4KKwmV111VbZuDo833ngjW1FCfAZBdYw/4Db3vhGbc889dxajLJlCw5rfq3feeSe70FkkWS8XX3zxbNHkSidWWUO1a0SqwbN+e9hhh+X2DjwVv//974fGafvNHXfckb8Dgao99J0wAO++bbGiXnjhhXmZEICI/6wHIT47kPfffz/3+IhMIw1PO+203MsjQhdaaKH8goZ7IehF5BwslpHpp58+NzpfNDghxGcQtM4TTzyR3zeCcrfddktXX311evjhh9NDDz2ULZfN75XZjsR2rrrqqun0009Pm2yySbZIKkJfLr744qFC87jjjssi06BAApKH4txzz00zzzxzmn322dOKK66YRh999BxKw3IKg21ZXr37fiscoMSFsn46FqK4OfQmGDxCfHYgf/nLX9L222+f/vznP2ch6uV87bXXckNrxDv3wpc1ukHQrfzjH//ILrnvf//7Q0WoGLRhxUYX8alBe/rppxtLgyAYEZ577rls9OAeJ0QL3jWC8kc/+lG6//7787Irrrgiv49EKlHoPSUKFe735vbqyiuvzILWu2l9FlMi1760bUa8E55mT5L1RdgZFzuLKour33L/E6+sqeJPCdJtt912mPVA0H5CfHYgGtIJJpggN5heKvEyyq677ppdGV7wvmkvgqDXYOHgvtOAeSeIUe8I91+h5CjUqPWNBw2C4Ivxzhj4qt0hIAvGHxB6zeLTFJtE4WKLLfal75rUTNo2llLvrr8rrbRSNrgQjwwuBhyxrnqvN9100zzq3rvM7W9w0dRTT51H04v//ulPf5ozw9x2221hlKkJIT47EGliDDIS52n2CMHVXlBuhRlnnDHHvUjJFC9Z0IuwZnLTFbwHXH5iy1g/xKUVV12IzyBoHW5yYV8sjLxwzQj/ana7E5TrrbdefgfFehKlZZCsv95JnUVxnlIs8UQYGV/ygHLlFzc6vNespsY62FcJuTHTkoFQ55xzTvaA6HCaiKXv74PBJcRnhyJomqtB7IzZjbyg4mTEzWhQ4yULehVWf50x1hBCtKCBO/roo/N3ZR7qEJ9B0D+8V0VENvO3v/0td/RKSiTWyjLynChltdx5551z2+XvmmuumS2UEtbz7o0o9t38LjdP8+lveADrSYjPIAi6Cq41MWhE5vDKgQcemBsl1pcpppgixGcQDDCsohdddFEej1Ao1kpic8kll0zTTTddds17X02essUWW2QjSnP86IgyLPEZ1JcQn0EQdB0aPtaTRRddNLv5CE6pXFhXuPZKOjJ/eQ8uvfTS8BYEQZsQj80yKoen989gJGmbZGlpNVxMblCW1g033DAGFXUAIT6DIAiCIOhoCFeufAOaxHwG9SbEZxAEQRAEHQ0vhoFMptI0zWZQb0J8BkEQBEHQ8UhEL/m83KAR91lvQnwGQRAEQdDxvPjii2nHHXdMJ598cojPmhPiMwiCIAiCjqfM9tec5zeoJyE+gyAIgiAIgrYR4jMIgiAIgiBoGyE+gyAIgiAIgrYR4jMIgiAIgiBoGyE+gyAIgiAIgrYR4jMIgiAIgiBoGyE+gyAIgiAIgrYR4jMIgiAIgiBoGyE+gyAIgiAIgrYR4jMIgiAIgiBoGyE+gyAIgiAIgrYR4jMIgiAIgiBoGyE+gyAIgiAIgrYR4jPoaj7//PP0xhtvpCeeeCLdcccd6S9/+UvjmyAIgqAOvPzyy+lvf/tb41PQC4T4DAYV4vCjjz5K7777bnrzzTfTK6+8kp599tn06KOPpvvuuy/dfvvt6brrrkt//vOf0wUXXJBOPfXUdOyxx6aDDjoo7bbbbmnrrbdO66+/flp55ZXTEksskX72s5+l6aefPk088cRptNFGS1/72tfSmGOOmSaffPI066yzpkUWWaSx5yAIgoHjqquuSvfee2968cUXc53W6/zrX/9Kjz/+eLrpppty3X3MMcekXXfdNa233nppySWXTDPPPHMaf/zx01e+8pX8d5ZZZkm//OUvG78Ovoh///vf6cMPP0zPP/98+sc//pFeeOGF9NRTTzW+7QzaLj5dsMHms88+Sx9//HF65513hgoeN/HJJ59MjzzySPr73/+e7r777ix8vDjXXnttrlguv/zydPHFF6edd945nXnmmVkInXDCCen3v/99OvLII9MhhxySDjjggLTddtulPfbYI4sjL9tOO+2U5ptvvrycWNp8883TxhtvnEXTWmutlVZfffW0yiqrpBVWWCEts8wyWUQRSQsuuGCad95501xzzZWF07jjjpuF1P77758OPvjg/Pfoo49Oxx13XD6O448/Pp1xxhnpnHPOyS/7n/70p/z/ySefnAXboYcemvbdd998TNtuu23adNNN07rrrpv3v+KKK6allloqLbzwwvlY7e+nP/1pmm666dKPf/zjNOmkk6YJJ5wwH8NYY42Vvv/976fvfe976X//93/TqKOO+l+F6PPd6KOPPrSUdb/73e+mUUYZJf3P//xPFobKt771rfw727T9H/7wh2nKKadMM8wwQ5ptttnyNVhooYXy8blGq622WlpnnXXyNdxiiy3ydXVPXO8999wz7bPPPvk8lb333jvtsMMO+Vxd67vuuitfT/e0F3j11VfTP//5z/zeqbCCIBg5/OQnP0mTTDJJrse+8Y1v5HrO/zq+6lL1qjrs17/+dRZZ22yzTW4nDjvssFx3n3322emyyy7LnW1WQG0RQfHMM89krw1R+9JLL2UroTbLu/3aa69l8fH6669nD4/27K233srvPPH39ttvD23n3nvvvfT++++nDz74INcH2sBPPvkkffrpp7lNZAjoW0dYz/a1jY7p6quvzm2K9kTdutVWW6Xf/OY3+bzmnnvuNNVUU6Vxxhknn792wPVw7toy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EEp8eq9ZywcSg41ies0gCAaVp556KqdZWmCBBRpLgipsv/322fLJelfH0cF1R4gF4d4rGPglntmgj6A6LOVyEvdSh2WgmW666fJgxE4hxGcQdCG33nprHu2+6KKLNpYEVTC151hjjZUtMQZPBdUQk9pLI791VsQzy0YQVMcAxBCf/cPg0rXWWqvxqf6E+AyCLkRqI5YYOTKD6hhNPsYYY/RU3OJAImZ2sKZ2HQyWXXbZLD5NfxpU55577hk64ChojXHHHTenqusUQnwGQRciGTmX+8orr9xYElRh8cUXzwMgIm9ja5hAYNRRR2186n5M6CAXpEwOQXVuu+22bPVUIsa6NdRVLPCdQojPIOhC9txzz+xyl9IoqM58882XB2wN1tz4nQ7xKU9qrzD66KOnE044ITp7LWLSBDHCnpkHH3ywsTQYUUyKobO3ww47NJbUnxCfQdCFyJ+5/PLL59jFoDqzzTZbHnCkBNUxzzxB1gtIs8RlLG1YdPZaY8MNN8xhLhNNNFGevjKoxpVXXplj/DfffPPGkvoT4jMIuhCjHpdeeumwxLTIjDPOmBvDwZqetNNhgTHJQS9gpi5TLpoooZPyLNYJaYImnnjinCf18MMPbywNRhQzxC288MJp3XXXbSypPyE+g6ALmXfeeXNDqEEMqmNmKIMfJp988saSoAoGPsgW0AvIaTrDDDPkqWR5HF599dVsDQ1GDDN0cblLFbTgggumjTfeuPFNMKL88pe/TJtuumlaZZVVGkvqT4jPIOhCpplmmjxo5g9/+ENjSVAFLqzvfve7+ToG1VljjTV6JmTBFKfm1RbnusUWW+TJCTw75rYXh+ezaTd1aGSg0DEUj21wErEgPY5O4pZbbpl23HHHPPPXfvvtlwcwEbTeYS79M844I5177rnpoosuSpdeemn685//nK6++up07bXXphtuuCHnVr3pppvSLbfckgfw3HnnnTn5/f33359TGT3++OPZSis+kOB755130scff9w4i5GDKXbt54033sizGEkBZzYzM52Vstdee6WZZ545TTXVVFmATj311GnIkCE59tNxfhGmm3377bfzLGQvvPBCevrpp9Njjz2Wz9d0vToGroFr4Zq4Nq6TWdWuueaadPbZZ+dr+Ne//jXv0zLX87rrrsvrKM3X1r22Dedx/PHH53noJcj/29/+lu666668L/u877778nV/4IEH8nk4nkceeSQ9+uij+b753m9t1/7dz/POOy+ddtppebAo669n4He/+13aZpttsiAX0rHiiivmel1MugkNXDPeGSEuMkzwdnUKIT6DoAsZf/zxsxXBHOVBdcxWY7S7UcxBdQgrkxz0AsSKzBIHHnhgjllcaKGFsrgkEvzlgZCCyfuoyAVq/fnnnz/HFpuFjHidY445sjglxIR9cEETYzpAOkOs8EQs9zTB4R2XXoe4ZWUmQIQ6CBfxv8E7nmECWEyqAXSmAP3mN7+ZLY3SiBHIprRk5Vcs972R09b3O0LaNmxLMQivlLLMOtb1O7+3bdv11z59Z/0xxxwzH5/jLMWx24b9+72/jt3+/d5v/VWMhvfXOo5dscw+rO/3fmsfXPm2XWK3LfdMNhfLXb9yHOX6Wbecn3NvPr+yH/ss17MU16+M2i/H3Hysronif8tLsb7flutvu/ZR9lfuQ997oThOyx2356VTCPEZBF2GOdQ1XISTHnmv43oMq3wRLAoq/ckmmyxbNOqMJPgsWO+//362Mr311lvZEsTqwhrE2mUGGVYXVhhWF+fE8sKSU6xAV111VU4Of8kll6QLL7wwW9tYhs4888x0+umnp1NPPTWddNJJ2eLDGmdWn2OOOSZb51hqDj300HTQQQflqUmJJY2kffYKa6655n+JiyIwmoVT30JkNIuUUvzWsr5ipWyHKCkipYgThRgp4ovQ85m4IlLHG2+8NMEEE2SBTMBOMskk+fkmahVizH2bcsop8/Nfis9F/FpffCvxa5vEm33Zj+NwbI6ViPLZct8Tdrbf9xgUy/zOeaq3zBJFsIthVAh14p0wV6c5HsdOfNsu8eXcXY9y7fper1KKwOtb/LZ5nebfjEgp91Ah/P0t97K5lGNTyr1V/KYcs9L3uCWQbxalzYLUe2aZ3xCfttMJhPgMgi6DsOKi01hw2YgHkjSdK9Tod7FB2223XXbp7LPPPtm9R0RIFeN3559/fnYDESOECRcTsVLcSA8//HB2bRE0pvEkbrjPCB0uPe61V155Jb322mtZBCmWP/vss3l97icuMdu0bfvg9uKOInwuuOCCLHoIHgKI2ClC54gjjsjuzYMPPjhbmvbff/8cbO88uO/22GOPtNtuu+Vz22WXXdLOO++cdtppp+zOVPxvme+ss+uuu+ZS1jNQRq48liiNmkpdY8ctuu222+bvref39mN/9mv/3GSEFwFGiDlOouzII4/Mxf+K9RXrOn6/lRqLu9UxORbH6jjcP4U7d7PNNssxhY7FfTS4QGJz7rjVVlstu3ANNJPlgOVRo12sb8Xixl2nIdfAzznnnNnyplHnwmMpYmUTv6iwshXrG5ex7xRuUeLB/5b73np+w2I300wzDRVHRuH2Cu6N6zLQcC2LIf3nP/+Z3ynvmXeGsG9+j7iHvUc6EBdffHF243pWpDHikuXO9R4dffTR+T0qnQXPrewEnuXyTnjmttpqq/zcGUHt2VNvNBfLfGcd62699dZD3xEhBK6HZ9SzyV1s3nGi0rvlWSGoCGPPShFk3g+ClgAjWtVhnlHP8K9//eu8Te+eOss56STpOLkG6iN1DVd/J6G+/vTTT3Mn0l/3W2fSeSgffvhhevPNN/Mz8O677+YOplAD+VA9E77j7vfs6Qh2ysQiIT6DoAsxH/kKK6yQR7urlFgriIli6SgWDkKCiCgCgmWBgFCIES5AwkQDUMrss8+eCzdhcyFmyt++xfpF5BAnGh/7tG/Hwc2tMSpuRcdcXIssI6wsCutJ+VtK+c561mdZ8VvbsC3nzlJi2+XcWXTKeZdzby6Op7lRtP2+x9K831KsU0o5jlJ8Voplqnxu/k3ztsr2WXd8bj5njbZiW8WtaD2FxauU4tokpO3TesWlyGJSLGalsKwUa4q/LFeK9RW/VVhebE+x/bI/+y/iQdFJ6BUIMs9bUB2hB94znV8C8vPPP298E4wIRCvL56qrrpqNCJ1AiM8g6EIE1xNfxx57bBaEhF+z6CM6+4o+Ao1Q04ASOM2Cqa9IUsryIvQUYqkIPULXtgk+++lbiMBSHEtx8xVBzKpWBHGxqhHEChFM1LLgseSJnyuuusUWWyxbWVh8WQFZA1lNVl999ewaZY3ZYIMN8ojsYsEp1hzLSiGqVOiK7Zc4PX8VlkSFVbHse5FFFhka77fEEksMPQ7TL7JIOJ6ynDXIsbFK6ySwXDpODQhrkeKYfbfeeuvlgSksneuss04+B8v8xrmwhH5RKVbv4a1ruWJ7rGUGOLAyNV8jhZVLcR1Zt5qtYtb1O793r1w3FuxewTUI8dkanjXuZx6XoDVKvWzQVScQ4jMIuhCNPgEpRo+LnQtX4RLj0i2uZy427mMu3+I+5gr2mdWK65iLjquOkOW2E+/HFc7txaVnhCaLxTnnnJP3x+VnoBOXKzegkaPNxQANrrLm0jzCtHnULnei2ESjVbnWzAHtM3ejEaRcj9xtKlyjaF988cXs9pfuhgXFKFuuKW4qbisWYS4tLq4Ct1eJm+TiEjvJvcXqy3rnOhLz4mftv3nfTz75ZA474Aq1X/vkBiv7sz1WiS+LMa2CbbEMKc2uumZ3nf06V8X5FJddSQNkufX8xjYG8vhw3HHHZfFp+70CEa4DFlRHSA3xaeR30Bo67TrMnUKIzyDoQsROskKKmwxaQ4gAAeVvUA3xhYR7L8EKHOKzNXRAiU+d0KA1eLaEN3UKIT6DoAsxQlmcIEtl0BoqcyNfufGDarCOu3a9hFAKVvKgOjwHxKdcm0FrCFXqlMFGCPEZBF2I2CkNodGsQWuINTViW1xmUA3i0+ClXsKIb/HPQXWEfRCfMm0ErSHGXvx2pxDiMwi6EOmMjD4Wuxm0hoFZxKcBNEE1xHwaEd9LSDUU4rN1iE/x5UFrCLOS6qpTCPEZBF2IvJpS48jXF7SGAH7iU/7DoBosn1I39RLeNdkfgtYgPg2ErCsG9hnIV1fU9/K0dgohPoOgC5GIWEMYVrvWkQJK3KLE10E1iE95QnsJ2STEWQetQXxKV1VXZPSQRu3L5psfLAzwk+KtUwjxGQRdiF66hlCuyKA15Cg1tZ1ZloJq9KL4XHLJJXNC/qA1iE+5b+sK8SnfcB2njJVOjpemzpbjvoT4DIIuRUNoBG7QGhLusybEIIjqSLXUi5bPGO3eOsSnSRzqSp3Fp1zDZh4z8UOnEOIzCLoUlk9zeQetYbYaU/5JnB9UwyQEvSY+Td5gatGgNYhP02zWlTqLzwceeCAPODJLWacQ4jMIuhRTVpqKMmgNlk/iU87UoBrml+61AUdcnnWbYUYKo/vuuy9Pyep5JvBM9Xr33XcP+KxW/cWxmaa2rhCfjs8sZnXDDHCmNTYdb6cQ4jMIuhSzXRCgvcC//vWvPK2nKTYHCrPVEJ+m/awLYnlZOUzxydX21ltv5fLwww/n6Uj9XwdMvdpr4tNUtVyfdcIUt0SnYjT+uuuum8NxfDZdbJ0Q4jLPPPM0PtWPQw89NC2yyCJ5yt664d03QHL55ZdvLKk/IT6DoEtRUXIddxPmLme1kUqqmaeeeiqfL+vEQCFnowbxhhtuaCwZfIjrtdZaKzcya665Zp7SUfnVr36VFl544XTFFVc01hxcDNL69re/3fjUG8ipO+qoozY+DT7SAm288cY5/OaOO+7Iy1g7xeOyMhJTdcIxcWvXlT333LO24lMdxdDQSbOxhfgMgi7FyFExaHXOTVeV5557Lq2xxho592az+0scloZLkmUCjVWwWAZbRaoq4vOmm25qLBl8HAvL1QorrJAHuGy44YZZhCrSarGA1IFTTjml52Y42nvvvWsnuFnIjz322P8STOecc04Wevvuu29jST1wTHUOEyI+xVR+8MEHjSX1gfg0KcZCCy3UWFJ/QnwGQZfCGsYSU8eeequ88sor2cJkGrmbb745W0GvueaadMghh2Qr7yyzzJLj21gD99lnn3Tbbbc1flkdFiPisy6CjtWKRdHAguuvvz43gq5HEdqvvvpqbeL4Tj311J6b233//fdPX//61xuf6slnn32WB0YRepdcckljaT1wTARUXamz+NQpnXbaaTtqgGmIzyDoUljD5H578cUXG0uGjThC4oXIevDBB7Nruy+ff/55rnSVLxI47733Xnr22WdzDObw1vvkk0/S+++/P8z9FPyWa/3WW29N9957b16/YNsC7IlOFsB55503zTzzzNkiyHIito1L0UAhFlHbeu211/4/cfbRRx/lazO84yji0zHUARbs3/72t+nnP/95Pu4RuY6DhQFHdRdiA81BBx2Unxf3pa4UD8Hcc8+d3/k6Ib56iimmaHyqFyWEgXAn4OuG+F3Cfc4552wsqT8hPoOgS9liiy1yYygecngQYCwghKqYQa56eS0tB7HG1X3MMcdkN69iMElfa6r1DIQxvZuYRPnmLrrooiwUC0Tu7bffnl3m2223Xdppp53ygIi+4sl63Eism2KslllmmZy6p2xL5e83jouF0zHtt99+aZpppsnza7/77rv/JTIJYsJA4vNyXsS0+Mj11lsv3XPPPXlZX8Yff/x8/RxzHXj66afzILK55porHXXUUXl0tevIEswK7JzqgmfEteslPGPc7nUZ9FXwPnhfhanssMMO2cIo7rP5HakDxOdUU03V+FQvdLpZPVk/64iYXp0KnfBOIcRnEHQphAnrk5HQw4LIu+yyy7IVxLRxRNwCCyyQg9aLtfSJJ55IW221VVpuueWyoGRVZHk77LDD8hSeBQ0uITjKKKPkmYHM9kI4EnwaP/tiqVxl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hUTHRHYE6bnERns2PDeeF4UVU/0tjnPyySfPAtP52I5lBvPpFBDbRDNhvfTSS6f33nuv8TTWmxCfQdCFGPzAUqJSizyV1TFgZp555smNCUERfDlmtjKFqcaPJYtFxujb5557rrFG90NUcL0TFIssskguZnqSBNz1YPn0XM0111xZaAiLmXbaadMUU0yROzvyNEoUTlARNf76TKia99x6fuO3c889d96ebS+++OI5qT+L6oorrphWXnnlbFEt6a7WWGONXFZbbbX061//Oq200kp5XWLFb23DtsSJzz777Fn0OIepp546Cx/W3PHHHz9b15wfgUkIEVxEFuFISFvHcRJSM8wwQxZQBLhrYF9yv7Kobrzxxjkd15Zbbpk233zzvD2izDkTbEVEev9sm5grQs7+DU5SxInap+PT2XZ9xNwS+o5f/ssi6LimLXP9dQb8db6uo/vhe+s5f8ftd8Shbbjmzofoc79cF8+2a6Mj0dyZcL+KgG4WxM7B+ShFrBbhWkRss9Aupe9vlL7rlOIZ6pTOXojPIOhS7r333mxl4Q4rLrpzzz03u9+457icuN64wq677rocK8Sd4y9300MPPZTTdjz99NNZQLzwwgvZHf3aa6+l119/Pb311lvp7bffzmLjgw8+yG4o4oMrStJ5RcL5zz//PAuTgZ5203btw2hPFa4BH9KNODauKcfr2LnLnItz4k4zAMv5ckm6Bueff36+NiwH3JtceK6bhkalz7rwxhtvpH/84x/p1Vdfzdt+6aWX8vZN2Sng/5lnnsnz2j/xxBN5f48++mh21fmruI6llO8ffvjh7H6U9N6xsFA7PveA60+srmN1PyQwd8xclO6X4/Yb9++KK67In7k1ueLcWy5F58Sl6vwU916xDnF46qmn5nW4IT0TnhHFcuXkk0/OhdvUtVF0aBTuTC5O7k9TaJrZiovUs7bLLrtkqychplFuxyQHdYArnFWK8CJkmgtxQ+QoRA9BZiak+eefP4s+wpQAJFa55ZdYYom01FJLZcHmr7Lkkkv+VynLrUN4LrPMMllQEpvEJQFKaBKgOlBSX62//vpZ+HmmCb/tttvuv9z93Njc1dz7ngXPDRe+587z6FnlVfEeDKSFjZgjzgg62SU8y55p74H8vJ4hz+Kw3gnPuHdiWO+Dd8H3whI897btWTcYznPvOT/llFNyaX7WTzjh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EARBEPST8cYbL/3yl79sfBp8WOhMq3nCCSdkwcnFLg+pv7vttlu69dZb0+eff95Ye3DZfvvt0ze+8Y00yyyzNJbUD9dTzlTz+NeN/fbbL8fwDhkypLGk/oT4DIIgCIJ+Qnwuu+yyjU9BFYwkN0MUy2JQnRNPPDFNNNFE6frrr28sqT8hPoMgCIKgnxjtPt988zU+BVVYffXVs/iUrzKozhVXXJHGHnvsdNNNNzWW1J8Qn0EQBEHQT8Ly2Tqu2ze/+c3aJenvFO69994cw3vLLbc0ltSfEJ9BEARB0E/qFvPZSZiW9Fvf+lZYjlvETFYsx6ZT7RRCfAZBEARBPzHafemll258Cqow77zzZvG5wAILNJYEVTFg669//WvjU/0J8RkEQRAE/USezxCfrSFHJfEpJVTQGtzup5xySuNT/QnxGQRBEAT9ZLTRRssiKqiO5PLE5y9+8YvGkqAqOj97771341P9CfEZBEEQBP3EaPc6JZnvJKaccso84GjuueduLAmqMumkk6b11luv8an+hPgMgiAIgn4yxhhj1Gp6zU5isskmC8tnP5luuuk66vqF+AyCIAiCfsLtbvagoDoSpLN8xoCj1pljjjmyiO8UQnwGQRAEQT/57ne/G+KzRaSpMlp7nnnmaSwJqmLq1K997Wvp448/biypNyE+gyAIgqCfsNzJVxlUZ5xxxsnCac4552wsCaqy6KKLph//+Mfp9ttvbyypNyE+gyAIgqCfEE8rrLBC41NQhe9///vp61//epp55pkbS4KqLLbYYjnm+JhjjmksqTchPoMgCIKgH9x8881ZfC6++OLpiSeeSM8//3x67bXX0nPPPdd2N+i///3v9Nlnn6VPPvkk7/vDDz9M77//fnrvvffSO++8k4/p7bffzv9b5jvrfPTRR/k3n376afr888/zdkYmjtH+zc4jR6UZesYff/x05plnpgsuuCBdfvnlOWn6jTfemO6444503333pYcffjg99dRT+fq++uqr6a233krvvvtuPnbHXAdcc9fWeb388sv5ensmHPv999+f7rrrrmydNA/7ddddl4YMGZKuvPLKPD/7pZdemi6++OJ8/ueee246++yz8/U4/fTT02mnnZbzeJ588snppJNOyuXEE08cWqSruvbaaxtHUX9CfAZBEARBP/jLX/6ShRMRNfHEE+eci6OPPnpOIcSiR5gazS0u1MCk733ve9naZ4S89RTLfGcbo446avrOd76TRhlllPw7Ln0xkYrtlW36+9WvfjWXr3zlK0NLWaZYr7nYZt/lzesrzdtyXM3fNf9OKcfS/LnvsrLN8nvfE5vOy9/m/fn87W9/O5+/66C4Js2l+fqUbZT92LZlvrOea+r6ut7SYY099thDC3d/+d93Y4455tD74ry/6F6UcyzXpfkcyrJSynXwG78t5257zefqWO3bcXiGPFMGY0mjNMUUU6Spp546EZksxLPPPnuOkf35z3+eBxs5B0K2UwjxGQRBEAT94M0338zxnoREXwFShEf5XL77IjFC6BAkfQvxqjQvs67fKEUc2V6zUFXKvvseXynD+64c//BK+W1Zt5yPY3F8RBUhR1ARexNMMEGaZJJJspiaZppp8v/+lm1Ztwi/cn7DO6/mYyil+TxKGdby5t/Yhu0NTxi65o6HOFSKkHU+plUlEieccMLc8XB+BlAZef6jH/0oTT755PlcxWMqOiSK3/hrme+b1/H78l353rZs0z7LfghUx+H4Hetvf/vbtPnmmzeeynoT4jMIgiAI+gH3L5Fies3lllsuu1BvuOGGdPfdd6dHH300vfDCC3mdOoxE5qb+5z//mQXzP/7xj+y+5h5+6aWX0osvvpieeeaZ9Nhjj6UHH3ww3XvvvdnlLayAS/fqq6/O7vCLLroou4W5g7l/jz322HT44YenAw88MO2xxx5pxx13zCJI0vNVV101LbPMMmmhhRZKc801V5phhhmykCKciDrir4hjfx3XQOF6CysQZuD6O18u+8cffzz9/e9/T1dddVW6/vrr899LLrkkn9Opp56a/vjHP+ZzOuKII9LBBx+c9ttvv7Tnnnum3/3ud2mTTTZJ2267bdpqq63yORJ8G220UVp//fXT2muvnTshyiqrrJJ+9atf5TjgZZddNj8bsiEYGORaTD/99Nlqqcw///y5zDfffHmee99JuK8YhMWyydI522yzZeun/xXfuaYEsWt31FFH5fvQCYT4DIIgCIJ+QMxx7a6zzjo5/i4YcW677bbsVmbhZMELqrPaaqtl8S6OtlMI8RkEQRAE/eBf//pXtj6xahkkE4w4rK7NcZZBdbbbbrtsRTbIrVMI8RkEQRAE/YBbV0ygwSCPPPJIY2kwooixZPkU6xhU59BDD80C/qGHHmosqT8h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/PJ0++23pyeeeCK9+eab6d///ndji93Bp59+mt59991sGZYR4NFHH0333HNPFudXXnllFuunnnpqFp3Ep7jFW265pfHr9uP6f/bZZ+mTTz5JH374YXr//ffz8f/rX/9Kb731Vj6PBx98MP9lofWd9ZznYHPeeeflZ+2GG25oLKk3IT6DIAiCoB+88MIL6X/+53/ST3/605zoW5E2aK655kqzzDJLFp9TTTVVFqQTTzxxGm+88bKVirV01FFHTaOMMkr6xje+kQeMKLbFje87rmgJ7McZZ5z8uwknnDD98Ic/zGl1TOkpt+N0002XR4rPNNNMeX+zzTZbFnKOYZ555knzzz9/+vnPf55nwCGCZ5111rTooovmudT9VRZZZJH8nXWsu8ACC2QRPe+88+bzmXHGGfM2Z5999rwPn/ue17jjjpuneTT4yvnIPekciKKJJpooTTnllPkYHZd9mYucIHcdiE/bWmWVVdJ2222X1l9//bTyyivnKTcdv3OyvymmmGLovlwXWQbsz/Vy3cp+XcevfvWrudh232I9f4f3vVJ+31zso9ynvvvwt9w/RefDvXV8ZiDyf7mXjt99dP88N+6T+7HiiiumpZZaKm200UZp1113zRbNY489NlvU//SnP2XB/ve//z099NBD6ZFHHsmWYpb3K664Iu/jkksuaTyV9SbEZxAEQRD0E4nSb7zxxlxY9m6++eZcWPKUW2+9NZfbbrstFxbPO+64I5c777wzWwD9P2TIkCwyjAD/wx/+kA477LA8il4ap6222irPorTWWmtlgUi8EWcLLbRQFopF7BKi0047bRanUvAQa+OPP34WPUUcFtHLYkusEU/++kzIFcFEGPqNv0U02VYRwZNOOmneB+slcUlMTTPNNFkQ/+QnPxluISQV69kv4VZEuGJ/9mF7xPKCCy6YZ0BiWRbfuOOOO6Z99903HXXUUdmKTJSdfPLJ+Zrfdddd6YEHHsjC7Nlnn00vvfRSHhTGusyK+c4772SrpZmBWDBZOD/44INc/G+5dQzeKVZPv2XxZL21PblJH3vssSwE7c+9Ncc/a/bpp5+exeIf//jHdOSRR6YDDzww7bnnnlnYu4eOf9VVV81CW2J9ywlyItT56mQ03y8C22AiYtZ9c69cI58tt0znoZMI8RkEQRAEPc7nn3+e3c0EGGFGlL3xxhtZtL3yyis5l6k4VqKLpU2oAPFFjD388MPZEsclTfQRZGIPlfvuu2+YxXfWU/zGb20rBhx9OUIDXn755XTvvfemq666KotdpZMI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQdsI8RkEQRAEQRC0jRCfQRAEQRAEQZtI6f8Bp+Ppb9yytIcAAAAASUVORK5CYII=)

`s`通过`&s1`赋值一个指向`s1`的引用，但不拥有它，当引用离开作用域后，其指向的值不会被丢弃。



与变量默认不可变一样，**引用指向的值默认是不可变的**

```rust
fn main() {
    let s = String::from("Hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

```shell
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
  --> src\main.rs:70:5
   |
69 | fn change(some_string: &String) {
   |                        ------- help: consider changing this to be a mutable reference: `&mut String`
   							------- 帮助：考虑将该参数类型修改为可变的引用: `&mut String`
70 |     some_string.push_str(", world");
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ `some_string` is a `&` reference, so the data it refers to cannot be borrowed as mutable
                                         `some_string`是一个`&`类型的引用，因此它指向的数据无法进行修改
```

##### 可变引用

引用指向的值默认是**不可变**的。想要可变，首先声明变量`s`为可变类型`let mut s`，其次创建一个可变的引用`&mut s`。

```rust
fn main() {
    let mut s = String::from("Hello");

    change(&mut s);
    
    println!("{}", s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

```shell
Hello, world
```

###### 同一作用域内，特定数据只能有一个可变引用

```rust
let mut s = String::from("hello");

let s1 = &mut s;
let s2 = &mut s;

println!("{}, {}", s1, s2);
```

```shell
error[E0499]: cannot borrow `s` as mutable more than once at a time
  --> src\main.rs:80:14
   |
79 |     let s1 = &mut s;
   |              ------ first mutable borrow occurs here
80 |     let s2 = &mut s;
   |              ^^^^^^ second mutable borrow occurs here
81 |
82 |     println!("{}, {}", s1, s2);
   |                        -- first borrow later used here
```

这种限制是出于安全的考虑。也是编译器`borrow checker`特征之一。

这种限制使rust在编译期就避免数据竞争。数据竞争会导致未定义行为，很可能超出预期结果，难以在运行时进行追踪，诊断和修复。

造成数据竞争的行为：

+ 两个或更多的指针同时访问同一数据
+ 至少有一个指针被用来写入数据
+ 没有同步数据访问的机制

###### 可变引用与不可变引用不能同时存在

防止数据被修改造成数据数据污染，可变引用与不可变引用**不能同时**存在。

```rust
let mut s = String::from("hello");

let s1 = &s;
let s2 = &s;
let s3 = &mut s;

println!("{}, {}, and{}", s1, s2, s3);
```

```shell
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
// 无法借用可变 `s` 因为它已经被借用了不可变
  --> src\main.rs:81:14
   |
79 |     let s1 = &s;
   |              -- immutable borrow occurs here // 不可变借用发生在这里
80 |     let s2 = &s;
81 |     let s3 = &mut s;
   |              ^^^^^^ mutable borrow occurs here // 可变借用发生在这里
82 |
83 |     println!("{}, {}, and{}", s1, s2, s3);
   |                               -- immutable borrow later used here 
   								  	  //  不可变借用在这里使用
```

##### 引用的作用域

旧版本(< rust 1.31)

引用的作用域与变量的作用域是一样的，这样同时借用不可变引用和可变引用同时存在，使得编译不通过。

新版本

引用的作用域为**最后一次使用引用的位置**，使得在一些情况下借用不可变引用和可变引用同时存在可以顺利通过。

```rust
fn main() {
    let mut s = String::from("hello");

    let s1 = &s;
    let s2 = &s;

    println!("s1: {}, s2: {}", s1, s2);
    // 新编译器中，s1, s2作用域在此结束

    let s3 = &mut s;
    s3.push_str(", world");

    println!("s3: {}", s3);
    // 新编译器中，s3作用域在此结束

    let s4 = &s;
    println!("s4: {}", s4);
} // 新编译器中，s4作用域在此结束
  // 旧编译器中，s1, s2，s3，s4作用域在此结束
```

```shell
s1: hello, s2: hello
s3: hello, world
s4: hello, world
```

##### NLL(Non-Lexical Lifetimes)

这种编译器优化行为，专门用于某个引用在作用域( } )结束前就不再被使用的代码位置。

##### 悬垂引用(Dangling References)

悬垂引用(悬垂指针)：指针在指向某个值后，这个值被释放掉了，而指针仍然存在，其指向的内存可能不存在任何值或已被其他变量重新使用。在rust中编译器可以确保引用不会变成悬垂状态。编译器确保数据不会在其引用前被释放，要释放必须停止其引用的使用。

```rust
fn main() {
    let ref_to_nothing = dangle();
    println!("{}", ref_to_nothing);
}

fn dangle() -> &String {// dangle 返回一个字符串的引用
    let s = String::from("hello"); // s 是一个字符串
    &s // 返回字符串 s 的引用
}// s 离开作用域，被丢弃，其内存被释放
```

```she
error[E0106]: missing lifetime specifier
   --> src\main.rs:100:16
    |
100 | fn dangle() -> &String {
    |                ^ expected named lifetime parameter
    |
    = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
    // 该函数返回了一个借用的值，但是已经找不到它所借用值的来源
help: consider using the `'static` lifetime
    |
100 | fn dangle() -> &'static String {
```

因为`s`是在`dangle()`内创建的，当`dangle()`被执行完毕后，`s`被释放，此时返回的**引用**指向的是一个无效的`String`。

**解决的方法是返回一个**`String`**将所有权转移给外面的调用者**

```rust
fn main() {
    let ref_to_nothing = dangle();
    println!("{}", ref_to_nothing);
}

fn dangle() -> String {
    let s = String::from("hello");
    s
}
```

```shell
hello
```

##### 借用规则总结

+ 同一作用域内，只能一个可变引用，或多个不可变引用
+ 引用必须总是有效的

