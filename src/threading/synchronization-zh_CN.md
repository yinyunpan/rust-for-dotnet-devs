# 同步

When data is shared between threads, one needs to synchronize read-write
access to the data in order to avoid corruption. The C# offers the `lock`
keyword as a synchronization primitive (which desugars to exception-safe use
of `Monitor` from .NET):

```csharp
using System;
using System.Threading;

var dataLock = new object();
var data = 0;
var threads = new List<Thread>();

for (var i = 0; i < 10; i++)
{
    var thread = new Thread(() =>
    {
        for (var j = 0; j < 1000; j++)
        {
            lock (dataLock)
                data++;
        }
    });
    threads.Add(thread);
    thread.Start();
}

foreach (var thread in threads)
    thread.Join();

Console.WriteLine(data);
```

In Rust, one must make explicit use of concurrency structures like `Mutex`:

```rust
use std::thread;
use std::sync::{Arc, Mutex};

fn main() {
    let data = Arc::new(Mutex::new(0)); // (1)

    let mut threads = vec![];
    for _ in 0..10 {
        let data = Arc::clone(&data); // (2)
        let thread = thread::spawn(move || { // (3)
            for _ in 0..1000 {
                let mut data = data.lock().unwrap();
                *data += 1; // (4)
            }
        });
        threads.push(thread);
    }

    for thread in threads {
        thread.join().unwrap();
    }

    println!("{}", data.lock().unwrap());
}
```

需要注意以下几点：

- 由于`Mutex`实例的所有权以及它所保护的数据将由多个线程共享，因此它被包装在`Arc`(1)中。
`Arc`提供原子引用计数，每次克隆时都会递增(2)，每次删除时都会递减。
当计数达到零时，互斥锁以及它所保护的数据将被删除。
这在[内存管理][Memory Management]中更详细地讨论。

- 每个线程的闭包实例都获得 _克隆的引用_ (2)的所有权(3)。

- 类似指针的代码`*data += 1`(4)，即使看起来像，也不是某种不安全的指针访问。
它正在更新 _包装_ 在[互斥锁保护][mutex guard]中的数据。

与C#版本不同，在C#版本中，可以通过注释掉`lock`语句使其线程不安全，
而Rust版本如果以任何方式（例如注释掉部分）更改，使其线程不安全，则会拒绝编译。
这表明，在C#和.NET 中，通过谨慎使用同步结构编写线程安全代码是开发人员的责任，而在Rust中，可以依赖编译器。

编译器能够提供帮助，因为Rust中的数据结构由特殊特征标记（参见[接口][Interfaces]）：`Sync`和`Send`。
[`Sync`][sync.rs]表示对类型实例的引用可以在线程之间安全地共享。
[`Send`][send.rs]表示跨线程边界对类型的实例是安全的。
有关更多信息，请参阅Rust书中的“[无畏并发][Fearless Concurrency]”章节。

  [Fearless Concurrency]: https://doc.rust-lang.org/book/ch16-00-concurrency.html
  [Memory Management]: ../memory-management/index.md
  [mutex guard]: https://doc.rust-lang.org/stable/std/sync/struct.MutexGuard.html
  [sync.rs]: https://doc.rust-lang.org/stable/std/marker/trait.Sync.html
  [send.rs]: https://doc.rust-lang.org/stable/std/marker/trait.Send.html
  [interfaces]: ../language/custom-types/interfaces.md
