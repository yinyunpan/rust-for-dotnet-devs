# 字符串

Rust中有两种字符串类型：`String` and `&str`。前者分配在堆上和，后者是`String`或者`&str`切片。

这些.NET映射显示在下面表格中:

| Rust               | .NET                 | 注释         |
| ------------------ | -------------------- | ----------- |
| `&mut str`         | `Span<char>`         |             |
| `&str`             | `ReadOnlySpan<char>` |             |
| `Box<str>`         | `String`             | 看注释1      |
| `String`           | `String`             |             |
| `String` (mutable) | `StringBuilder`      | 看注释1      |

Rust和.NET使用字符串有些不同，上面对应是一个很好的起点。一个不同点Rust字符串UTF-8是编码, 但.NET字符型是UTF-16编码。
此外.NET字符串是不可变的, 但Rust字符串可以是可变的当如此声明，例如`let s = &mut String::from("hello");`。

由于所有权的概念，在使用字符串方面也存在不同。有关字符串类型所有权的更多信息，请参阅[Rust书][ownership-string-type-example].

[ownership-string-type-example]: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-string-type

注释：

1. Rust中`Box<str>`类型对应.NET中`String`类型。Rust中`Box<str>`和`String`类型不同点是前者
  存储指针、大小而后者存储指针、大小、容量，`String`允许大小增长。Rust中`String`声明可变，.NET类似是`StringBuilder`类型.

C#:

```csharp
ReadOnlySpan<char> span = "Hello, World!";
string str = "Hello, World!";
StringBuilder sb = new StringBuilder("Hello, World!");
```

Rust:

```rust
let span: &str = "Hello, World!";
let str = Box::new("Hello World!");
let mut sb = String::from("Hello World!");
```

## 字符串字面值

.NET字符串字面值`String`类型是分配在堆上不可变的。Rust它是`&'static str`, 不可变的具有全局生命周期、不能分配在堆上；嵌入在编译的二进制文件中。

C#

```csharp
string str = "Hello, World!";
```

Rust

```rust
let str: &'static str = "Hello, World!";
```

C#原本字符串字面值对应Rust原始字符串字面值

C#

```csharp
string str = @"Hello, \World/!";
```

Rust

```rust
let str = r#"Hello, \World/!"#;
```

C#UTF-8字符串字面值对应Rust字节字符串字面值。

C#

```csharp
ReadOnlySpan<byte> str = "hello"u8;
```

Rust

```rust
let str = b"hello";
```

## 字符串插值

C#有内嵌的字符串插值特性，允许嵌入表达式在字符串字面值中。下面例子展示在C#怎么用字符串插值：

```csharp
string name = "John";
int age = 42;
string str = $"Person {{ Name: {name}, Age: {age} }}";
```

Rust没有内嵌的字符串插值特性。替代用`format!`宏格式化字符串。面例子展示在Rust怎么用字符串插值：

```rust
let name = "John";
let age = 42;
let str = format!("Person {{ name: {name}, age: {age} }}");
```

在C#中自定义类和结构也可以用插值，因为每种类型可用`ToString()`方法，因为它继承自`object`。

```csharp
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    public override string ToString() =>
        $"Person {{ Name: {Name}, Age: {Age} }}";
}

var person = new Person { Name = "John", Age = 42 };
Console.Writeline(person);
```

在Rust中每种类型的实现/继承没有默认格式化。替代必须为每个类型实现`std::fmt::Display`trait，来转换字符串。

```rust
use std::fmt::*;

struct Person {
    name: String,
    age: i32,
}

impl Display for Person {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        write!(f, "Person {{ name: {}, age: {} }}", self.name, self.age)
    }
}

let person = Person {
    name: "John".to_owned(),
    age: 42,
};

println!("{person}");
```

另一个选项是使用`std::fmt::Debug` trait。所有标准类型实现`Debug` trait，可用于打印类型内部形式。
以下示例展示了如何使用`derive`属性打印自定义结构的内部形式，使用`Debug`宏。此声明为`Person`结构体
自动实现`Debug`trait：

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: i32,
}

let person = Person {
    name: "John".to_owned(),
    age: 42,
};

println!("{person:?}");
```

> 注释：使用:? 格式化说明符将使用`Debug` trait打印这结构体，省略它将使用`Display` trait。

另请参阅：

- [Rust例子-调试](https://doc.rust-lang.org/stable/rust-by-example/hello/print/print_debug.html?highlight=derive#debug)
