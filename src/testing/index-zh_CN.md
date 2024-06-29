# 测试

## 测试组织

.NET解决方案使用单独的项目来托管测试代码，无论使用的测试框架（xUnit、NUnit、MSTest 等）和编写的测试类型（单元或集成）如何。
因此，测试代码位于与被测试的应用程序或库代码不同的程序集中。
在Rust中，更常见的是  _单元测试_ 位于单独的测试子模块中（传统上）名为`tests`，但该子模块与测试主题的应用程序或库模块代码位于同一 _源文件_ 中。
这有两个好处：

- 代码/模块及其单元测试并存。

- 不需要像.NET中存在的`[InternalsVisibleTo]`这样的解决方法，因为测试可以通过虚拟子模块访问内部内容。

测试子模块使用`#[cfg(test)]`属性进行注释，其效果是，只有发出`cargo test`命令时，才会（有条件地）编译和运行整个模块。

在测试子模块中，测试函数使用`#[test]`属性进行注释。

集成测试通常位于名为`tests`的目录中，该目录与包含单元测试和源代码的`src`目录相邻。`cargo test`将该目录中的每个文件编译为单独的包，并运行使用`#[test]`属性注释的所有方法。
由于可以理解集成测试在`tests`目录中，因此无需使用`#[cfg(test)]`属性标记其中的模块。

另请参阅：

- [测试组织][test-org]

  [test-org]: https://doc.rust-lang.org/book/ch11-03-test-organization.html

## 运行测试

简单来说，Rust 中 `dotnet test` 的对应物是 `cargo test`。

`cargo test` 的默认行为是并行运行所有测试，但可以将其配置为仅使用单个线程连续运行：

    cargo test -- --test-threads=1

有关详细信息，请参阅“[并行或连续运行测试][tests-exec]”。

  [tests-exec]: https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-tests-in-parallel-or-consecutively

## 测试输出

对于非常复杂的集成或端到端测试，.NET 开发人员有时会记录测试期间发生的情况。
他们实际执行此操作的方式因每个测试框架而异。
例如，在NUnit中，这就像使用`Console.WriteLine`一样简单，但在 XUnit 中，人们使用`ITestOutputHelper`。
在Rust中，它类似于NUnit；也就是说，人们只需使用`println！`写入标准输出。
除非使用`--show-output`选项运行`cargo test`，否则默认情况下不会显示测试运行期间捕获的输出：

    cargo test --show-output

有关详细信息，请参阅“[显示函数输出][test-output]”。

  [test-output]: https://doc.rust-lang.org/book/ch11-02-running-tests.html#showing-function-output

## 断言

.NET用户有多种断言方式，具体取决于所使用的框架。例如，xUnit.net 的断言可能如下所示：

```csharp
[Fact]
public void Something_Is_The_Right_Length()
{
    var value = "something";
    Assert.Equal(9, value.Length);
}
```

Rust不需要单独的框架或板条箱。标准库附带内置的 _宏_，足以满足测试中的大多数断言：

- [`assert!`][assert]
- [`assert_eq!`][assert_eq]
- [`assert_ne!`][assert_ne]

下面是`assert_eq`实际运行的一个示例：

```rust
#[test]
fn something_is_the_right_length() {
    let value = "something";
    assert_eq!(9, value.len());
}
```

标准库没有提供任何数据驱动测试方向的内容，例如xUnit.net中的`[Theory]`。

  [assert]: https://doc.rust-lang.org/std/macro.assert.html
  [assert_eq]: https://doc.rust-lang.org/std/macro.assert_eq.html
  [assert_ne]: https://doc.rust-lang.org/std/macro.assert_ne.html

## 模拟

When writing tests for a .NET application or library, there exist several
frameworks, like Moq and NSubstitute, to mock out the dependencies of types.
There are similar crates for Rust too, like [`mockall`][mockall], that can
help with mocking. However, it is also possible to use [conditional
compilation] by making use of the [`cfg` attribute][cfg-attribute] as a simple
means to mocking without needing to rely on external crates or frameworks. The
`cfg` attribute conditionally includes the code it annotates based on a
configuration symbol, such as `test` for testing. This is not very different
to using `DEBUG` to conditionally compile code specifically for debug builds.
One downside of this approach is that you can only have one implementation for
all tests of the module.

When specified, the `#[cfg(test)]` attribute tells Rust to compile and run the
code only when executing the `cargo test` command, which behind-the-scenes
executes the compiler with `rustc --test`. The opposite is true for the
`#[cfg(not(test))]` attribute; it includes the annotated only when testing
with `cargo test`.

