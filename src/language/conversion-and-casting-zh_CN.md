# 转换

C#和Rust在编译时都是静态类型的。
因此，在声明变量后，禁止将不同类型的值（除非它可以隐式转换为目标类型）赋给变量。
在C#中有几种方法可以转换，Rust中具有等效类型的类型。

## 隐式转换

C#隐式转换，也存在Rust中（称为[类型强制][type coercions]）。
请看以下示例：

```csharp
int intNumber = 1;
long longNumber = intNumber;
```

Rust对于允许的类型强制有更严格的限制：

```rust
let int_number: i32 = 1;
let long_number: i64 = int_number; // error: expected `i64`, found `i32`
```

使用 [子类型][subtyping.rs] 进行有效隐式转换的示例为：

```rust
fn bar<'a>() {
    let s: &'static str = "hi";
    let t: &'a str = s;
}
```

另请参考：

- [解除强制][Deref coercion]
- [子类型和变异][Subtyping and variance]

[type coercions]: https://doc.rust-lang.org/reference/type-coercions.html
[subtyping.rs]: https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md#subtyping
[deref coercion]: https://doc.rust-lang.org/std/ops/trait.Deref.html#more-on-deref-coercion
[Subtyping and variance]: https://doc.rust-lang.org/reference/subtyping.html#subtyping-and-variance

## 显式转换

如果转换可能导致信息丢失，则C#需要使用强制转换表达式进行显式转换：

```csharp
double a = 1.2;
int b = (int)a;
```

显式转换可能会在运行时失败，并在 _向下转换_ 时出现诸如`OverflowException`或`InvalidCastException`之类的异常。

Rust不提供基元类型之间的强制，而是使用[显示转换][casting.rs]通过[`as`][as.rs]关键字（转换）。
在Rust中转换不会引起恐慌。

```rust
let int_number: i32 = 1;
let long_number: i64 = int_number as _;
```

[casting.rs]: https://doc.rust-lang.org/rust-by-example/types/cast.html
[as.rs]: https://doc.rust-lang.org/reference/expressions/operator-expr.html#type-cast-expressions

## 自定义转换

通常，.NET类型提供用户定义的转换运算符来将一种类型转换为另一种类型。
此外，`System.IConvertible` 可用于将一种类型转换为另一种类型。

在Rust中，标准库包含一个抽象，用于将值转换为不同类型的值，形式为[`From`][from.rs] 特征及其倒数[`Into`][into.rs]。
在为类型实现`From` 时，会自动提供`Into`的默认实现（在Rust中称为 _全面实现_）。
以下示例说明了两种此类类型转换：

```rust
fn main() {
    let my_id = MyId("id".into()); // 由于`String`的`From<&str>`特征实现，`into()`是自动实现的。
    println!("{}", String::from(my_id)); // 这使用`From<MyId>`实现`String`。
}

struct MyId(String);

impl From<MyId> for String {
    fn from(MyId(value): MyId) -> Self {
        value
    }
}
```

另请参阅：

- [`TryFrom`][try-from.rs]和[`TryInto`][try-into.rs]针对可能失败的`From`和`Into`版本​​。

[from.rs]: https://doc.rust-lang.org/std/convert/trait.From.html
[into.rs]: https://doc.rust-lang.org/std/convert/trait.Into.html
[try-from.rs]: https://doc.rust-lang.org/std/convert/trait.TryFrom.html
[try-into.rs]: https://doc.rust-lang.org/std/convert/trait.TryInto.html
