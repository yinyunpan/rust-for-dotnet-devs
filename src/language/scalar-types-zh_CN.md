# 标量类型

下面表格列出Rust中基本类型及其C#和.NET中的等效类型：

| Rust    | C#        | .NET                   | 注释              |
| ------- | --------- | ---------------------- | ---------------- |
| `bool`  | `bool`    | `Boolean`              |                  |
| `char`  | `char`    | `Char`                 | 看注释1           |
| `i8`    | `sbyte`   | `SByte`                |                  |
| `i16`   | `short`   | `Int16`                |                  |
| `i32`   | `int`     | `Int32`                |                  |
| `i64`   | `long`    | `Int64`                |                  |
| `i128`  |           | `Int128`               |                  |
| `isize` | `nint`    | `IntPtr`               |                  |
| `u8`    | `byte`    | `Byte`                 |                  |
| `u16`   | `ushort`  | `UInt16`               |                  |
| `u32`   | `uint`    | `UInt32`               |                  |
| `u64`   | `ulong`   | `UInt64`               |                  |
| `u128`  |           | `UInt128`              |                  |
| `usize` | `nuint`   | `UIntPtr`              |                  |
| `f32`   | `float`   | `Single`               |                  |
| `f64`   | `double`  | `Double`               |                  |
|         | `decimal` | `Decimal`              |                  |
| `()`    | `void`    | `Void` or `ValueTuple` | 看注释2和3        |
|         | `object`  | `Object`               | 看注释3           |

注释：

1. [`char`][char.rs]在Rust和[`Char`][char.net]在.NET有不同定义。Rust中一个`char`是4字节宽度 [Unicode标量值]，但.NET中，一个`Char`是2字节宽度存储UTF-16编码的字符。 更多信息，请看[Rust `char`
   文档][char.rs]。

2. Rust中单元`()` (空元组) 是 _表达式值_，C#中最接近是`void`代表什么都不是。`void`
   不是 _表达式值_ 不能用于指针和不安全代码。.NET中[`ValueTuple`][ValueTuple]是一个空元组，但C#没有像`()`字面语法来表示它。 `ValueTuple`可以在C#中使用，但这非常罕见。不像C#像Rust，[F#有单元类型][unit.fs]。

3. `void`和`object`不是标量类型(标量类型如`int`在.NET类型结构中是`object`的子类型)，为方便理解它们包含在上面表格中。

另请参阅：

- [基本类型(Rust示例)][primitives.rs]

[char.net]: https://learn.microsoft.com/en-us/dotnet/api/system.char
[char.rs]: https://doc.rust-lang.org/std/primitive.char.html
[Unicode标量值]: https://www.unicode.org/glossary/#unicode_scalar_value
[ValueTuple]: https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple?view=net-7.0
[unit.fs]: https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/unit-type
[primitives.rs]: https://doc.rust-lang.org/rust-by-example/primitives.html
