# 基准测试

在Rust中运行基准测试是通过[`cargo bench`][cargo-bench]完成的，这是`cargo`的一个特定命令，它执行所有用 `#[bench]` 属性注释的方法。
此属性目前[不稳定][bench-unstable]，仅适用于夜间频道。

.NET 用户可以使用`BenchmarkDotNet`库来对方法进行基准测试并跟踪其性能。
`BenchmarkDotNet`的等效项是名为`Criterion`的包。

根据其[文档][criterion-docs]，`Criterion`会收集和存储每次运行的统计信息，并可以自动检测性能回归以及测量优化。

使用`Criterion`可以使用`#[bench]`属性，而无需转到夜间频道。

与`BenchmarkDotNet`一样，也可以将基准测试结果与[用于持续基准测试的GitHub Action][gh-action-bench] 集成。
事实上，`Criterion`支持多种输出格式，其中还有`bencher`格式，模仿夜间的`libtest`基准测试并与上述操作兼容。

[cargo-bench]: https://doc.rust-lang.org/cargo/commands/cargo-bench.html
[bench-unstable]: https://doc.rust-lang.org/rustc/tests/index.html#test-attributes
[criterion-docs]: https://bheisler.github.io/criterion.rs/book/index.html
[gh-action-bench]: https://github.com/benchmark-action/github-action-benchmark
