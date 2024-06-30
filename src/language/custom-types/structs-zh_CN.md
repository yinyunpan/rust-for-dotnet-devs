# 结构体

Rust和C#中的结构体有一些相似之处：

- 它们是用`struct`关键字定义的，但在Rust中，`struct`只是定义数据/字段。函数和方法的行为方面定义 _实现块_（`impl`）。

- Rust中它们能实现多个traits，C#实现多个interfaces。

- 它们不能被子类化。

- 默认情况下，它们在栈上分配，除非：
  - .NET中装箱或强制转换为interface。
  - Rust包裹在一个智能指针中，例如`Box`, `Rc`/`Arc`。

在C#中，`struct`是一种在.NET 中对 _值类型_ 进行建模的方法，这通常是
一些特定领域的原语或具有值相等语义的复合。在Rust中，`struct`是任何数据结构的主要构造（另一个是`enum`）。

在C#中的`struct`（或`record struct`）具有按值复制和默认值相等语义，但在Rust中，这只需要使用[`#derive`属性][derive]和列出要实现的traits：

  [derive]: https://doc.rust-lang.org/stable/reference/attributes/derive.html

```rust
#[derive(Clone, Copy, PartialEq, Eq, Hash)]
struct Point {
    x: i32,
    y: i32,
}
```

C#/.NET中的值类型通常由开发人员设计为不可变的。这被认为是从语义上来说的最佳实践，但这种语言并不
防止设计进行破坏性修改或就地修改的`struct`。在Rust中也是如此。一种类型必须有意识地发展为不变的。

由于Rust没有类，因此类型层次结构基于通过traits和泛型实现子类共享行为，以及使用虚拟调度实现多态性[trait objects]。

  [trait objects]: https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types

在C#中考虑 `struct`以下表示矩形：

```c#
struct Rectangle
{
    public Rectangle(int x1, int y1, int x2, int y2) =>
        (X1, Y1, X2, Y2) = (x1, y1, x2, y2);

    public int X1 { get; }
    public int Y1 { get; }
    public int X2 { get; }
    public int Y2 { get; }

    public int Length => Y2 - Y1;
    public int Width => X2 - X1;

    public (int, int) TopLeft => (X1, Y1);
    public (int, int) BottomRight => (X2, Y2);

    public int Area => Length * Width;
    public bool IsSquare => Width == Length;

    public override string ToString() => $"({X1}, {Y1}), ({X2}, {Y2})";
}
```

在Rust中对应是:

```rust
#![allow(dead_code)]

struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }

    pub fn x1(&self) -> i32 { self.x1 }
    pub fn y1(&self) -> i32 { self.y1 }
    pub fn x2(&self) -> i32 { self.x2 }
    pub fn y2(&self) -> i32 { self.y2 }

    pub fn length(&self) -> i32 {
        self.y2 - self.y1
    }

    pub fn width(&self)  -> i32 {
        self.x2 - self.x1
    }

    pub fn top_left(&self) -> (i32, i32) {
        (self.x1, self.y1)
    }

    pub fn bottom_right(&self) -> (i32, i32) {
        (self.x2, self.y2)
    }

    pub fn area(&self)  -> i32 {
        self.length() * self.width()
    }

    pub fn is_square(&self)  -> bool {
        self.width() == self.length()
    }
}

use std::fmt::*;

impl Display for Rectangle {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        write!(f, "({}, {}), ({}, {})", self.x1, self.y2, self.x2, self.y2)
    }
}
```

在C#中`struct`从`object`继承了`ToString`方法，因此它 _覆盖_ 实现提供自定义字符串表示。由于Rust中没有继承，
因此类型的方式支持 _formatted_ 表示，是通过实现`Display` trait。这使得该结构的实例能够参与格式化，如下面展示调用 `println!`：

```rust
fn main() {
    let rect = Rectangle::new(12, 34, 56, 78);
    println!("Rectangle = {rect}");
}
```
