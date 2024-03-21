# 简介

这是一本（非全面）指南适用于对Rust编程语言完全陌生的C#和.NET开发人员。一些概念和结构
在C#/.NET和Rust之间转换得相当好，会有少许表达方式
不同，而另外一些完全不同，例如内存管理。
本指南通过简单示例对这些概念和结构做简明的比较和对照。

本指南的原作者本身就是C#/.NET开发人员对Rust完全陌生。本指南收集
作者在编写Rust代码几个月过程中获得的知识。这是作者在开始他们的Rust之旅时希望拥有的指南
。作者鼓励你读Rust及术语的书和网络上其他材料，而不是试图完全通过C#和.NET的视角来学习它。
同时，本指南可以帮助快速回答一些问题，例如：Rust是否支持继承、线程、异步编程等等？

假设：

- 读者是一位经验丰富的 C#/.NET 开发人员。
- 读者对Rust完全陌生。

目标：

- Provide a brief comparison and mapping of various C#/.NET topics to their
  counterparts in Rust.
- Provide links to Rust reference, book and articles for further reading on
  topics.

Non-goals:

- Discussion of design patterns and architectures.
- Tutorial on the Rust language.
- Reader is proficient in Rust after reading this guide.
- While there are short examples that contrast C# and Rust code for some
  topics, this guide is not meant to be a cookbook of coding recipes in the
  two languages.

---
[^authors]: The original authors of this guide were (in alphabetical order):
[Atif Aziz], [Bastian Burger], [Daniele Antonio Maggio], [Dariusz Parys] and
[Patrick Schuler].

  [Atif Aziz]: https://github.com/atifaziz
  [Bastian Burger]: https://github.com/bastbu
  [Daniele Antonio Maggio]: https://github.com/danigian
  [Dariusz Parys]: https://github.com/dariuszparys
  [Patrick Schuler]: https://github.com/p-schuler
