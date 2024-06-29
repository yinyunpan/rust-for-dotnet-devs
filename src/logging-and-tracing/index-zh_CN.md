# 日志记录和跟踪

.NET支持多种日志记录 API。在大多数情况下，`ILogger`是一个很好的默认选择，因为它可以与各种内置和第三方日志记录提供程序配合使用。
在C#中，结构化日志记录的一个最小示例可能如下所示：

```csharp
using Microsoft.Extensions.Logging;

using var loggerFactory = LoggerFactory.Create(builder => builder.AddConsole());
var logger = loggerFactory.CreateLogger<Program>();
logger.LogInformation("Hello {Day}.", "Thursday"); // Hello Thursday.
```

在Rust中，[log][log.rs]提供了一个轻量级的日志外观。它的功能比`ILogger`少，例如，它尚未提供（稳定的）结构化日志记录或日志范围。

对于具有与.NET功能相同的功能，Tokio提供了[`tracing`][tracing.rs]。`tracing`是一个用于检测Rust应用程序以收集结构化、基于事件的诊断信息的框架。
[`tracing_subscriber`][tracing-subscriber.rs]可用于实现和编写`tracing`订阅者。
上面带有`tracing`和`tracing_subscriber`的相同结构化日志记录示例如下所示：

```rust
fn main() {
    // install global default ("console") collector.
    tracing_subscriber::fmt().init();
    tracing::info!("Hello {Day}.", Day = "Thursday"); // Hello Thursday.
}
```

[OpenTelemetry][opentelemetry.rs]提供了一系列工具、API和SDK，用于根据OpenTelemetry规范检测、生成、收集和导出遥测数据。
在撰写本文时，[OpenTelemetry Logging API][opentelemetry-logging]尚不稳定，Rust实现[尚不支持日志记录][opentelemetry-status.rs]，但支持跟踪API。

[opentelemetry.rs]: https://crates.io/crates/opentelemetry
[tracing-subscriber.rs]: https://docs.rs/tracing-subscriber/latest/tracing_subscriber/
[opentelemetry-logging]: https://opentelemetry.io/docs/reference/specification/status/#logging
[opentelemetry-status.rs]: https://opentelemetry.io/docs/instrumentation/rust/#status-and-releases
[tracing.rs]: https://crates.io/crates/tracing
[log.rs]: https://crates.io/crates/log
