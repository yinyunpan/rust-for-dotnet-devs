# 成员

## 构造函数

Rust没有构造函数的任何概念。代替你只需要编写工厂方法返回该类型实例的函数。工厂函数可以是独立或类型 _关联函数_。在C#术语中，关联函数就像在类型上具有静态方法。
通常，如果`struct`存在一个工厂函数，它的名字就是`new`：

```rust
struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }
}
```

由于Rust函数（关联或其他）不支持重载，这个工厂函数必须具有唯一名称。例如，下面是一些`String`上可用的所谓构造函数或工厂函数的示例：

- `String::new`: 创建一个空字符串。
- `String::with_capacity`: 创建具有初始缓冲区容量的字符串。
- `String::from_utf8`: 从UTF-8编码文本的字节创建字符串。
- `String::from_utf16`: 从UTF-16编码文本的字节创建字符串。

在Rust的`enum`类型中，变体充当构造函数。更多信息请参见[枚举类型部分][enums]。

另请参阅：

- [构造函数是静态的、固有的方法（C-CTOR）][rs-api-C-CTOR]

  [enums]: enums.md
  [rs-api-C-CTOR]: https://rust-lang.github.io/api-guidelines/predictability.html?highlight=new#constructors-are-static-inherent-methods-c-ctor

## 方法（静态和基于实例）

与C#一样，Rust类型（包括`enum`和`struct`）可以具有静态和基于实例的方法。
在Rust语言中，_方法_ 总是基于实例的，并且通过其第一个命名为`self`参数这一事实来识别。`self`参数没有类型注释，因为它始终是属于方法。
一个静态方法被称为 _关联函数_。在下面的例子中，`new`是一个关联函数，其余的（`length`, `width`和`area`）是以下类型的方法：

```rust
struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }

    pub fn length(&self) -> i32 {
        self.y2 - self.y1
    }

    pub fn width(&self)  -> i32 {
        self.x2 - self.x1
    }

    pub fn area(&self)  -> i32 {
        self.length() * self.width()
    }
}
```

## 常量

与C#一样，Rust中的类型可以具有常量。然而，最有趣的方面是注意Rust也允许将类型实例定义为常量：

```rust
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    const ZERO: Point = Point { x: 0, y: 0 };
}
```

在C#中，同样的需要一个静态只读字段：

```c#
readonly record struct Point(int X, int Y)
{
    public static readonly Point Zero = new(0, 0);
}
```

## 事件

Rust没有内置类型成员支持通告和触发事件，就像C#的`event`关键字一样。

## 属性

在C#中，某个类型的字段通常是私有的。然后他们就是由属性成员使用访问器方法（`get`和`set`）读取或写入这些字段。
访问器方法可以包含额外的逻辑，例如，在设置值时验证值或在读取时计算值。
Rust只有方法[其中字段getter命名（在Rust方法中，名称可以与字段共享相同的标识符）和setter使用`set_`前缀][get-set-name.rs]。

  [get-set-name.rs]: https://github.com/rust-lang/rfcs/blob/master/text/0344-conventions-galore.md#gettersetter-apis

下面的示例显示了，对于Rust中类型的类似属性访问器方法的典型外观：

```rust
struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }

    // like property getters (each shares the same name as the field)

    pub fn x1(&self) -> i32 { self.x1 }
    pub fn y1(&self) -> i32 { self.y1 }
    pub fn x2(&self) -> i32 { self.x2 }
    pub fn y2(&self) -> i32 { self.y2 }

    // like property setters

    pub fn set_x1(&mut self, val: i32) { self.x1 = val }
    pub fn set_y1(&mut self, val: i32) { self.y1 = val }
    pub fn set_x2(&mut self, val: i32) { self.x2 = val }
    pub fn set_y2(&mut self, val: i32) { self.y2 = val }

    // like computed properties

    pub fn length(&self) -> i32 {
        self.y2 - self.y1
    }

    pub fn width(&self)  -> i32 {
        self.x2 - self.x1
    }

    pub fn area(&self)  -> i32 {
        self.length() * self.width()
    }
}
```

## 扩展方法

C#中的扩展方法使开发人员能够附加给现有类型新的静态绑定方法，无需修改类型的原始定义。
在下面的C#示例中，一个新的`Wrap`方法被添加给`StringBuilder`类 _通过扩展_：

```csharp
using System;
using System.Text;
using Extensions; // (1)

var sb = new StringBuilder("Hello, World!");
sb.Wrap(">>> ", " <<<"); // (2)
Console.WriteLine(sb.ToString()); // 打印： >>> Hello, World! <<<

namespace Extensions
{
    static class StringBuilderExtensions
    {
        public static void Wrap(this StringBuilder sb,
                                string left, string right) =>
            sb.Insert(0, left).Append(right);
    }
}
```

请注意，要使(2)扩展方法可用，包含类型扩展方法的命名空间必须导入(1)。Rust通过traits提供了非常相似的设施，称为 _扩展traits_。
下列Rust中的示例相当于上面的C#示例；它扩展了`String`使用`wrap`方法：

