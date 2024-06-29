# 元编程

元编程可以看作是一种编写代码来编写/生成其他代码的方式。

Roslyn为C#中的元编程提供了一项功能，该功能自.NET 5开始提供，称为[`Source Generators`][source-gen]。
源生成器可以在构建时创建新的C#源文件，并将其添加到用户的编译中。
在引入`Source Generators`之前，Visual Studio一直通过[`T4 Text Templates`][T4]提供代码生成工具。
T4工作原理的一个示例是以下[模板]或其[具体化]。

Rust还提供了元编程功能：[宏]。有`声明性宏`和`过程宏`。

声明性宏允许您编写控制结构，该控制结构采用表达式，将表达式的结果值与模式进行比较，然后运行与匹配模式相关的代码。

以下示例是`println!`宏的定义，可以调用该宏来打印一些文本 `println!("Some text")`

```rust
macro_rules! println {
    () => {
        $crate::print!("\n")
    };
    ($($arg:tt)*) => {{
        $crate::io::_print($crate::format_args_nl!($($arg)*));
    }};
}
```

要了解有关编写声明性宏的更多信息，请参阅Rust参考章节[宏示例][macros by example]或[Rust宏小册子][The Little Book of Rust Macros]。

[过程宏]与声明性宏不同。它们接受一些代码作为输入，对该代码进行操作，并生成一些代码作为输出。

C#中用于元编程的另一种技术是反射。Rust不支持反射。

[source-gen]: https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview

## 类似函数的宏

函数式宏的形式如下：`function!(...)`

以下代码片段定义了一个名为`print_something`的函数式宏，它生成一个`print_it`方法来打印“Something”字符串。

在lib.rs中：

```rust
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro]
pub fn print_something(_item: TokenStream) -> TokenStream {
    "fn print_it() { println!(\"Something\") }".parse().unwrap()
}
```

在main.rs中：

```rust
use replace_crate_name_here::print_something;
print_something!();

fn main() {
    print_it();
}
```

## 派生宏

派生宏可以根据结构、枚举或联合的标记流创建新项。
派生宏的一个示例是`#[derive(Clone)]`，它生成使输入结构/枚举/联合实现`Clone`特征所需的代码。

为了了解如何定义自定义派生宏，可以阅读[派生宏][derive macros]的rust参考

[derive macros]: https://doc.rust-lang.org/reference/procedural-macros.html#derive-macros

## 属性宏

属性宏定义新属性可以附加到rust项目。
在使用异步代码时，如果使用Tokio，第一步将是用属性宏装饰新的异步主程序，如以下示例所示：

```rust
#[tokio::main]
async fn main() {
    println!("Hello world");
}
```

为了理解如何定义自定义属性宏，可以阅读rust参考中的[属性宏][attribute macros]

[attribute macros]: https://doc.rust-lang.org/reference/procedural-macros.html#attribute-macros

[T4]: https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2015/modeling/code-generation-and-t4-text-templates?view=vs-2015&redirectedfrom=MSDN
[template]: https://github.com/atifaziz/Jacob/blob/master/src/JsonReader.g.tt
[concretization]: https://github.com/atifaziz/Jacob/blob/master/src/JsonReader.g.cs
[macros]: https://doc.rust-lang.org/book/ch19-06-macros.html
[macros by example]: https://doc.rust-lang.org/reference/macros-by-example.html
[procedural macros]: https://doc.rust-lang.org/reference/procedural-macros.html
[The Little Book of Rust Macros]: https://veykril.github.io/tlborm/
