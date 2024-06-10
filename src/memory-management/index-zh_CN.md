# 内存管理

像C#和.NET 一样，Rust 也 _内存安全_ 来避免与内存访问有关的类错误，最终成为许多安全的来源软件中的漏洞。
但是，Rust可以保证内存安全编译时；没有运行时（如CLR）进行检查。
一个例外情况是数组绑定检查，这些检查是由编译后的代码完成的在运行时，无论是Rust编译器还是.NET中的JIT 编译器。
像C#一样，它也是[可以在Rust中编写不安全的代码][unsafe-rust]，事实上，两种语言甚至共享相同的关键字，_字面意思_`unsafe`，
以标记不再保证内存安全的函数和代码块。

  [unsafe-rust]: https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html

Rust没有垃圾收集器（GC）。
所有内存管理完全是开发者的责任。
也就是说，_Rust安全_ 有相关所有权规则确保内存在不再使用时立即释放（例如当离开块或函数的范围时）。
编译器执行通过（编译时）静态分析，帮助管理这一点，这是一项艰巨的工作通过[所有权]规则。
如果违反，编译器将拒绝代码出现编译错误。

  [ownership]: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html

在.NET里面，除了GC根（静态字段、线程堆栈上的局部变量、CPU寄存器、句柄等）之外，没有内存所有权的概念。
GC在收集过程中从根开始确定通过以下引用并清除其余引用，所有正在使用的内存。
设计类型和编写代码时，.NET开发人员可以不在意所有权、内存管理、甚至垃圾收集器在大多数情况下是如何工作的，
除非对性能敏感的代码需要注意数量以及在堆上分配对象的速率。
相比之下，Rust的所有权规则要求开发者明确思考和表达所有权在任何时候都会影响到从功能设计、类型、数据结构以及代码的编写方式。
除此之外，Rust关于如何使用数据的严格规则，使得它可以在编译时识别，
数据[竞态条件][race conditions]以及损坏问题（需要线程安全）这可能在运行时发生。
本节仅关注内存管理和所有权。

  [race conditions]: https://doc.rust-lang.org/nomicon/races.html

某些内存只能有一个所有者，无论是在堆栈上还是在堆上，在Rust中的任何给定时间支持一个结构。
编译器分配[生命周期][lifetimes.rs]并跟踪所有权。可以传递或让出所有权，在Rust中称为 _转移_。
这些想法简要地在下面的Rust代码示例中进行了说明：

  [lifetimes.rs]: https://doc.rust-lang.org/rust-by-example/scope/lifetime.html

```rust
#![allow(dead_code, unused_variables)]

struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let a = Point { x: 12, y: 34 }; // a有point所有权
    let b = a;                      // b现在有point所有权
    println!("{}, {}", a.x, a.y);   // 编译错误！
}
```

`main` 中的第一个语句将分配 `Point`，并且该内存将归 `a` 所有。
在第二个语句中，所有权从 `a` 转移到 `b`，并且 `a` 不再可用，因为它不再拥有任何东西或代表有效内存。
最后一个尝试通过 `a` 打印 point 字段的语句将编译失败。假设 `main` 已修复为如下所示：

```rust
fn main() {
    let a = Point { x: 12, y: 34 }; // a有point所有权
    let b = a;                      // b现在有point所有权
    println!("{}, {}", b.x, b.y);   // 用b，没问题
}   // b之后point被丢弃
```

请注意，当`main`退出时，`a`和`b`将超出范围。
由于堆栈返回到调用`main`之前的状态，`b`后面的内存将被释放。
在Rust中，人们说`b`后面的point被_丢弃_。
但是，请注意，由于`a`将其对该point的所有权让给了`b`，因此当`a`超出范围时，没有什么可丢弃的。

Rust中的 `struct` 可以通过实现 [`Drop`][drop.rs] 特征来定义在丢弃实例时执行的代码。

  [drop.rs]: https://doc.rust-lang.org/std/ops/trait.Drop.html

