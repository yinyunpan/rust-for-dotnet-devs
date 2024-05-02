# 命名空间

命名空间在.NET中用于组织类型，以及用于控制项目中的类型和方法的范围。

在Rust中，命名空间指的是一个不同的概念。
等效的命名空间在Rust中是一个[模块][rust-module]。
对于C#和Rust，项目的可见性可以使用访问修饰符和可见性修饰符进行限制。
在Rust，默认可见性是 _private_（只有少数例外）。
C#的`public`在Rust中是`pub`，而`internal`对应于`pub(crate)`。
有关更细粒度的访问控制，请参阅 [可见性修饰符][visibility modifiers]。

[rust-module]: https://doc.rust-lang.org/reference/items/modules.html
[visibility modifiers]: https://doc.rust-lang.org/reference/visibility-and-privacy.html