```rust
#![allow(dead_code)]

mod exts {
    pub trait StrWrapExt {
        fn wrap(&mut self, left: &str, right: &str);
    }

    impl StrWrapExt for String {
        fn wrap(&mut self, left: &str, right: &str) {
            self.insert_str(0, left);
            self.push_str(right);
        }
    }
}

fn main() {
    use exts::StrWrapExt as _; // (1)

    let mut s = String::from("Hello, World!");
    s.wrap(">>> ", " <<<"); // (2)
    println!("{s}"); // 打印：>>> Hello, World! <<<
}
```

就像在C#中一样，扩展trait中的方法使得（2）可用，必须导入扩展trait（1）。还要注意，扩展特性标识符`StrWrapExt` 本身可以在导入时通过`_`丢弃
而不影响`String`的`wrap` 的可用性。

## 可见性/访问修改器

C#有许多可访问性或可见性修饰符：

- `private`
- `protected`
- `internal`
- `protected internal` (family)
- `public`

在Rust中，编译是由模块树构建的，其中模块包含并定义 [_项目_][items]，如类型、特征、枚举、常量和函数。默认情况下，几乎所有内容都是私有的。
一个例外是，对于例如，公共trait中 _关联项目_，默认情况下是公共的。
这类似于C#接口的成员在没有任何公共声明方式，默认情况下源代码中的修饰符是公共的。Rust只有`pub`修改器来更改相对于模块树的可见性。
这里是`pub`的变体，可更改公众可见性的范围：

- `pub(self)`
- `pub(super)`
- `pub(crate)`
- `pub(in PATH)`

有关更多详细信息，请参阅Rust参考的[可见性和隐私][privis]部分

  [privis]: https://doc.rust-lang.org/reference/visibility-and-privacy.html
  [items]: https://doc.rust-lang.org/reference/items.html

下表是C#和Rust修饰符的映射近似值：

| C#                            | Rust         | 注释        |
| ----------------------------- | ------------ | ----------- |
| `private`                     | (default)    | 看注释 1. |
| `protected`                   | N/A          | 看注释 2. |
| `internal`                    | `pub(crate)` |             |
| `protected internal` (family) | N/A          | 看注释 2. |
| `public`                      | `pub`        |             |

1. 没有关键字表示私有可见性；这是Rust中的默认设置。

2. 由于 Rust 中没有基于类的类型层次结构，因此没有等同于`protected`。

## 可变性

在C#中设计类型时，开发人员有责任确定类型是可变的还是不可变的；是否支持破坏性或非破坏性可变。
C#确实支持不可变设计类型通过 _记录声明_（`record class`或`readonly record struct`）。
在Rust中可变性通过在类型方法上的`self`参数，如下例所示：

```rust
struct Point { x: i32, y: i32 }

impl Point {
    pub fn new(x: i32, y: i32) -> Self {
        Self { x, y }
    }

    // self是不可变的

    pub fn x(&self) -> i32 { self.x }
    pub fn y(&self) -> i32 { self.y }

    // self是可变的

    pub fn set_x(&mut self, val: i32) { self.x = val }
    pub fn set_y(&mut self, val: i32) { self.y = val }
}
```

在C#中，可以使用`with`进行非破坏性可变：

```c#
var pt = new Point(123, 456);
pt = pt with { X = 789 };
Console.WriteLine(pt.ToString()); // 打印： Point { X = 789, Y = 456 }

readonly record struct Point(int X, int Y);
```

Rust中没有`with`，但Rust模仿类似内容，它可以融入类型设计中：

```rust
struct Point { x: i32, y: i32 }

impl Point {
    pub fn new(x: i32, y: i32) -> Self {
        Self { x, y }
    }

    pub fn x(&self) -> i32 { self.x }
    pub fn y(&self) -> i32 { self.y }

    // 以下方法通过self并返回一个新实例

    pub fn set_x(self, val: i32) -> Self { Self::new(val, self.y) }
    pub fn set_y(self, val: i32) -> Self { Self::new(self.x, val) }
}
```

在C#中，`with`也可以与`struct`一起使用（而不是record），公开其读写字段：

```c#
struct Point
{
    public int X;
    public int Y;

    public override string ToString() => $"({X}, {Y})";
}

var pt = new Point { X = 123, Y = 456 };
Console.WriteLine(pt.ToString()); // 打印： (123, 456)
pt = pt with { X = 789 };
Console.WriteLine(pt.ToString()); // 打印： (789, 456)
```

Rust有一个 _[struct更新语法]_，可能看起来很相似：

```rust
mod points {
    #[derive(Debug)]
    pub struct Point { pub x: i32, pub y: i32 }
}

fn main() {
    use points::Point;
    let pt = Point { x: 123, y: 456 };
    println!("{pt:?}"); // 打印： Point { x: 123, y: 456 }
    let pt = Point { x: 789, ..pt };
    println!("{pt:?}"); // 打印： Point { x: 789, y: 456 }
}
```

但是，虽然C#中的`with`会进行非破坏性可变（复制然后更新），[struct更新语法]执行（部分）_移动_ 并仅限适用于字段。
由于语法需要访问类型的字段，因此它通常更常见的是在Rust模块中使用它，该模块可以访问其类型的私有详细信息。

  [struct更新语法]: https://doc.rust-lang.org/stable/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
