# LINQ

本节讨论上下文中的LINQ，目的是查询或转换序列（`IEnumerable`/`IEnumerable<T>`）以及通常的集合（如lists, sets and dictionaries）。

## `IEnumerable<T>`

Rust中`IEnumerable<T>`的等价物是[`IntoIterator`][into-iter.rs]。
正如`IEnumerable<T>.GetEnumerator()`的实现在.NET中返回一个`IEnumerator<T>`，
`IntoIterator::into_iter`的实现返回一个[`Iterator`][iter.rs]。
然而，当需要迭代容器中的项目时，通过上述类型来宣传迭代支持，这两种语言都以循环构造的形式为可迭代对象提供语法糖。
在C#中，有`foreach`：

```csharp
using System;
using System.Text;

var values = new[] { 1, 2, 3, 4, 5 };
var output = new StringBuilder();

foreach (var value in values)
{
    if (output.Length > 0)
        output.Append(", ");
    output.Append(value);
}

Console.Write(output); // Prints: 1, 2, 3, 4, 5
```

在Rust中，等效的是`for`：

```rust
use std::fmt::Write;

fn main() {
    let values = [1, 2, 3, 4, 5];
    let mut output = String::new();

    for value in values {
        if output.len() > 0 {
            output.push_str(", ");
        }
        // ! discard/ignore any write error
        _ = write!(output, "{value}");
    }

    println!("{output}");  // Prints: 1, 2, 3, 4, 5
}
```

可迭代对象的`for`循环本质上简化为以下内容：

```rust
use std::fmt::Write;

fn main() {
    let values = [1, 2, 3, 4, 5];
    let mut output = String::new();

    let mut iter = values.into_iter();      // get iterator
    while let Some(value) = iter.next() {   // loop as long as there are more items
        if output.len() > 0 {
            output.push_str(", ");
        }
        _ = write!(output, "{value}");
    }

    println!("{output}");
}
```

Rust的所有权和数据竞争条件规则适用于所有实例和数据，迭代也不例外。
因此，虽然循环遍历数组可能看起来很简单，并且与C#非常相似，但当需要多次迭代同一个集合/可迭代对象时，必须注意所有权。
以下示例迭代整数列表两次，一次打印它们的和，另一次确定并打印最大整数：

```rust
fn main() {
    let values = vec![1, 2, 3, 4, 5];

    // sum all values

    let mut sum = 0;
    for value in values {
        sum += value;
    }
    println!("sum = {sum}");

    // determine maximum value

    let mut max = None;
    for value in values {
        if let Some(some_max) = max { // if max is defined
            if value > some_max {     // and value is greater
                max = Some(value)     // then note that new max
            }
        } else {                      // max is undefined when iteration starts
            max = Some(value)         // so set it to the first value
        }
    }
    println!("max = {max:?}");
}
```

但是，上面的代码由于一个细微的差别而被编译器拒绝：`values`已从数组更改为[`Vec<int>`][vec.rs]，即 _vector_，这是Rust的可增长数组类型（如.NET中的 `List<T>`）。
`values` 的第一次迭代最终在整数相加时 _消耗_ 每个值。换句话说，vector中 _每个项目_ 的所有权传递给循环的迭代变量：`value`。
由于`value`在循环的每次迭代结束时超出范围，因此它拥有的实例将被删除。如果`values`是堆分配数据的vector，则当循环移动到下一个项目时，支持每个项目的堆内存将被释放。
要解决该问题，必须在`for`循环中通过`&values`请求对 _共享_ 引用进行迭代。因此，`value`最终成为对某个项目的共享引用，而不是取得其所有权。

  [vec.rs]: https://doc.rust-lang.org/stable/std/vec/struct.Vec.html

下面是编译的上一个示例的更新版本。修复方法是简单地在每个`for`循环中将`values`替换为`&values`。

```rust
fn main() {
    let values = vec![1, 2, 3, 4, 5];

    // sum all values

    let mut sum = 0;
    for value in &values {
        sum += value;
    }
    println!("sum = {sum}");

    // determine maximum value

    let mut max = None;
    for value in &values {
        if let Some(some_max) = max { // if max is defined
            if value > some_max {     // and value is greater
                max = Some(value)     // then note that new max
            }
        } else {                      // max is undefined when iteration starts
            max = Some(value)         // so set it to the first value
        }
    }
    println!("max = {max:?}");
}
```

