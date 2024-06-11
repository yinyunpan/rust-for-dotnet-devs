# 生产者-消费者

生产者-消费者模式非常常见，用于在线程之间分配工作，其中数据从生产线程传递到消费线程，而无需共享或锁定。
.NET对此有非常丰富的支持，但在最基本的层面上，`System.Collections.Concurrent`提供了`BlockingCollection`，
如下一个C#示例中所示：

```csharp
using System;
using System.Threading;
using System.Collections.Concurrent;

var messages = new BlockingCollection<string>();
var producer = new Thread(() =>
{
    for (var n = 1; i < 10; i++)
        messages.Add($"Message #{n}");
    messages.CompleteAdding();
});

producer.Start();

// main thread is the consumer here
foreach (var message in messages.GetConsumingEnumerable())
    Console.WriteLine(message);

producer.Join();
```

在Rust中，也可以使用 _channels_ 来实现同样的效果。
标准库主要提供`mpsc::channel`，这是一个支持多个生产者和单个消费者的通道。
上述C#示例在Rust中的粗略翻译如下：

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    let producer = thread::spawn(move || {
        for n in 1..10 {
            tx.send(format!("Message #{}", n)).unwrap();
        }
    });

    // main thread is the consumer here
    for received in rx {
        println!("{}", received);
    }

    producer.join().unwrap();
}
```

与Rust中的通道一样，.NET也在`System.Threading.Channels`命名空间中提供通道，但它主要设计用于使用`async`和`await`的任务和异步编程。
[Rust空间中的异步友好通道的等效项是Tokio运行时提供][tokio-channels]。

  [tokio-channels]: https://tokio.rs/tokio/tutorial/channels
