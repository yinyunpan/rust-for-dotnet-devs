# 环境和配置

## 访问环境变量

.NET通过`System.Environment.GetEnvironmentVariable`方法提供对环境变量的访问。
此方法在运行时检索环境变量的值。

```csharp
using System;

const string name = "EXAMPLE_VARIABLE";

var value = Environment.GetEnvironmentVariable(name);
if (string.IsNullOrEmpty(value))
    Console.WriteLine($"Variable '{name}' not set.");
else
    Console.WriteLine($"Variable '{name}' set to '{value}'.");
```

Rust通过`std::env`模块中的`var`和`var_os`函数提供在运行时访问环境变量的相同功能。

`var`函数返回`Result<String, VarError>`，如果设置则返回变量，如果未设置变量或变量不是有效的Unicode则返回错误。

`var_os`具有不同的签名，返回`Option<OsString>`，如果设置了变量则返回某个值，如果未设置变量则返回None。
`OsString`不需要是有效的Unicode。

```rust
use std::env;


fn main() {
    let key = "ExampleVariable";
    match env::var(key) {
        Ok(val) => println!("{key}: {val:?}"),
        Err(e) => println!("couldn't interpret {key}: {e}"),
    }
}
```

```rust
use std::env;

fn main() {
    let key = "ExampleVariable";
    match env::var_os(key) {
        Some(val) => println!("{key}: {val:?}"),
        None => println!("{key} not defined in the enviroment"),
    }
}
```

Rust还提供了在编译时访问环境变量的功能。`std::env`中的`env!`宏在编译时扩展变量的值，返回`&'static str`。如果未设置变量，则会发出错误。

```rust
use std::env;

fn main() {
    let example = env!("ExampleVariable");
    println!("{example}");
}
```

在.NET中，可以通过[源生成器][source-gen]以不太直接的方式实现对环境变量的编译时访问。

[source-gen]: https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview

## 配置

.NET中的配置可通过配置提供程序实现。该框架通过`Microsoft.Extensions.Configuration`命名空间和NuGet包提供多种提供程序实现。

配置提供程序使用不同的来源从键值对中读取配置数据，并通过`IConfiguration`类型提供配置的统一视图。

```csharp
using Microsoft.Extensions.Configuration;

class Example {
    static void Main()
    {
        IConfiguration configuration = new ConfigurationBuilder()
            .AddEnvironmentVariables()
            .Build();

        var example = configuration.GetValue<string>("ExampleVar");

        Console.WriteLine(example);
    }
}
```

其他提供程序示例可在官方文档[.NET中的配置提供程序][conf-net]中找到。

通过使用第三方包（例如[figment]或[config]），Rust中也提供类似的配置体验。

请参阅以下使用[config]包的示例：

```rust
use config::{Config, Environment};

fn main() {
    let builder = Config::builder().add_source(Environment::default());

    match builder.build() {
        Ok(config) => {
            match config.get_string("examplevar") {
                Ok(v) => println!("{v}"),
                Err(e) => println!("{e}")
            }
        },
        Err(_) => {
            // something went wrong
        }
    }
}

```

[conf-net]: https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration-providers
[figment]: https://crates.io/crates/figment
[config]: https://crates.io/crates/config
