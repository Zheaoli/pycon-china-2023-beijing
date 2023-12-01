---
transition: fade-out
---

# 程序的可观测性手段

通常来说，程序的可观测性手段大致分为两类

<v-clicks>

- 用户态
- 内核态

</v-clicks>

---
transition: fade-out
---

# 程序的可观测性手段：用户态

以 Python 举例， 我们用户态的可观测性手段可以分为三类

<v-clicks>

- 内置埋点， 日志
- 旁路 REPL 
- 类 Debugger 工具

</v-clicks>

---
transition: fade-out
---

# 程序的可观测性手段：用户态-内置埋点

我们来看一段程序，感谢 @laike9m 提出的问题

<div class="grid grid-cols-2 gap-10 pt-4 -mb-6">

<div height=600>

<v-clicks>

![image](https://github.com/Zheaoli/pycon-china-2023-beijing/assets/7054676/0bce781e-5716-4eea-9787-9119af346648)

</v-clicks>
</div>

<v-click>

<div>

这段代码的问题在于，这一段很简单的程序，内存使用却异常的高

而我们这段程序不需要考虑复现的问题，所以我们可以考虑侵入式的埋点

常见的埋点手段有这样一些

- 日志
- 利用 gc，traceback 等内置的库
- Python 3.12 后，PEP 669 可以让我们更详细监控代码事件

</div>
</v-click>

</div>

---
transition: fade-out
monaco: true
---

# 程序的可观测性手段：用户态-内置埋点

对于前面有问题的代码，我利用 GC 来监控起内存对象的分配情况

<div class="grid grid-cols-2 gap-10 pt-4 -mb-6">

<v-click>

![image](https://github.com/Zheaoli/pycon-china-2023-beijing/assets/7054676/a6cace72-66b2-4826-8e81-137e53292a3b)

</v-click>

<v-click>

<div>

我们可以看到，在这段代码中我们通过使用 gc 模块获取内存中的对象，通过 InstanceID 来关联相关的对象。

</div>

</v-click>

</div>

---
transition: fade-out
---

# 程序的可观测性手段：用户态-Repl

我们通常还面临这样的一种情况

<div class="grid grid-cols-2 gap-10 pt-4 -mb-6">

<v-clicks>

- 需要跑较长时间才能复现
- 最好不要重启应用

</v-clicks>

<v-click>

<div>

在这种情况下，我们调试手段就没有直接侵入式注入代码来的灵活

通过预置 REPL，然后通过特定方式开启，可能会是我们的优先选择

</div>

</v-click>

</div>

---
transition: fade-out
---

# 程序的可观测性手段：用户态-Repl

就目前来讲，我们可以通过一些已有的线程的库，以及配合信号来完成这样的工作

<div class="grid grid-cols-2 gap-10 pt-4 -mb-6">

<v-clicks>

- python-manhole
- gevent REPL

</v-clicks>

<v-click>

<div>

还记得我们第一个例子吗？交互式 REPL 在调试过程中起到了很大的作用

</div>

</v-click>

</div>

---
transition: fade-out
---

# 程序的可观测性手段：用户态-类 Debugger 工具

除了 REPL 之外，我们还可以通过类 Debugger 工具来完成这样的工作


<v-clicks>

- py-spy
- pyrasite
- etc...

</v-clicks>


---
transition: fade-out
---

# 程序的可观测性手段：内核态

很多时候，我们无法单纯的通过用户态的手段来完成我们的信息获取的工作

让我们来看个例子

<div class="grid grid-cols-3 gap-10 pt-4 -mb-6">

```rust
use std::io::Read;
use std::fs::OpenOptions;

fn main() {
    let mut bs = vec![0; 1024 * 1024 * 64];
    let mut f = OpenOptions::new().read(true).open("/tmp/demofile2").unwrap();
    f.read_exact(&mut bs).unwrap();

}
```

```python
import pathlib
import timeit

root = pathlib.Path(__file__).parent
filename = "file"

def read_file_with_normal() -> bytes:
    with open("/tmp/demofile2", "rb") as fp:
        result = fp.read()
    return result

if __name__ == "__main__":
    read_file_with_normal()
```

两段代码，谁更快

</div>

---
transition: fade-out
---

# 程序的可观测性手段：内核态

我们来看看结果

<div class="grid grid-cols-3 gap-10 pt-4 -mb-6">

```text
Benchmark 1: python test_fs.py
  Time (mean ± σ):      1.191 s ±  0.060 s    [User: 0.019 s, System: 1.167 s]
  Range (min … max):    1.134 s …  1.302 s    10 runs

Benchmark 2: ./opendal-test/target/release/opendal-test
  Time (mean ± σ):      2.759 s ±  0.132 s    [User: 0.001 s, System: 2.751 s]
  Range (min … max):    2.654 s …  3.046 s    10 runs
```

Rust 的程序虽然比 Python 快，但是在内核态的时间上却比 Python 慢了很多

Why？

</div>

---
transition: fade-out
---

# 程序的可观测性手段：内核态

在最基础的结果上，我们需要对于整个过程中涉及到的 syscall 耗时进行精确测量。

<div class="grid grid-cols-2 gap-10 pt-4 -mb-6">

<v-click>

![image](https://github.com/Zheaoli/pycon-china-2023-beijing/assets/7054676/65c68eed-9420-465e-9569-0b7a8ea517e7)

</v-click>

<v-click>

我们利用 eBPF+tracepoint 来实现对于关键 syscall 的测量

</v-click>

</div>

---
transition: fade-out
---

# 程序的可观测性手段：内核态

通常来说，内核态的观测手段分为两种

1. 依托用户程序内置的一些静态埋点，在调用代码的时候触发相关回调（USDT 是典型的例子）
2. 依托内核的已有的基础设施如 kprobe，ftrace 等，进行内核态的观测
