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

```python {data-line-numbers=1}
from mypy import api


code = """
import typing


def foo(x: typing.Any):
    pass

def should_pass():
    foo(1)
    foo("10")


def should_fail():
    foo(1, 2)
"""

from memory_profiler import profile


@profile
def run():
    for i in range(20):
        result = api.run(["-c", code])
        print(result)


run()
```

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

```python {monaco}
def collect_memory_stats() -> tuple[dict[str, int], dict[str, int]]:
    objs = gc.get_objects()
    find_recursive_objects(objs)

    inferred = {}
    for obj in objs:
        if type(obj) is FakeInfo:
            continue
        n = type(obj).__name__
        if hasattr(obj, "__dict__"):
            inferred[id(obj.__dict__)] = f"{n} (__dict__)"
        if isinstance(obj, (Node, Type)):
            if hasattr(obj, "__dict__"):
                for x in obj.__dict__.values():
                    if isinstance(x, list):
                        inferred[id(x)] = f"{n} (list)"
                    if isinstance(x, tuple):
                        inferred[id(x)] = f"{n} (tuple)"

            for k in get_class_descriptors(type(obj)):
                x = getattr(obj, k, None)
                if isinstance(x, list):
                    inferred[id(x)] = f"{n} (list)"
                if isinstance(x, tuple):
                    inferred[id(x)] = f"{n} (tuple)"

    freqs: dict[str, int] = {}
    memuse: dict[str, int] = {}
    for obj in objs:
        if id(obj) in inferred:
            name = inferred[id(obj)]
        else:
            name = type(obj).__name__
        freqs[name] = freqs.get(name, 0) + 1
        memuse[name] = memuse.get(name, 0) + sys.getsizeof(obj)

    return freqs, memuse
```