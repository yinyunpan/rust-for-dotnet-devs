# 文档注释

C#提供了一种使用注释语法为类型编写API文档的机制其中包含XML文本。
C#编译器生成一个XML文件，该文件包含表示评论和API签名的结构化数据。
其他工具可以处理该输出，以不同的方式提供人类可读的文档形式。
C#中的一个简单示例：

```csharp
/// <summary>
/// This is a document comment for <c>MyClass</c>.
/// </summary>
public class MyClass {}
```

在Rust[文档注释][doc comments]中提供了与C#文档注释等效的内容。
Rust中的文档注释使用Markdown语法。
[`rustdoc`][rustdoc]是Rust代码的文档编译器，通常通过调用[`cargo doc`][cargo doc]，它将注释编译成文档。
例如：

```rust
/// This is a doc comment for `MyStruct`.
struct MyStruct;
```

在中.NET SDK中没有与`cargo doc`等效，例如`dotnet doc`。

另请参阅:

- [如何编写文档][How to write documentation]
- [文档测试][Documentation tests]

[doc comments]: https://doc.rust-lang.org/rust-by-example/meta/doc.html
[rustdoc]: https://doc.rust-lang.org/rustdoc/index.html
[cargo doc]: https://doc.rust-lang.org/cargo/commands/cargo-doc.html
[How to write documentation]: https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html
[documentation tests]: https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html
