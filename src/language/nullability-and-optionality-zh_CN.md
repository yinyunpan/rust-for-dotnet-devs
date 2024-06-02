# 可空性和可选性

在C#中，`null`通常用于表示缺失、不存在或逻辑上未初始化的值。例如：

```csharp
int? some = 1;
int? none = null;
```

Rust没有`null`，因此没有可空上下文可启用。可选或缺失值则由[`Option<T>`][option]表示。
上述C#代码在Rust中的等效代码如下：

```rust
let some: Option<i32> = Some(1);
let none: Option<i32> = None;
```

Rust中的`Option<T>`实际上与F中的[`'T option`][opt.fs]相同。

[opt.fs]: https://fsharp.github.io/fsharp-core-docs/reference/fsharp-core-option-1.html

## 可选性的控制流程

在C#中，当使用可空值时，您可能一直使用`if`/`else`语句来控制流程。

```csharp
uint? max = 10;
if (max is { } someMax)
{
    Console.WriteLine($"The maximum is {someMax}."); // The maximum is 10.
}
```

您可以使用模式匹配在 Rust 中实现相同的行为：
使用`if let`甚至会更简洁：

```rust
let max = Some(10u32);
if let Some(max) = max {
    println!("The maximum is {}.", max); // The maximum is 10.
}
```

## 空条件运算符

The null-conditional operators (`?.` and `?[]`) make dealing with `null` in C#
more ergonomic. In Rust, they are best replaced by using the [`map`][optmap]
method. The following snippets show the correspondence:

空条件运算符（`?.`和`?[]`）使C#中处理`null`更加符合人体工程学。
在Rust中，最好使用[`map`][optmap]方法来替换它们。
以下代码片段显示了对应关系：

```csharp
string? some = "Hello, World!";
string? none = null;
Console.WriteLine(some?.Length); // 13
Console.WriteLine(none?.Length); // (blank)
```

```rust
let some: Option<String> = Some(String::from("Hello, World!"));
let none: Option<String> = None;
println!("{:?}", some.map(|s| s.len())); // Some(13)
println!("{:?}", none.map(|s| s.len())); // None
```

## 空合并运算符

当可空值为`null`时，通常使用空合并运算符（`??`）默认为另一个值：

```csharp
int? some = 1;
int? none = null;
Console.WriteLine(some ?? 0); // 1
Console.WriteLine(none ?? 0); // 0
```

在Rust中，你可以使用 [`unwrap_or`][unwrap-or]来获得相同的行为：

```rust
let some: Option<i32> = Some(1);
let none: Option<i32> = None;
println!("{:?}", some.unwrap_or(0)); // 1
println!("{:?}", none.unwrap_or(0)); // 0
```

**注意**：如果默认值的计算成本很高，您可以改用`unwrap_or_else`。
它以闭包作为参数，允许您延迟初始化默认值。

## 空值包容运算符

空值包容运算符(`!`) 在Rust中没有对应等效构造，因为它仅影响C#中编译器的静态流分析。、
在Rust中，无需使用它的替代品。

[option]: https://doc.rust-lang.org/std/option/enum.Option.html
[optmap]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map
[unwrap-or]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or
