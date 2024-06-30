# 项目结构

虽然在.NET中有一些关于构建项目的惯例，但与Rust项目结构惯例相比，它们不那么严格。
当使用Visual Studio 2022（一个类库和一个xUnit测试项目）创建双项目解决方案时，它将创建以下结构：

    .
    |   SampleClassLibrary.sln
    +---SampleClassLibrary
    |       Class1.cs
    |       SampleClassLibrary.csproj
    +---SampleTestProject
            SampleTestProject.csproj
            UnitTest1.cs
            Usings.cs

- 每个项目都位于单独的目录中，并有自己的`.csproj`文件。
- 存储库的根目录下有一个`.sln`文件。

Cargo使用以下约定来定义[包布局][package layout]，以便轻松深入了解新的Cargo[包][rust-package]：

    .
    +-- Cargo.lock
    +-- Cargo.toml
    +-- src/
    |   +-- lib.rs
    |   +-- main.rs
    +-- benches/
    |   +-- some-bench.rs
    +-- examples/
    |   +-- some-example.rs
    +-- tests/
        +-- some-integration-test.rs

- `Cargo.toml`和`Cargo.lock`存储在包的根目录中。
- `src/lib.rs`是默认库文件，`src/main.rs`是默认可执行文件（参见[目标自动发现][target auto-discovery]）。
- 基准测试放在`benches`目录中，集成测试放在`tests`目录中（参见[测试][section-testing]、[基准测试][section-benchmarking]）。
- 示例放在`examples`目录中。
- 没有单独的单元测试包，单元测试与代码位于同一文件中（参见[测试][section-testing]）。

[package layout]: https://doc.rust-lang.org/cargo/guide/project-layout.html
[rust-package]: https://doc.rust-lang.org/cargo/appendix/glossary.html#package
[target auto-discovery]: https://doc.rust-lang.org/cargo/reference/cargo-targets.html#target-auto-discovery
[section-testing]: ../testing/index.md
[section-benchmarking]: ../benchmarking/index.md

## 管理大型项目

对于Rust中的大型项目，Cargo提供[工作区][cargo-workspaces]来组织项目。
工作区可以帮助管理同时开发的多个相关包。有些项目使用[_虚拟清单_][cargo-virtual-manifest]，尤其是在没有主包的情况下。

[cargo-workspaces]: https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html
[cargo-virtual-manifest]: https://doc.rust-lang.org/cargo/reference/workspaces.html#virtual-workspace

## 管理依赖项版本

在.NET中管理较大的项目时，使用诸如[中央包管理][Central Package Management]之类的策略来集中管理依赖项的版本可能是合适的。
Cargo引入了[工作区继承][workspace inheritance]来集中管理依赖项。

[Central Package Management]: https://learn.microsoft.com/en-us/nuget/consume-packages/Central-Package-Management
[workspace inheritance]: https://doc.rust-lang.org/cargo/reference/workspaces.html#the-package-table
