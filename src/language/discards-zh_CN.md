# 丢弃

在C#中，[丢弃][net-discards] 表示编译器和其他程序忽略表达式的结果（或部分）。

有多种情况可以应用此方法，例如，作为一个基本示例，忽略表达式的结果。在C#中，它看起来像：

```csharp
_ = city.GetCityInformation(cityName);
```

在Rust中，[忽略表达式的结果][rust-ignoring-values]看起来相同的：

```rust
_ = city.get_city_information(city_name);
```

在C#中，丢弃也适用于解构元组：

```csharp
var (_, second) = ("first", "second");
```

并且，在Rust中同样如此：

```rust
let (_, second) = ("first", "second");
```

除了解构元组之外，Rust还提供了使用`..`解构结构体和枚举的功能，其中`..`代表类型的剩余部分：

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x), // x is 0
}
```

进行模式匹配时，丢弃或忽略匹配表达式的一部分通常很有用，例如在C#中：

```csharp
_ = ("first", "second") switch
{
    ("first", _) => "first element matched",
    (_, _) => "first element did not match"
};
```

同样，在Rust中这看起来几乎完全相同：

```rust
_ = match ("first", "second")
{
    ("first", _) => "first element matched",
    (_, _) => "first element did not match"
};
```

[net-discards]: https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/discards
[rust-ignoring-values]: https://doc.rust-lang.org/stable/book/ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern
[rust-destructuring]: https://doc.rust-lang.org/reference/patterns.html#destructuring
