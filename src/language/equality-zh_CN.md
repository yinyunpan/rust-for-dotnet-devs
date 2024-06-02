# 相等

在C#中比较相等时，这在某些情况下是指测试_等价性_（也称为 _值相等性_），而在其他情况下是指测试 _引用平等性_，即测试两个变量是否引用内存中的同一个底层对象。
每个自定义类型都可以进行比较相等性，因为它继承自 `System.Object`（对于值类型，则为 `System.ValueType`，它继承自 `System.Object`），使用上述任一语义。

例如，在C#中比较等价性和引用相等性时：

```csharp
var a = new Point(1, 2);
var b = new Point(1, 2);
var c = a;
Console.WriteLine(a == b); // (1) True
Console.WriteLine(a.Equals(b)); // (1) True
Console.WriteLine(a.Equals(new Point(2, 2))); // (1) False
Console.WriteLine(ReferenceEquals(a, b)); // (2) False
Console.WriteLine(ReferenceEquals(a, c)); // (2) True

record Point(int X, int Y);
```

1. 相等运算符 `==` 和 `record Point` 上的 `Equals` 方法比较值相等性，因为记录默认支持值类型相等性。

2. 比较引用相等性测试变量是否引用内存中的相同底层对象。

在Rust中相等：

```rust
#[derive(Copy, Clone)]
struct Point(i32, i32);

fn main() {
    let a = Point(1, 2);
    let b = Point(1, 2);
    let c = a;
    println!("{}", a == b); // Error: "an implementation of `PartialEq<_>` might be missing for `Point`"
    println!("{}", a.eq(&b));
    println!("{}", a.eq(&Point(2, 2)));
}
```

上面的编译器错误说明，在Rust中，相等性比较 _总是_ 与特征实现相关。要支持使用 `==` 的比较，
类型必须实现 [`PartialEq`][partialeq.rs]。

修复上述示例意味着为 `Point` 派生 `PartialEq`。默认情况下，派生 `PartialEq` 将比较所有字段的相等性，
因此必须自己实现 `PartialEq`。这相当于C#中记录的相等性。

```rust
#[derive(Copy, Clone, PartialEq)]
struct Point(i32, i32);

fn main() {
    let a = Point(1, 2);
    let b = Point(1, 2);
    let c = a;
    println!("{}", a == b); // true
    println!("{}", a.eq(&b)); // true
    println!("{}", a.eq(&Point(2, 2))); // false
    println!("{}", a.eq(&c)); // true
}
```

另请参阅：

- [`Eq`][eq.rs] 为更严格的 `PartialEq`版本。

[partialeq.rs]: https://doc.rust-lang.org/std/cmp/trait.PartialEq.html
[eq.rs]: https://doc.rust-lang.org/std/cmp/trait.Eq.html
