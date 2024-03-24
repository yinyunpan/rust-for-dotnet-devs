# 结构化类型

.NET常用对象和集合类型，它们在Rust映射

| C#           | Rust      |
| ------------ | --------- |
| `Array`      | `Array`   |
| `List`       | `Vec`     |
| `Tuple`      | `Tuple`   |
| `Dictionary` | `HashMap` |

## 数组

Rust支持固定数组的方式和.NET相同

C#:

```csharp
int[] someArray = new int[2] { 1, 2 };
```

Rust:

```rust
let someArray: [i32; 2] = [1,2];
```

## 列表

Rust对应`List<T>`是`Vec<T>`。Arrays可以转换到Vecs，反之亦然。

C#:

```csharp
var something = new List<string>
{
    "a",
    "b"
};

something.Add("c");
```

Rust:

```rust
let mut something = vec![
    "a".to_owned(),
    "b".to_owned()
];

something.push("c".to_owned());
```

## 元组

C#:

```csharp
var something = (1, 2)
Console.WriteLine($"a = {something.Item1} b = {something.Item2}");
```

Rust:

```rust
let something = (1, 2);
println!("a = {} b = {}", something.0, something.1);

// 支持解构
let (a, b) = something;
println!("a = {} b = {}", a, b);
```

> **注释**: Rust元组的元素不能命名，在C#中可以。唯一的方法访问元祖的原始是通过元素的索引或者解构元组。

## 字典

Rust对应`Dictionary<TKey, TValue>`是`Hashmap<K, V>`.

C#:

```csharp
var something = new Dictionary<string, string>
{
    { "Foo", "Bar" },
    { "Baz", "Qux" }
};

something.Add("hi", "there");
```

Rust:

```rust
let mut something = HashMap::from([
    ("Foo".to_owned(), "Bar".to_owned()),
    ("Baz".to_owned(), "Qux".to_owned())
]);

something.insert("hi".to_owned(), "there".to_owned());
```

另请参阅：

- [Rust标准库-集合](https://doc.rust-lang.org/std/collections/index.html)