The ownership and dropping can be seen in action even with `values` being an
array instead of a vector. Consider just the summing loop from the above
example over an array of a structure that wraps an integer:

```rust
struct Int(i32);

impl Drop for Int {
    fn drop(&mut self) {
        println!("{} dropped", self.0)
    }
}

fn main() {
    let values = [Int(1), Int(2), Int(3), Int(4), Int(5)];
    let mut sum = 0;

    for value in values {
        sum += value.0;
    }

    println!("sum = {sum}");
}
```

`Int`实现了`Drop`，这样当实例被丢弃时就会打印一条消息。运行上述代码将打印：

    value = Int(1)
    Int(1) dropped
    value = Int(2)
    Int(2) dropped
    value = Int(3)
    Int(3) dropped
    value = Int(4)
    Int(4) dropped
    value = Int(5)
    Int(5) dropped
    sum = 15

很明显，每个值都是在循环运行时获取和丢弃的。循环完成后，将打印总和。
如果将`for`循环中的`values`改为`&values`，如下所示：

```rust
for value in &values {
    // ...
}
```

那么程序的输出将发生根本性的变化：

    value = Int(1)
    value = Int(2)
    value = Int(3)
    value = Int(4)
    value = Int(5)
    sum = 15
    Int(1) dropped
    Int(2) dropped
    Int(3) dropped
    Int(4) dropped
    Int(5) dropped

这次，在循环过程中会获取值但不会丢弃，因为每个项都不会被迭代循环的变量所拥有。
循环完成后会打印总和。最后，当仍然拥有所有`Int`实例的`values`数组在`main`末尾超出范围时，它的丢弃会依次丢弃所有`Int`实例。

这些示例表明，虽然迭代集合类型似乎在Rust和C#之间有很多相似之处，从循环构造到迭代抽象，但在所有权方面仍然存在细微的差异，这可能导致编译器在某些情况下拒绝代码。

另请参阅：

- [Iterator][iter-mod]
- [Iterating by reference]

[into-iter.rs]: https://doc.rust-lang.org/std/iter/trait.IntoIterator.html
[iter.rs]: https://doc.rust-lang.org/core/iter/trait.Iterator.html
[iter-mod]: https://doc.rust-lang.org/std/iter/index.html
[iterating by reference]: https://doc.rust-lang.org/std/iter/index.html#iterating-by-reference

## 运算符

LINQ中的运算符以C#扩展方法的形式实现，这些扩展方法可以链接在一起形成一组操作，最常见的操作是对某种数据源进行查询。
C#还提供了受SQL启发的 _查询语法_，其中包含 `from`、`where`、`select`、`join` 等子句，可以作为方法链接的替代或补充。
许多命令式循环可以在LINQ中重写为更具表现力和可组合性的查询。

Rust不提供任何类似C#的查询语法。它具有在Rust术语中称为[适配器]的可迭代类型方法，因此可直接与C#中的方法链接相媲美。
但是，虽然在C#中将命令式循环重写为 LINQ 代码通常有利于提高表现力、稳健性和可组合性，但性能会有所降低。
计算绑定的命令式循环通常运行得更快，因为它们可以通过JIT编译器进行优化，并且产生的虚拟调度或间接函数调用更少。
Rust中令人惊讶的部分是，选择在抽象（如迭代器）上使用方法链与手动编写命令式循环之间没有性能权衡。
因此，在代码中看到前者更为常见。

下表列出了 Rust 中最常见的LINQ方法及其近似对应方法。

