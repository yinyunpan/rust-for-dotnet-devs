# 继承

正如 [结构体][structures] 一节中所解释的，Rust不提供如C#中继承（基于类）。
在结构之间提供共享行为的一种方法是通过利用特征。
但是与C#中的 _接口继承_ 类似，Rust允许通过使用[_超级特征_][supertrait.rs]。

[structures]: ./custom-types/structs.md
[supertrait.rs]: https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-supertraits-to-require-one-traits-functionality-within-another-trait
