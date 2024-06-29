# 异步编程

.NET和Rust都支持异步编程模型，它们的用法看起来彼此相似。以下示例从非常高的层次展示了C#中的异步代码：

```csharp
async Task<string> PrintDelayed(string message, CancellationToken cancellationToken)
{
    await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    return $"Message: {message}";
}
```

Rust代码的结构类似。以下示例依赖[async-std]来实现 `sleep`：

```rust
use std::time::Duration;
use async_std::task::sleep;

async fn format_delayed(message: &str) -> String {
    sleep(Duration::from_secs(1)).await;
    format!("Message: {}", message)
}
```

1. Rust[`async`][async.rs]关键字将代码块转换为状态机，该状态机实现了名为[`Future`][future.rs] 的特征，
   类似于C#编译器将`async`代码转换为状态机的方式。在这两种语言中，这都允许按顺序编写异步代码。

2. 请注意，对于Rust和C#，异步方法/函数都以async关键字为前缀，但返回类型不同。
   C#中的异步方法指示完整和实际的返回类型，因为它可能会有所不同。
   例如，通常会看到一些方法返回`Task<T>`，而其他方法返回`ValueTask<T>`。
   在Rust中，指定 _内部类型_ `String`就足够了，因为它 _总是一些future_；也就是说，实现`Future`特征的类型。

3. C#和Rust中的`await`关键字位置不同。在C#中，通过在表达式前添加`await`来等待`Task`。
   在Rust中，在表达式后添加`.await`关键字可以实现 _方法链_，即使`await`不是方法。

另请参阅：

- [Rust中的异步编程][Asynchronous programming in Rust]

[async-std]: https://docs.rs/async-std/latest/async_std/
[async.rs]: https://doc.rust-lang.org/std/keyword.async.html
[future.rs]: https://doc.rust-lang.org/std/future/trait.Future.html
[Asynchronous programming in Rust]: https://rust-lang.github.io/async-book/

## 执行任务

从以下示例中可以看出，尽管没有等待`PrintDelayed`方法，但它仍会执行：

```csharp
var cancellationToken = CancellationToken.None;
PrintDelayed("message", cancellationToken); // Prints "message" after a second.
await Task.Delay(TimeSpan.FromSeconds(2), cancellationToken);

async Task PrintDelayed(string message, CancellationToken cancellationToken)
{
    await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    Console.WriteLine(message);
}
```

在Rust中，相同的函数调用不会打印任何内容。

```rust
use async_std::task::sleep;
use std::time::Duration;

#[tokio::main] // used to support an asynchronous main method
async fn main() {
    print_delayed("message"); // Prints nothing.
    sleep(Duration::from_secs(2)).await;
}

async fn print_delayed(message: &str) {
    sleep(Duration::from_secs(1)).await;
    println!("{}", message);
}
```

这是因为Future是惰性的：它们在运行之前什么都不做。
运行`Future`的最常见方式是`.await`。当在`Future`上调用`.await`时，它将尝试运行它直至完成。
如果`Future`被阻止，它将放弃对当前线程的控制。
当可以取得更多进展时，执行器将拾取`Future`并恢复运行，从而允许`.await`解析（参见 [`async/.await`][async-await.rs]）。

虽然等待函数可以从其他`async`函数中工作，但`main`[不允许是`async`][error-E0752]。
这是因为Rust本身不提供用于执行异步代码的运行时。因此，有用于执行异步代码的库，称为[异步运行时]。
[Tokio][tokio.rs]就是这样一个异步运行时，它被频繁使用。
上面例子中的[`tokio::main`][tokio-main.rs]将`async main`函数标记为运行时要执行的入口点，该入口点在使用宏时自动设置。

[tokio.rs]: https://crates.io/crates/tokio
[tokio-main.rs]: https://docs.rs/tokio/latest/tokio/attr.main.html
[async-await.rs]: https://rust-lang.github.io/async-book/03_async_await/01_chapter.html#asyncawait
[error-E0752]: https://doc.rust-lang.org/error-index.html#E0752
[async runtimes]: https://rust-lang.github.io/async-book/08_ecosystem/00_chapter.html#async-runtimes

## Task cancellation

前面的C#示例包括将`CancellationToken`传递给异步方法，这在.NET 中被认为是最佳实践。
`CancellationToken` 可用于中止异步操作。

由于futures在Rust中是惰性的（它们只有在轮询时才会取得进展），因此在Rust中取消的工作方式不同。
当删除`Future`时，`Future`将不会再取得进展。它还将删除所有实例化值，直到由于某些未完成的异步操作而暂停future为止。
这就是为什么Rust中的大多数异步函数不接受参数来发出取消信号的原因，也是为什么删除future有时被称为 _取消_ 的原因。

[`tokio_util::sync::CancellationToken`][cancellation-token.rs]提供与.NET`CancellationToken`等效的功能，
用于在无法在`Future`上实现`Drop`特征的情况下，对取消发出信号并做出反应。

[cancellation-token.rs]: https://docs.rs/tokio-util/latest/tokio_util/sync/struct.CancellationToken.html

## 执行多个任务

