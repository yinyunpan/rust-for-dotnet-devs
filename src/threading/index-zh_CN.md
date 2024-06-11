# 线程

Rust标准库支持线程、同步和并发。
此外，语言本身和标准库确实对这些概念提供了基本支持，许多附加功能由crate提供，本文将不予介绍。

以下列出了.NET中线程类型和方法到Rust的近似映射：

| .NET               | Rust                      |
| ------------------ | ------------------------- |
| `Thread`           | `std::thread::thread`     |
| `Thread.Start`     | `std::thread::spawn`      |
| `Thread.Join`      | `std::thread::JoinHandle` |
| `Thread.Sleep`     | `std::thread::sleep`      |
| `ThreadPool`       | -                         |
| `Mutex`            | `std::sync::Mutex`        |
| `Semaphore`        | -                         |
| `Monitor`          | `std::sync::Mutex`        |
| `ReaderWriterLock` | `std::sync::RwLock`       |
| `AutoResetEvent`   | `std::sync::Condvar`      |
| `ManualResetEvent` | `std::sync::Condvar`      |
| `Barrier`          | `std::sync::Barrier`      |
| `CountdownEvent`   | `std::sync::Barrier`      |
| `Interlocked`      | `std::sync::atomic`       |
| `Volatile`         | `std::sync::atomic`       |
| `ThreadLocal`      | `std::thread_local`       |

在C#/.NET和Rust中，启动线程并等待其完成的方式相同。
下面是一个简单的C#程序，它创建一个线程（线程将一些文本打印到标准输出），然后等待它结束：

```csharp
using System;
using System.Threading;

var thread = new Thread(() => Console.WriteLine("Hello from a thread!"));
thread.Start();
thread.Join(); // wait for thread to finish
```

Rust中的相同代码如下：

```rust
use std::thread;

fn main() {
    let thread = thread::spawn(|| println!("Hello from a thread!"));
    thread.join().unwrap(); // wait for thread to finish
}
```

在.NET中，创建和初始化线程对象以及启动线程是两个不同的操作，
而在Rust中，这两个操作通过`thread::spawn`同时发生。

在.NET中，可以将数据作为参数发送给线程：

```csharp
#nullable enable

using System;
using System.Text;
using System.Threading;

var t = new Thread(obj =>
{
    var data = (StringBuilder)obj!;
    data.Append(" World!");
});

var data = new StringBuilder("Hello");
t.Start(data);
t.Join();

Console.WriteLine($"Phrase: {data}");
```

然而，更现代或更简洁的版本将使用闭包：

```csharp
using System;
using System.Text;
using System.Threading;

var data = new StringBuilder("Hello");

var t = new Thread(obj => data.Append(" World!"));

t.Start();
t.Join();

Console.WriteLine($"Phrase: {data}");
```

在Rust中，没有 `thread::spawn` 的变体可以实现相同的功能。
相反，数据通过闭包传递给线程：

```rust
use std::thread;

fn main() {
    let data = String::from("Hello");
    let handle = thread::spawn(move || {
        let mut data = data;
        data.push_str(" World!");
        data
    });
    println!("Phrase: {}", handle.join().unwrap());
}
```

需要注意以下几点：

- 需要使用`move`关键字来将`data`的所有权移动或传递给线程的闭包。
完成此操作后，在`main`中继续使用`main`的`data`变量就不再合法。
如果需要，必须复制或克隆`data`（取决于值支持的类型）。

- Rust线​​程可以返回值，就像C#中的任务一样，它将成为`join`方法的返回值。

- 也可以通过闭包将数据传递给C#线程，就像Rust示例一样，但C#版本不需要担心所有权，因为一旦没有人再引用数据，GC就会回收数据背后的内存。
