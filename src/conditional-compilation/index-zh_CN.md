# 条件编译

.NET和Rust都提供了根据外部条件编译特定代码的可能性。

在.NET中，可以使用一些[预处理器指令][preproc-dir]来控制条件编译

```csharp
#if debug
    Console.WriteLine("Debug");
#else
    Console.WriteLine("Not debug");
#endif
```

除了预定义符号外，还可以使用编译器选项 _[定义常量][DefineConstants]_ 来定义可与`#if`、`#else`、`#elif`和`#endif`一起使用的符号，以有条件地编译源文件。

在Rust中，可以使用[`cfg属性`][cfg]、[`cfg_attr属性`][cfg-attr] 或[`cfg宏`][cfg-macro]来控制条件编译。

根据.NET，除了预定义符号外，还可以使用[编译器标志`--cfg`][cfg-flag] 任意设置配置选项

[`cfg属性`][cfg]需要并评估`配置谓词`

```rust
use std::fmt::{Display, Formatter};

struct MyStruct;

// This implementation of Display is only included when the OS is unix but foo is not equal to bar
// You can compile an executable for this version, on linux, with 'rustc main.rs --cfg foo=\"baz\"'
#[cfg(all(unix, not(foo = "bar")))]
impl Display for MyStruct {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        f.write_str("Running without foo=bar configuration")
    }
}

// This function is only included when both unix and foo=bar are defined
// You can compile an executable for this version, on linux, with 'rustc main.rs --cfg foo=\"bar\"'
#[cfg(all(unix, foo = "bar"))]
impl Display for MyStruct {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        f.write_str("Running with foo=bar configuration")
    }
}

// This function is panicking when not compiled for unix
// You can compile an executable for this version, on windows, with 'rustc main.rs'
#[cfg(not(unix))]
impl Display for MyStruct {
    fn fmt(&self, _f: &mut Formatter<'_>) -> std::fmt::Result {
        panic!()
    }
}

fn main() {
    println!("{}", MyStruct);
}
```

[`cfg_attr属性`][cfg-attr]根据配置谓词有条件地包含属性。

```rust
#[cfg_attr(feature = "serialization_support", derive(Serialize, Deserialize))]
pub struct MaybeSerializableStruct;

// When the `serialization_support` feature flag is enabled, the above will expand to:
// #[derive(Serialize, Deserialize)]
// pub struct MaybeSerializableStruct;
```

内置的 [`cfg宏`][cfg-macro]接受单个配置谓词，并在谓词为真时计算为真文字，为假时计算为假文字。

```rust
if cfg!(unix) {
  println!("I'm running on a unix machine!");
}
```

另请参阅：

- [条件编译][conditional-compilation]

## 特征

当需要提供可选依赖项时，条件编译也很有用。
使用Cargo“功能”，包在Cargo.toml的`[features]`表中定义一组命名的功能，每个功能都可以启用或禁用。
可以使用`--features`等标志在命令行上启用正在构建的包的功能。可以在Cargo.toml中的依赖项声明中启用依赖项的功能。

另请参阅：

- [功能][features]

[features]: https://doc.rust-lang.org/cargo/reference/features.html
[conditional-compilation]: https://doc.rust-lang.org/reference/conditional-compilation.html#conditional-compilation
[cfg]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-attribute
[cfg-flag]: https://doc.rust-lang.org/rustc/command-line-arguments.html#--cfg-configure-the-compilation-environment
[cfg-attr]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg_attr-attribute
[cfg-macro]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-macro
[preproc-dir]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives#conditional-compilation
[DefineConstants]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/language#defineconstants
