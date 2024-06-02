# 泛型

C#中的泛型提供了一种为类型和方法创建定义的方法可以在其他类型上进行参数化。
这提高了代码重用和类型安全性和性能（例如，避免运行时强制转换）。
考虑以下示例为任何值添加时间戳的泛型类型：

```csharp
using System;

sealed record Timestamped<T>(DateTime Timestamp, T Value)
{
    public Timestamped(T value) : this(DateTime.UtcNow, value) { }
}
```

Rust也有如上面所示的等价泛型：

```rust
use std::time::*;

struct Timestamped<T> { value: T, timestamp: SystemTime }

impl<T> Timestamped<T> {
    fn new(value: T) -> Self {
        Self { value, timestamp: SystemTime::now() }
    }
}
```

另请参阅：

- [泛型数据类型][Generic data types]

[Generic data types]: https://doc.rust-lang.org/book/ch10-01-syntax.html

## 泛型类型约束

在C#中，通过使用`where`语句[约束泛型类型][type-constraints.cs]。
以下示例显示了C#中的此类约束：

```csharp
using System;

// 注意：记录自动实现`IEquatable`。以下实现明确地显示了这一点，以便与Rust进行比较。
sealed record Timestamped<T>(DateTime Timestamp, T Value) :
    IEquatable<Timestamped<T>>
    where T : IEquatable<T>
{
    public Timestamped(T value) : this(DateTime.UtcNow, value) { }

    public bool Equals(Timestamped<T>? other) =>
        other is { } someOther
        && Timestamp == someOther.Timestamp
        && Value.Equals(someOther.Value);

    public override int GetHashCode() => HashCode.Combine(Timestamp, Value);
}
```

在Rust中也可以实现同样的目标

```rust
use std::time::*;

struct Timestamped<T> { value: T, timestamp: SystemTime }

impl<T> Timestamped<T> {
    fn new(value: T) -> Self {
        Self { value, timestamp: SystemTime::now() }
    }
}

impl<T> PartialEq for Timestamped<T>
    where T: PartialEq {
    fn eq(&self, other: &Self) -> bool {
        self.value == other.value && self.timestamp == other.timestamp
    }
}
```

泛型类型约束在Rust中称为[bounds][bounds.rs]。

在C#版本中，`Timestamped<T>`实例 _仅仅_ 可以为`T`创建，该实例自己实现`IEquatable<T>`，
但请注意 Rust 版本更多灵活，因为它的`Timestamped<T>` _有条件实现_`PartialEq`。
这意味着不可等式的`T`仍然可以创建`Timestamped<T>`实例，但随后`Timestamped<T>`将不会
通过`PartialEq`表示这样的`T`方式实现平等。

另请参阅：

- [作为参数的特征][Traits as parameters]
- [实现特征的返回类型][Returning types that implement traits]

[type-constraints.cs]: https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
[bounds.rs]: https://doc.rust-lang.org/rust-by-example/generics/bounds.html
[Traits as parameters]: https://doc.rust-lang.org/book/ch10-02-traits.html#traits-as-parameters
[Returning types that implement traits]: https://doc.rust-lang.org/book/ch10-02-traits.html#returning-types-that-implement-traits