The example below shows mocking of a stand-alone function `var_os` from the
standard that reads and returns the value of an environment variable. It
conditionally imports a mocked version of the `var_os` function used by
`get_env`. When built with `cargo build` or run with `cargo run`, the compiled
binary will make use of `std::env::var_os`, but `cargo test` will instead
import `tests::var_os_mock` as `var_os`, thus causing `get_env` to use the
mocked version during testing:

在为.NET应用程序或库编写测试时，存在多个框架（如 Moq 和 NSubstitute）来模拟类型的依赖关系。
Rust也有类似的包，如 [`mockall`][mockall]，可以帮助模拟。
但是，也可以通过使用 [`cfg`属性][cfg-attribute] 作为简单的模拟方法，而无需依赖外部包或框架，从而使用[条件编译]。
`cfg`属性根据配置符号（例如用于测试的`test`）有条件地包含它注释的代码。
这与使用`DEBUG`有条件地编译专门用于调试版本的代码没有太大区别。
这种方法的一个缺点是，您只能为模块的所有测试提供一个实现。

指定后，`#[cfg(test)]`属性会告诉Rust仅在执行`cargo test`命令时编译并运行代码，该命令在后台使用`rustc --test`执行编译器。
`#[cfg(not(test))]`属性则相反；它仅在使用`cargo test`进行测试时才包含注释。

以下示例显示了标准中独立函数`var_os`的模拟，该函数读取并返回环境变量的值。
它有条件地导入`get_env`使用的`var_os`函数的模拟版本。当使用`cargo build`构建或使用`cargo run`运行时，编译后的二进制文件将使用`std::env::var_os`，
但`cargo test` 将改为导入`tests::var_os_mock`作为`var_os`，从而导致`get_env`在测试期间使用模拟版本：

```rust
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT license.

/// Utility function to read an environmentvariable and return its value If
/// defined. It fails/panics if the valus is not valid Unicode.
pub fn get_env(key: &str) -> Option<String> {
    #[cfg(not(test))]                 // for regular builds...
    use std::env::var_os;             // ...import from the standard library
    #[cfg(test)]                      // for test builds...
    use tests::var_os_mock as var_os; // ...import mock from test sub-module

    let val = var_os(key);
    val.map(|s| s.to_str()     // get string slice
                 .unwrap()     // panic if not valid Unicode
                 .to_owned())  // convert to "String"
}

#[cfg(test)]
mod tests {
    use std::ffi::*;
    use super::*;

    pub(crate) fn var_os_mock(key: &str) -> Option<OsString> {
        match key {
            "FOO" => Some("BAR".into()),
            _ => None
        }
    }

    #[test]
    fn get_env_when_var_undefined_returns_none() {
        assert_eq!(None, get_env("???"));
    }

    #[test]
    fn get_env_when_var_defined_returns_some_value() {
        assert_eq!(Some("BAR".to_owned()), get_env("FOO"));
    }
}
```

  [mockall]: https://docs.rs/mockall/latest/mockall/
  [conditional compilation]: ../conditional-compilation/index.md
  [cfg-attribute]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-attribute

## 代码覆盖率

在分析测试代码覆盖率时，.N​​ET有复杂的工具。
在Visual Studio中，工具是内置和集成的。
在Visual Studio Code中，存在插件。.NET 开发人员可能也熟悉[coverlet]。

Rust提供[内置代码覆盖率实现][built-in-cov]来收集测试代码覆盖率。

Rust也有插件可帮助进行代码覆盖率分析。
它不是无缝集成的，但通过一些手动步骤，开发人员可以以可视化的方式分析他们的代码。

Visual Studio Code的[Coverage Gutters][coverage.gutters]插件和[Tarpaulin]的组合允许在Visual Studio Code中可视化分析代码覆盖率。
Coverage Gutters需要LCOV文件。除了[Tarpaulin] 之外，其他工具也可用于生成该文件。

设置后，运行以下命令：

```bash
cargo tarpaulin --ignore-tests --out Lcov
```

这将生成一个 LCOV 代码覆盖率文件。
一旦启用`Coverage Gutters: Watch`，它将被Coverage Gutters插件拾取，该插件将在源代码编辑器中显示有关行覆盖率的内联视觉指示器。

> 注意：LCOV 文件的位置至关重要。
如果存在包含多个包的工作区（请参阅[项目结构][project structure]），并且使用 `--workspace` 在根目录中生成了 LCOV 文件，则该文件就是正在使用的文件-即使包的根目录中直接存在一个文件。
与在根目录中生成 LCOV 文件相比，将文件隔离到正在测试的特定包会更快。

[coverage.gutters]: https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters
[tarpaulin]: https://github.com/xd009642/tarpaulin
[coverlet]: https://github.com/coverlet-coverage/coverlet
[built-in-cov]: https://doc.rust-lang.org/stable/rustc/instrument-coverage.html#test-coverage
[project structure]: ../project-structure/index.md
