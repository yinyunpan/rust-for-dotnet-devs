# 编译和构建

## .NET CLI

Rust中.NET CLI（`dotnet`）的对应工具是[Cargo]（`cargo`）。
这两种工具都是入口点包装器，可简化其他低级工具的使用。
例如，尽管您可以直接调用C#编译器（`csc`）或通过`dotnet msbuild`调用 MSBuild，但开发人员倾向于使用`dotnet build`来构建他们的解决方案。
同样，在Rust中，虽然您可以直接使用Rust编译器（`rustc`），但使用`cargo build`通常要简单得多。

[cargo]: https://doc.rust-lang.org/cargo/

## 构建

使用[`dotnet build`][net-build-output]在.NET中构建可执行文件会恢复包，将项目源编译为[程序集]。
程序集包含中间语言(IL)中的代码，通常可以在.NET支持的任何平台上运行，前提是主机上安装了.NET运行时。
来自依赖包的程序集通常与项目的输出程序集位于同一位置。
Rust中的[`cargo build`][cargo-build]执行相同的操作，只是Rust编译器将所有代码静态链接（尽管存在其他[链接选项][linkage]）为单个平台相关的二进制文件。

开发人员使用`dotnet publish`准备.NET可执行文件以供分发，无论是作为 _框架相关部署_(FDD)还是 _自包含部署_(SCD)。
在Rust中，没有与`dotnet publish`等同的功能，因为构建输出已经包含每个目标的单个、依赖于平台的二进制文件。

使用[`dotnet build`][net-build-output]在.NET中构建库时，它仍将生成包含IL的[程序集][assembly]。
在Rust中，构建输出同样是每个库目标的依赖于平台的已编译库。

另请参阅：

- [Crate]

[net-build-output]: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-build#description
[assembly]: https://learn.microsoft.com/en-us/dotnet/standard/assembly/
[cargo-build]: https://doc.rust-lang.org/cargo/commands/cargo-build.html#cargo-build1
[linkage]: https://doc.rust-lang.org/reference/linkage.html
[crate]: https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html

## 依赖

在.NET中，项目文件的内容定义构建选项和依赖项。
在Rust中，当使用Cargo时，`Cargo.toml`会声明包的依赖项。
典型的项目文件如下所示：

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="morelinq" Version="3.3.2" />
  </ItemGroup>

</Project>
```

Rust中等效的`Cargo.toml`定义为：

```toml
[package]
name = "hello_world"
version = "0.1.0"

[dependencies]
tokio = "1.0.0"
```

Cargo遵循一个约定，即`src/main.rs`是与包同名的二进制包的包根。
同样，Cargo知道，如果包目录包含`src/lib.rs`，则包包含与包同名的库包。

## 包

NuGet最常用于安装软件包，各种工具都支持它。
例如，使用.NET CLI添加NuGet软件包引用将向项目文件添加依赖项：

  dotnet add package morelinq

在Rust中，如果使用Cargo添加软件包，其工作原理几乎相同。

  cargo add tokio

.NET最常见的软件包注册是[nuget.org]，Rust软件包通常通过[crates.io]共享。

[nuget.org]: https://www.nuget.org/
[crates.io]: https://crates.io

## 静态代码分析

从.NET 5开始，Roslyn分析器与.NET SDK捆绑在一起，提供代码质量和代码样式分析。Rust中等效的linting工具是[Clippy]。

与.NET类似，如果通过将[`TreatWarningsAsErrors`][treat-warnings-as-errors]设置为`true`出现警告，构建就会失败，如果编译器或Clippy发出警告（`cargo clippy -- -D warnings`），Clippy也会失败。

还有进一步的静态检查可以考虑添加到Rust CI管道中：

- 运行[`cargo doc`][cargo-doc]以确保文档正确。
- 运行[`cargo check --locked`][cargo-check]以强制`Cargo.lock`文件是最新的。

[clippy]: https://github.com/rust-lang/rust-clippy
[treat-warnings-as-errors]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/errors-warnings
[cargo-doc]: https://doc.rust-lang.org/cargo/commands/cargo-doc.html
[cargo-check]: https://doc.rust-lang.org/cargo/commands/cargo-check.html#manifest-options