C#中 _dropping_ 的大致对应物是类 [finalizer]，但虽然GC会在未来的某个时间点 _自动_ 调用终结器，
但Rust中的dropping始终是即时且确定的；也就是说，它发生在编译器根据范围和生命周期确定实例没有所有者时。
在.NET中，`Drop` 的对应物是[`IDisposable`][IDisposable]，并由类型实现以释放它们持有的任何非托管资源或内存。
_确定性处置_ 不是强制执行或保证的，但C#中的`using`语句通常用于确定一次性类型的实例，以便它在`using`语句块的末尾被确定地处置。

  [finalizer]: https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers
  [IDisposable]: https://learn.microsoft.com/en-us/dotnet/api/system.idisposable

Rust具有全局生命周期的概念，用`'static`表示，这是一个保留的生命周期说明符。
C#中一个非常粗略的近似值是静态的 _只读_ 类型的字段。

在C#和.NET 中，引用可以自由共享而无需过多考虑，
因此单一所有者和让出/移动所有权的想法在Rust中似乎非常有限，但可以使用智能指针类型[`Rc`][rc.rs]在Rust中拥有 _共享所有权_；
它添加了引用计数。每次[克隆智能指针][Rc::clone]时，引用计数都会增加。
当克隆删除时，引用计数会减少。当引用计数达到零时，智能指针背后的实际实例将被删除。
以下基于前一个示例的示例说明了这些要点：

  [rc.rs]: https://doc.rust-lang.org/stable/std/rc/struct.Rc.html
  [Rc::clone]: https://doc.rust-lang.org/stable/std/rc/struct.Rc.html#method.clone

```rust
#![allow(dead_code, unused_variables)]

use std::rc::Rc;

struct Point {
    x: i32,
    y: i32,
}

impl Drop for Point {
    fn drop(&mut self) {
        println!("Point dropped!");
    }
}

fn main() {
    let a = Rc::new(Point { x: 12, y: 34 });
    let b = Rc::clone(&a); // share with b
    println!("a = {}, {}", a.x, a.y); // okay to use a
    println!("b = {}, {}", b.x, b.y);
}

// prints:
// a = 12, 34
// b = 12, 34
// Point dropped!
```

请注意：

- `Point`实现了`Drop`特征的`drop`方法，并在丢弃`Point`实例时打印一条消息。

- 在`main`中创建的point被包装在智能指针`Rc`后面，因此智能指针拥有该point而不是`a`。

- `b`获取智能指针的克隆，从而有效地将引用计数增加到 2。与前面的示例不同，其中`a`将其对point的所有权转让给`b`，
`a`和`b`都拥有智能指针的各自不同克隆，因此可以继续使用`a`和`b`。

- 编译器将确定`a`和`b`在`main`结束时超出范围时注入调用以丢弃每个。
`Rc`的`Drop`实现将减少引用计数，并且如果引用计数已达到零，还将丢弃其拥有的内容。
当发生这种情况时，`Point`的`Drop`实现将打印消息“Point 已丢弃！”。
该消息只打印一次，表明只创建、共享和丢弃了一个point。

`Rc`不是线程安全的。对于多线程程序中的共享所有权，Rust标准库提供了[`Arc`][arc.rs]。
Rust语言将阻止跨线程使用 `Rc`。

  [arc.rs]: https://doc.rust-lang.org/std/sync/struct.Arc.html

在.NET中，值类型（如 C# 中的`enum`和`struct`）位于栈中，而引用类型（C# 中的 `interface`、`record class` 和 `class`）则分配在堆中。
在Rust中，类型的种类（Rust中基本上是`enum`或`struct`）并不决定后备内存最终将位于何处。
默认情况下，它始终位于栈中，但.NET和C#具有装箱值类型的概念，即将它们复制到堆中，在堆上分配类型的方法是使用[`Box`][box.rs] 对其进行装箱：

  [box.rs]: https://doc.rust-lang.org/std/boxed/struct.Box.html

```rust
let stack_point = Point { x: 12, y: 34 };
let heap_point = Box::new(Point { x: 12, y: 34 });
```

与`Rc`和`Arc`一样，`Box`是一个智能指针，但与`Rc`和`Arc`不同，它独占拥有其背后的实例。
所有这些智能指针都分配堆上其类型参数“T”的实例。

C#中的`new`关键字创建一个类型的实例，而成员因为您在示例中看到的`Box::new`和`Rc::new`似乎具有类似的目的，`new`在Rust中没有特殊名称。
它只是一个 _约定名称_，这意味着一个工厂。
事实上，他们被称为类型的我 _相关函数_，这是Rust静态方法的方式。
