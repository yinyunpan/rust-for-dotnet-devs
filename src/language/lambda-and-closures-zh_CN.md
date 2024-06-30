# Lambda和闭包

C#和Rust允许将函数用作第一类值，从而实现编写 _高阶函数_。
高阶函数本质上是接受其他函数作为参数以允许调用方参与被调用函数的代码。
在C#中，_类型安全函数指针_ 最常见的是`Func`和`Action`。
C#语言允许这些委托的临时实例是通过 _lambda表达式_ 创建。

Rust也有函数指针，`fn`类型是最简单的：

```rust
fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(|x| x + 1, 5);
    println!("The answer is: {}", answer); // 打印： The answer is: 12
}
```

However, Rust makes a distinction between _function pointers_ (where `fn`
defines a type) and _closures_: a closure can reference variables from its
surrounding lexical scope, but not a function pointer. While C# also has
[function pointers][*delegate] (`*delegate`), the managed and type-safe
equivalent would be a static lambda expression.

但是，Rust区分 _函数指针_（其中`fn`定义类型）和 _闭包_：闭包作为变量引用在围绕词法范围，但不是函数指针。
而C#也有[函数指针][*delegate]（`*delegate`），托管和类型安全等效的是一个静态lambda表达式。

  [*delegate]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-9.0/function-pointers

接受闭包的函数和方法是使用泛型类型编写的，这些泛型类型绑定到表示函数的特征之一：`Fn`、`FnMut`和`FnOnce`。
当需要值通过函数指针或闭包，Rust开发人员使用一个 _闭包表达式_（如上面的例子`|x| x + 1`），它转换与C#中的lambda表达式相同。
闭包表达式是创建函数指针还是闭包取决于关于闭包表达式是否引用其上下文。

当闭包从其环境中捕获变量时，所有权规则开始发挥作用，因为所有权最终以关闭而告终。
查看更多信息，请参阅“[将捕获的值移出闭包和Fn Traits][closure-move]“部分。

  [closure-move]: https://doc.rust-lang.org/book/ch13-01-closures.html#moving-captured-values-out-of-closures-and-the-fn-traits