在.NET中，`Task.WhenAny`和`Task.WhenAll`经常用于处理多个任务的执行。

`Task.WhenAny`在任何任务完成后立即完成。
例如，Tokio提供了[`tokio::select!`][tokio-select]宏作为`Task.WhenAny`的替代方案，这意味着等待多个并发分支。

```csharp
var cancellationToken = CancellationToken.None;

var result =
    await Task.WhenAny(Delay(TimeSpan.FromSeconds(2), cancellationToken),
                       Delay(TimeSpan.FromSeconds(1), cancellationToken));

Console.WriteLine(result.Result); // Waited 1 second(s).

async Task<string> Delay(TimeSpan delay, CancellationToken cancellationToken)
{
    await Task.Delay(delay, cancellationToken);
    return $"Waited {delay.TotalSeconds} second(s).";
}
```

对于Rust来说也是同样的例子：

```rust
use std::time::Duration;
use tokio::{select, time::sleep};

#[tokio::main]
async fn main() {
    let result = select! {
        result = delay(Duration::from_secs(2)) => result,
        result = delay(Duration::from_secs(1)) => result,
    };

    println!("{}", result); // Waited 1 second(s).
}

async fn delay(delay: Duration) -> String {
    sleep(delay).await;
    format!("Waited {} second(s).", delay.as_secs())
}
```

同样，这两个示例在语义上存在重大差异。
最重要的是，`tokio::select!`将取消所有剩余分支，而`Task.WhenAny`则让用户自行决定是否取消任何正在进行的任务。

类似地，`Task.WhenAll`可以被[`tokio::join!`][tokio-join] 替换。

[tokio-select]: https://docs.rs/tokio/latest/tokio/macro.select.html
[tokio-join]: https://docs.rs/tokio/latest/tokio/macro.join.html

## 多个消费者

在.NET中，`Task`可以跨多个消费者使用。
所有消费者都可以等待任务，并在任务完成或失败时收到通知。
在Rust中，`Future`无法克隆或复制，并且`await`会转移所有权。
`futures::FutureExt::shared`扩展为`Future`创建可克隆的句柄，然后可以将其分发给多个消费者。

```rust
use futures::FutureExt;
use std::time::Duration;
use tokio::{select, time::sleep, signal};
use tokio_util::sync::CancellationToken;

#[tokio::main]
async fn main() {
    let token = CancellationToken::new();
    let child_token = token.child_token();

    let bg_operation = background_operation(child_token);

    let bg_operation_done = bg_operation.shared();
    let bg_operation_final = bg_operation_done.clone();

    select! {
        _ = bg_operation_done => {},
        _ = signal::ctrl_c() => {
            token.cancel();
        },
    }

    bg_operation_final.await;
}

async fn background_operation(cancellation_token: CancellationToken) {
    select! {
        _ = sleep(Duration::from_secs(2)) => println!("Background operation completed."),
        _ = cancellation_token.cancelled() => println!("Background operation cancelled."),
    }
}
```

## 异步迭代

虽然.NET中有[`IAsyncEnumerable<T>`][async-enumerable.net]和[`IAsyncEnumerator<T>`][async-enumerator.net]，
但Rust在标准库中还没有异步迭代的API。为了支持异步迭代，[`futures`][futures-stream.rs]中的[`Stream`][stream.rs]特性提供了一组类似的功能。

在C#中，编写异步迭代器的语法与编写同步迭代器的语法类似：

```csharp
await foreach (int item in RangeAsync(10, 3).WithCancellation(CancellationToken.None))
    Console.Write(item + " "); // Prints "10 11 12".

async IAsyncEnumerable<int> RangeAsync(int start, int count)
{
    for (int i = 0; i < count; i++)
    {
        await Task.Delay(TimeSpan.FromSeconds(i));
        yield return start + i;
    }
}
```

在Rust中，有几种类型实现了`Stream`特性，因此可用于创建流，例如`futures::channel::mpsc`。
对于更接近C#的语法，[`async-stream`][tokio-async-stream]提供了一组宏，可用于简洁地生成流。

```rust
use async_stream::stream;
use futures_core::stream::Stream;
use futures_util::{pin_mut, stream::StreamExt};
use std::{
    io::{stdout, Write},
    time::Duration,
};
use tokio::time::sleep;

#[tokio::main]
async fn main() {
    let stream = range(10, 3);
    pin_mut!(stream); // needed for iteration
    while let Some(result) = stream.next().await {
        print!("{} ", result); // Prints "10 11 12".
        stdout().flush().unwrap();
    }
}

fn range(start: i32, count: i32) -> impl Stream<Item = i32> {
    stream! {
        for i in 0..count {
            sleep(Duration::from_secs(i as _)).await;
            yield start + i;
        }
    }
}
```

[async-enumerable.net]: https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.iasyncenumerable-1
[async-enumerator.net]: https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.iasyncenumerator-1
[stream.rs]: https://rust-lang.github.io/async-book/05_streams/01_chapter.html
[futures-stream.rs]: https://docs.rs/futures/latest/futures/stream/trait.Stream.html
[tokio-async-stream]: https://github.com/tokio-rs/async-stream