| .NET              | Rust         | Note        |
| ----------------- | ------------ | ----------- |
| `Aggregate`       | `reduce`     | See note 1. |
| `Aggregate`       | `fold`       | See note 1. |
| `All`             | `all`        |             |
| `Any`             | `any`        |             |
| `Concat`          | `chain`      |             |
| `Count`           | `count`      |             |
| `ElementAt`       | `nth`        |             |
| `GroupBy`         | -            |             |
| `Last`            | `last`       |             |
| `Max`             | `max`        |             |
| `Max`             | `max_by`     |             |
| `MaxBy`           | `max_by_key` |             |
| `Min`             | `min`        |             |
| `Min`             | `min_by`     |             |
| `MinBy`           | `min_by_key` |             |
| `Reverse`         | `rev`        |             |
| `Select`          | `map`        |             |
| `Select`          | `enumerate`  |             |
| `SelectMany`      | `flat_map`   |             |
| `SelectMany`      | `flatten`    |             |
| `SequenceEqual`   | `eq`         |             |
| `Single`          | `find`       |             |
| `SingleOrDefault` | `try_find`   |             |
| `Skip`            | `skip`       |             |
| `SkipWhile`       | `skip_while` |             |
| `Sum`             | `sum`        |             |
| `Take`            | `take`       |             |
| `TakeWhile`       | `take_while` |             |
| `ToArray`         | `collect`    | See note 2. |
| `ToDictionary`    | `collect`    | See note 2. |
| `ToList`          | `collect`    | See note 2. |
| `Where`           | `filter`     |             |
| `Zip`             | `zip`        |             |

1. 不接受种子值的`Aggregate`重载相当于`reduce`，而接受种子值的`Aggregate`重载相当于`fold`。

2. Rust中的[`collect`][collect.rs]通常适用于任何可收集类型，该类型被定义为[一种可以从迭代器初始化自身的类型（参见 `FromIterator`）][FromIter.rs]。
`collect`需要一个目标类型，编译器有时很难推断，因此 _turbofish_ (`::<>`)经常与其一起使用，如 `collect::<Vec<_>>()`。
这就是为什么`collect`出现在许多 LINQ 扩展方法旁边，这些方法将可枚举/可迭代源转换为某些集合类型实例。

  [FromIter.rs]: https://doc.rust-lang.org/stable/std/iter/trait.FromIterator.html

以下示例显示了C#中的转换序列与Rust中的转换序列有多么相似。首先在 C# 中：

```csharp
var result =
    Enumerable.Range(0, 10)
              .Where(x => x % 2 == 0)
              .SelectMany(x => Enumerable.Range(0, x))
              .Aggregate(0, (acc, x) => acc + x);

Console.WriteLine(result); // 50
```

在Rust中：

```rust
let result = (0..10)
    .filter(|x| x % 2 == 0)
    .flat_map(|x| (0..x))
    .fold(0, |acc, x| acc + x);

println!("{result}"); // 50
```

[adapters]: https://doc.rust-lang.org/std/iter/index.html#adapters
[collect.rs]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect

## 延迟执行（惰性）

LINQ中的许多运算符被设计为惰性的，因此它们只在绝对需要时才工作。
这样就可以组合或链接多个操作/方法而不会产生任何副作用。
例如，LINQ运算符可以返回已初始化的`IEnumerable<T>`，但在迭代之前不会生成、计算或实现 `T` 的任何项。
该运算符被认为具有 _延迟执行_ 语义。如果每个`T`在迭代到达时计算（而不是在迭代开始时计算），则该运算符被称为 _流式传输_ 结果。

Rust迭代器具有相同的[_惰性_][iter-laziness]和流式传输概念。

  [iter-laziness]: https://doc.rust-lang.org/std/iter/index.html#laziness

在这两种情况下，这都允许表示 _无限序列_，其中底层序列是无限的，但开发人员决定如何终止序列。
以下示例在 C# 中显示了这一点：

```csharp
foreach (var x in InfiniteRange().Take(5))
    Console.Write($"{x} "); // Prints "0 1 2 3 4"

IEnumerable<int> InfiniteRange()
{
    for (var i = 0; ; ++i)
        yield return i;
}
```

Rust通过无限范围支持相同的概念：

```rust
// Generators and yield in Rust are unstable at the moment, so
// instead, this sample uses Range:
// https://doc.rust-lang.org/std/ops/struct.Range.html

for value in (0..).take(5) {
    print!("{value} "); // Prints "0 1 2 3 4"
}
```

## 迭代器方法 (`yield`)

C#具有`yield`关键字，使开发人员能够快速编写 _迭代器方法_。
迭代器方法的返回类型可以是`IEnumerable<T>`或`IEnumerator<T>`。
然后，编译器将方法的主体转换为返回类型的具体实现，而开发人员不必每次都编写一个完整的类。
在撰写本文时，_[Coroutines][coroutines.rs]_（Rust中的名称）仍被视为不稳定的功能。

  [coroutines.rs]: https://doc.rust-lang.org/unstable-book/language-features/coroutines.html
