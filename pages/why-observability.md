---
transition: fade-out
image-layout: image-right
image: https://github.com/Zheaoli/pycon-china-2023-beijing/assets/7054676/eee7e3f7-1e00-4988-8646-ecb9722e816f
---

# 为什么我们需要强调程序的可观测性

为了更好的~~拉磨~~赚钱.jpg

<v-clicks>

1. 程序复杂度的日益提升
2. 对于问题的快速定位的要求
3. 生产上各种玄学的问题
    1. 我的进程去哪里了
    2. 我的进程被谁杀了
    3. 我的请求为什么这么慢
4. 所以小朋友你一定有很多的问号

</v-clicks>

---
transition: fade-out
---

# 为什么我们需要强调程序的可观测性

<h2>我猜你们肯定没直观的体感，那么我们来看点真实的案例</h2>

---
transition: fade-out
---

# 为什么我们需要强调程序的可观测性

<h2>案例一</h2>

假设我们有一个 Python 程序，正在经历如下的现象

<v-clicks>

- 通过 Gunicorn 启动一个 Flask based 的 Web Server
- 你发现这个服务有一定概率请求无法响应
- 进程经常会被杀死

</v-clicks>

<v-click>

所以你该怎么办？

</v-click>

---
transition: fade-out
---

# 为什么我们需要强调程序的可观测性

<h2>案例二</h2>

感谢 @yihong0618 大哥曾经给的灵感

假设你有一个链接 PGSQL 的服务，你又在经历如下的困惑

<v-clicks>

- 到 PGSQL 服务器延迟没有异常，但是你的查询时不时会玄学超时
- 你本地的 CPU 时不时的玄学过山车
- 你想知道在一个 SQL 从查询到返回的过程中，发生了什么，经历了哪些函数

</v-clicks>

---
transition: fade-out
---

# 为什么我们需要强调程序的可观测性

不知道你们发现没有，不知不觉中我已经把 Python 程序的可观测性变成程序的可观测性了

<v-clicks>

- 是的，我们今天会继续以 Python 作为样例来讨论一些可观测性的手段和思路（
- 本人擅长挂羊头卖狗肉

</v-clicks>
