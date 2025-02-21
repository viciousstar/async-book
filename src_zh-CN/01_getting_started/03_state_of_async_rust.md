# 异步 Rust 编程目前状态

一部分异步编程设施已经有了和同步编程一样的稳定性保证，另外的部分仍在发展和变化之中。有了异步 Rust，你可以预见：

- 为典型的并发负载提供优秀的运行时性能
- 更频繁地与先进的语言特性交互，例如生命周期（lifetime）和固定（pinning）
- 一些兼容性保证，例如同步和异步代码之间，以及不同异步运行时之间的兼容性。
- 更高的维护负担，因为异步运行时和语言支持都在持续演进。

简而言之，异步 Rust 相比同步 Rust 更难使用，并且可能导致更高维护负担，但是会给你一流性能作为回报。异步 Rust 的所有地方都在持续提高，所以这些问题的影响会随着时间慢慢消退。

## 语言和库支持

尽管 Rust 自身提供了异步编程支持，大部分异步应用基于社区库（community crates）提供的功能。因此，你需要需要同时依靠语言特性和库支持：

- 最基础的 traits、类型（types）和函数（functions）, 例如 [`Future`](https://doc.rust-lang.org/std/future/trait.Future.html) trait，由标准库提供。
- `async/await` 语法由 Rust 编译器直接支持。
- 很多工具类型、宏和函数由 [`futures`](https://docs.rs/futures/) 库提供。他们可以用在任何异步 Rust 应用
- 异步代码的执行、IO 和任务生成均由 “异步运行时” 提供支持，例如 Tokio 和 async-std. 多数异步应用，和一些异步库，都只依赖于一个特定的运行时，详情参见[“异步生态”](../08_ecosystem/00_chapter.md)

有一些语言特性，读者也许在同步 Rust 编程中很习惯了，但是在异步 Rust 中不可用。要补充说明的是，Rust 不允许你在 trait 中声明异步函数。作为代替，你需要用一些方法绕过以实现相同结果，但是会显得更啰嗦。

## 编译与调试

多数情况下，编译器的和运行时的错误在异步 Rust 中以和在 Rust 中相同的方式工作，，但是会有以下值得注意的区别：

### 编译错误

异步 Rust 中的编译错误遵循和同步 Rust 一样的高标准，但因为异步 Rust 通常依赖于更复杂的语言特性，例如生命周期和固定（pinning），你可能更频繁遇到这些错误。

### 运行时错误

无论编译器在什么时候看到一个异步函数，它会在底层生成一个状态机。异步 Rust 中的堆栈追踪通常会包含这些状态机的细节，以及来自运行时的函数调用。因此对照堆栈追踪信息也会相比同步 Rust 更可能需要关注。

### 新失败模式

好几种新的失败模式可以在异步 Rust 中使用。举例来说，如果你从异步上下文中调用一个阻塞函数，或者检查是否正确地实现了 `Future` trait。这些错误会静默传递到编译器，有时会甚至会传到单元测试中。对这些底层概念形成扎实充分的理解，也是这本书设定的目标，也会帮助你避开这些坑。

## 兼容性考虑

异步的和同步的代码不总是能自由地结合在一起。例如，你不能直接在同步函数里直接调用一个异步函数。同步的和异步的代码倾向于不同的设计模式，会使整合为不同环境设计的代码很困难。

甚至，异步代码之间也不总是能自由地结合在一起。一些库依赖于特定运行时来提供功能。如此，它通常会在库的依赖列表中指定。

这些兼容性问题会限制你权衡，所以要尽早调查要使用哪个异步运行时和那些库。一旦以决定好了运行时，你就不需要太关心兼容性的问题了。

## 性能特征
异步Rust的性能取决于您所使用的异步运行时的实现。尽管为异步Rust应用程序提供支持的运行时相对较新，但对于大多数实际工作负载而言，它们仍然表现出色。

也就是说，大多数异步生态系统都采用 _多线程_ 运行时。这使得难以获取单线程异步应用程序的理论性能优势，即便宜的同步。另一个被忽略的用例是对 _延迟敏感的任务_，这些任务对于驱动程序，GUI应用程序等非常重要。此类任务取决于运行时和/或OS支持，以便进行适当的调度。您可以期望将来会为这些用例提供更好的库支持。
