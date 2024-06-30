# 枚举类型

在C#中，`enum`是一种将符号名称映射到整数值的值类型：

```c#
enum DayOfWeek
{
    Sunday = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
}
```

Rust 具有几乎 _相同_ 的语法来执行相同的操作：

```rust
enum DayOfWeek
{
    Sunday = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
}
```

与.NET不同，Rust中`enum`类型的实例没有任何继承的预定义行为。它甚至不能参与平等检查就像`dow == DayOfWeek::Friday`一样简单。
为了使其在某种程度上达到C#中`enum`的函数同等水平，使用 [`#derive`属性][derive]来自动让宏实现通常需要的功能：

```rust,不可编译
#[derive(Debug,     // 启用格式"{:?}"
         Clone,     // 复制需要
         Copy,      // 启用按值复制语义
         Hash,      // 启用用于映射类型的哈希功能
         PartialEq  // 启用值相等（==）
)]
enum DayOfWeek
{
    Sunday = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
}

fn main() {
    let dow = DayOfWeek::Wednesday;
    println!("Day of week = {dow:?}");

    if dow == DayOfWeek::Friday {
        println!("Yay! It's the weekend!");
    }

    // 强制为整数
    let dow = dow as i32;
    println!("Day of week = {dow:?}");

    let dow = dow as DayOfWeek;
    println!("Day of week = {dow:?}");
}
```

如上面的示例所示，`enum`可以被强制为其分配整数值，但与C#一样，相反的情况是不可能的（有时C#/.NET缺点是`enum`实例可以保存未表示的值）。相反，由开发人员提供这样的辅助函数：

```rust
impl DayOfWeek {
    fn try_from_i32(n: i32) -> Result<DayOfWeek, i32> {
        use DayOfWeek::*;
        match n {
            0 => Ok(Sunday),
            1 => Ok(Monday),
            2 => Ok(Tuesday),
            3 => Ok(Wednesday),
            4 => Ok(Thursday),
            5 => Ok(Friday),
            6 => Ok(Saturday),
            _ => Err(n)
        }
    }
}
```

`try_from_i32`函数在`Result`中返回一个`DayOfWeek`，表示成功（`Ok`）。如果`n`有效将按原样返回`Result`中的`n`指示失败（`Err`）：

```rust
let dow = DayOfWeek::try_from_i32(5);
println!("{dow:?}"); // prints: Ok(Friday)

let dow = DayOfWeek::try_from_i32(50);
println!("{dow:?}"); // prints: Err(50)
```

Rust中存在一些板条箱可以帮助实现这样从整数类型的映射，而不必手动编码。

Rust中的`enum`类型也可以作为设计（有区别的）联合的一种方式类型，允许不同的 _变体 保存特定于每个变体的数据。
例如：

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

这种形式的`enum` 声明在C#中不存在，但可以用（类）记录模拟：

```c#
var home = new IpAddr.V4(127, 0, 0, 1);
var loopback = new IpAddr.V6("::1");

abstract record IpAddr
{
    public sealed record V4(byte A, byte B, byte C, byte D): IpAddr;
    public sealed record V6(string Address): IpAddr;
}
```

两者之间的区别在于Rust定义会产生一个 _closed类型_ 变体。换句话说，编译器知道除了`IpAddr::V4`和`IpAddr::V6`之外，没有其他`IpAddr`的变体，并且它可以使用
使用这些知识进行更严格的检查。例如，在`match`中这种表达式类似于C#的`switch`表达式，Rust编译器会除非涵盖所有变体，否则将返回失败代码。相比之下，使用C#进行模拟
实际上创建了一个类层次结构（虽然表达得很简洁），并且由于`IpAddr`是一个抽象基类，因此它可以包含的所有类型的集合表示对编译器来说是未知的。

  [derive]: https://doc.rust-lang.org/stable/reference/attributes/derive.html
