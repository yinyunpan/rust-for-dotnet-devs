# 变量

考虑以下有关C#中变量赋值的示例：

```csharp
int x = 5;
```

在Rust中也是如此：

```rust
let x: i32 = 5;
```

到目前为止，这两种语言之间唯一明显的区别是类型声明的位置不同。
此外，C#和Rust都是类型安全：编译器保证存储在变量中的值始终是指定类型的。
使用编译器的自动推断变量类型的能力。在C#中：

```csharp
var x = 5;
```

在Rust中：

```rust
let x = 5;
```

扩展第一个示例以更新变量的值时（重新赋值），C#和Rust的行为不同：

```csharp
var x = 5;
x = 6;
Console.WriteLine(x); // 6
```

在Rust中，相同的语句不会编译：

```rust
let x = 5;
x = 6; // 错误：无法为不可变变量“x”赋值两次。
println!("{}", x);
```

在Rust中, 变量默认是 _不可变_。
将值绑定到名称后，无法更改变量的值。
变量可以通过以下方式 _可变_ 在变量名称前面添加 [`mut`][mut.rs]：

```rust
let mut x = 5;
x = 6;
println!("{}", x); // 6
```

Rust提供了一种替代方法来修复上面的示例，它不需要通过可变性，而是用变量 _遮蔽_：

```rust
let x = 5;
let x = 6;
println!("{}", x); // 6
```

C#也支持遮蔽，例如，局部变量可以对字段进行遮蔽，类型成员可以遮蔽基类型中的成员。
在Rust中，上面的例子该遮蔽还允许在不更改名称情况下更改变量的类型，如果要将数据转换为不同的类型，而不必每次都想出一个不同的名称。

另请参阅：

- [数据争用和争用条件][data races and race conditions] 以获取有关可变性影响的更多信息
- [范围和遮蔽][scope and shadowing]
- [内存管理][memory-management-section] 有关 _移动_ 和 _所有权_ 的解释

[mut.rs]: https://doc.rust-lang.org/std/keyword.mut.html
[memory-management-section]: ../memory-management/index.md
[data races and race conditions]: https://doc.rust-lang.org/nomicon/races.html
[scope and shadowing]: https://doc.rust-lang.org/stable/rust-by-example/variable_bindings/scope.html#scope-and-shadowing
