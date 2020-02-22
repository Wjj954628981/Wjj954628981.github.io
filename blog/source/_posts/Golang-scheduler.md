---
title: Golang调度模型
date: 2019-07-31 18:50:16
tags: [Golang]
---

协程可谓是Golang的精髓，了解熟悉Golang协程的调度在Golang学习过程中非常重要。

<!--more-->

# Golang scheduler

**线程模型**
- 第一种是N：1，一个用户空间线程在一个OS线程上运行。可以快速上下文切换，但无法利用多核系统的优点；
- 第二种是1:1，一个执行线程与一个OS线程匹配。利用了机器上所有核心，但需要陷入内核实现上下文切换所以很慢。
- Golang尝试M：N调度程序充分利用上述两种方式，将任意数量的Goroutine调度到任意数量的线程。实现快速上下文切换并充分利用系统中所有核心，但增加了调度程序的复杂性。

**Golang实现调度，使用了以下三种实体：**
{% asset_img GMP.jpg %}

> - M代表OS线程
> - G代表一个Goroutine，包括堆栈、指令指针和其他对调度Goroutine重要的信息
> - P（P for processor）代表调度的上下文，这是我们从N：1调度转到M：N调度的重要部分

**以下是调度程序的稳定状态**
{% asset_img in-motion.jpg %}

**破坏上述稳定状态的情况：**
- syscall：
{% asset_img syscall.jpg %}
- steal：
{% asset_img steal.jpg %}