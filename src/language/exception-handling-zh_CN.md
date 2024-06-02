# 异常处理

在.NET中，异常是继承自[`System.Exception`][net-system-exception]类。
如果代码段中出现问题，则引发异常。引发的异常将传递到堆栈中直到应用程序处理它或程序终止。

Rust没有异常，而是区分 _可恢复_ 和 _不可恢复_ 错误。
可恢复错误表示应该报告但程序仍继续运行的问题。
操作结果失败的可恢复错误属于[`Result<T, E>`][rust-result] 类型，其中`E`是错误变体的类型。
当程序遇到不可恢复的错误时，[`panic!`][panic] 宏会停止执行。不可恢复的错误始终是漏洞症状。

## 自定义错误类型

在.NET中，自定义异常源自`Exception`类。
有关[如何创建用户定义异常][net-user-defined-exceptions] 的文档提到了以下示例：

```csharp
public class EmployeeListNotFoundException : Exception
{
    public EmployeeListNotFoundException() { }

    public EmployeeListNotFoundException(string message)
        : base(message) { }

    public EmployeeListNotFoundException(string message, Exception inner)
        : base(message, inner) { }
}
```

在Rust中，可以通过实现[`Error`][rust-std-error]特征来实现对错误值的基本期望。
Rust中最小的用户定义错误实现是：

```rust
#[derive(Debug)]
pub struct EmployeeListNotFound;

impl std::fmt::Display for EmployeeListNotFound {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        f.write_str("Could not find employee list.")
    }
}

impl std::error::Error for EmployeeListNotFound {}
```

Rust中的`Error::source()`方法相当于.NET的`Exception.InnerException`属性。
但是不需要为`Error::source()` 提供实现，统一（默认）实现将返回`None`。

## 引发异常

要在C#中引发异常，请抛出异常的一个实例：

```csharp
void ThrowIfNegative(int value)
{
    if (value < 0)
    {
        throw new ArgumentOutOfRangeException(nameof(value));
    }
}
```

对于Rust中的可恢复错误，从方法返回`Ok`或`Err`变体：

```rust
fn error_if_negative(value: i32) -> Result<(), &'static str> {
    if value < 0 {
        Err("Specified argument was out of the range of valid values. (Parameter 'value')")
    } else {
        Ok(())
    }
}
```

[`panic!`][panic] 宏会产生无法恢复的错误：

```rust
fn panic_if_negative(value: i32) {
    if value < 0 {
        panic!("Specified argument was out of the range of valid values. (Parameter 'value')")
    }
}
```

## 错误传播

在.NET中，异常会沿堆栈向上传递，直到被处理或程序终止。
在Rust中，不可恢复的错误表现类似，但处理它们并不常见。

但是，可恢复错误需要明确传播和处理。
它们的存在始终由Rust函数或方法签名指示。
捕获异常允许您根据C#中错误的存在与否采取行动：

```csharp
void Write()
{
    try
    {
        File.WriteAllText("file.txt", "content");
    }
    catch (IOException)
    {
        Console.WriteLine("Writing to file failed.");
    }
}
```

在Rust中，这大致相相当于：

```rust
fn write() {
    match std::fs::File::create("temp.txt")
        .and_then(|mut file| std::io::Write::write_all(&mut file, b"content"))
    {
        Ok(_) => {}
        Err(_) => println!("Writing to file failed."),
    };
}
```

通常，可恢复错误只需传播，而不需要处理。
为此，方法签名需要与传播错误的类型兼容。
[`?`运算符][question-mark-operator]以符合人体工程学的方式传播错误：

```rust
fn write() -> Result<(), std::io::Error> {
    let mut file = std::fs::File::create("file.txt")?;
    std::io::Write::write_all(&mut file, b"content")?;
    Ok(())
}
```

**注意**：要使用问号运算符传播错误，错误实现需要 _兼容_，如[_传播错误的快捷方式_][propagating-errors-rust-book]中所述。
最通用的“兼容”错误类型是错误[特征对象]`Box<dyn Error>`。

## 堆栈跟踪

在.NET中抛出未处理的异常将导致运行时打印堆栈跟踪，从而允许使用其他上下文调试问题。

对于Rust中不可恢复的错误，[`panic!` Backtraces][panic-backtrace] 提供了类似的行为。

稳定版Rust中的可恢复错误尚不支持Backtraces，但目前在实验性Rust中使用 [provide方法]支持它。

[net-system-exception]: https://learn.microsoft.com/en-us/dotnet/api/system.exception?view=net-6.0
[rust-result]: https://doc.rust-lang.org/std/result/enum.Result.html
[panic-backtrace]: https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#using-a-panic-backtrace
[net-user-defined-exceptions]: https://learn.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions
[rust-std-error]: https://doc.rust-lang.org/std/error/trait.Error.html
[provide method]: https://doc.rust-lang.org/std/error/trait.Error.html#method.provide
[question-mark-operator]: https://doc.rust-lang.org/std/result/index.html#the-question-mark-operator-
[panic]: https://doc.rust-lang.org/std/macro.panic.html
[propagating-errors-rust-book]: https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
[trait object]: https://doc.rust-lang.org/reference/types/trait-object.html
