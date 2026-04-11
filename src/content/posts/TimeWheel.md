---
title: 层式结构-时间轮
pubDate: 2024-05-25
categories: ['Tech']
description: ''
---

后端开发常见层式结构：时间轮、跳表、LSM-Tree

1. 海量并发的定时任务：时间轮
2. 高并发读写的有序结构组织：跳表
3. 空间利用率以及写性能高的磁盘数据组织：LSM-Tree

> 什么是层式结构（GPT）：层式结构（Layered Structure）在计算机科学和软件工程中通常指的是将系统分成若干层次，每个层次负责不同的功能和任务。这样设计的好处是可以将复杂系统的不同部分进行解耦和模块化，从而提高系统的可维护性、可扩展性和可理解性。

## 时间轮

### 单层级时间轮

![photo](./_assets/TimeWheel/Snipaste_2024-05-25_20-10-14.png)

定时任务是用时间轮进行实现的，那么它是如何去组织数据的呢？

- 一个格子代表一个时刻
- 一个格子可存储多个任务
- 按执行顺序组织数据

### 多层级时间轮

![photo](./_assets/TimeWheel/Snipaste_2024-05-25_20-17-25.png)

按照任务的**轻重缓急**来进行层次划分的，当我们的任务是在秒这个单位下需要执行的，那么只需要放在前60秒即可，那么如果任务是分、时单位下，那么只需要放在对应的层级即可。对比单层级时间轮，多层级时间轮可以**减少比较的次数**，因此可以提升性能。（避免任务的轮询）

### 时间轮如何运行

多层级时间轮是如何进行运作的呢？

![photo](./_assets/TimeWheel/Snipaste_2024-05-25_20-27-14.png)

当我们的秒针走完了60秒之后，分针相对于应该移动一格，这时候需要将分钟对应时刻的任务，映射到第一层级中。同理如果移动到时针，那么将任务，映射到分针当中。

### 怎么设计时间轮？

1. 设计最小时间精度； 相当于是允许的定时任务误差
2. 设计最大的时间范围；超过时间范围会出现错误
3. 设计最大的层级；层级的个数决定了映射的频繁程度（消耗时间）

### 为什么使用时间轮

海量**并发**的定时任务，也就是说我们要找到一个适合在多线程环境中使用的数据结构，在并发中必须考虑的就是**加锁**的问题

```tex
lock()
操作数据结构
unlock()
```

为什么说时间轮在多线程的环境下效率高呢？主要体现**操作数据结构时间足够的短**。需要考虑锁的粒度，锁的粒度就是操作数据结构的时间。

减小锁粒度有以下方法

1. 较小的时间复杂度
2. 更细粒度上加锁（例如说在各个节点上加锁）

**时间轮就是拥有较小的时间复杂度**，同理任务队列也是一个效率很高的，它选用的就是在**更细粒度上加锁**

### 实现一个单层级的时间轮定时任务

- 首先创建`TaskElement`代表任务

  - 任务函数
  - 所在位置
  - 循环次数
  - 键

- `TimeWheel`实现

  简单的来说，就是将时间划分为多个槽（slots），每个槽内存放任务队列。

  - 初始化：创建指定数量的槽，每个槽用一个双向链表表示，初始化调度器（`ScheduledExecutorService`），用于定期触发时间轮的`tick`操作。
  - 执行`tick`该方法获取当前位置上的任务队列进行执行。
  - `execute`执行函数，主要根据`TaskElement`中的`cycle`属性判断是否应该执行。

  添加一个任务

  - 首先需要先计算，任务的执行位置和循环次数，使用`getPosAndCycle`进行计算
  - 使用返回的执行位置和循环次数，实例化`TaskElement`。

  删除一个任务

  - 只需要根据`TaskElement`中的`key`即可

![photo](./_assets/TimeWheel/Snipaste_2024-05-26_00-11-55.png)
![photo](./_assets/TimeWheel/Snipaste_2024-05-26_00-16-24.png)

[代码实现](https://github.com/AnnularLabs/java-timewheel)

### 代码解析

#### Duration

Duration 是 Java 8 引入的时间类，位于 java.time 包中。它表示两个瞬时时间点之间的时间量。Duration 类提供了一些方法，用于创建、操作和获取时间段的信息。

常用方法

- Duration.ofDays(long days): 创建指定天数的 Duration 实例。
- Duration.ofHours(long hours): 创建指定小时数的 Duration 实例。
- Duration.ofMinutes(long minutes): 创建指定分钟数的 Duration 实例。
- Duration.ofSeconds(long seconds): 创建指定秒数的 Duration 实例。
- Duration.ofMillis(long millis): 创建指定毫秒数的 Duration 实例。
- Duration.between(Temporal startInclusive, Temporal endExclusive): 通过两个时间点之间的差异创建 Duration。
- Duration.parse(CharSequence text): 解析标准ISO-8601格式的 Duration 字符串。

#### scheduleAtFixedRate

scheduler.scheduleAtFixedRate(this::tick, interval.toMillis(), interval.toMillis(), TimeUnit.MILLISECONDS); 这一行代码的作用是使用调度器定期执行 tick 方法。具体来说，这是一个基于固定速率调度的定时任务，每隔指定的时间间隔执行一次 tick 方法。

scheduleAtFixedRate 方法有四个参数：

- command: 要执行的任务，这里是 this::tick。
- initialDelay: 第一次执行任务前的延迟时间，这里是 interval.toMillis() 毫秒。
- period: 连续执行任务之间的周期时间，这里也是 interval.toMillis() 毫秒。
- unit: 时间单位，这里是 TimeUnit.MILLISECONDS。

scheduleAtFixedRate(this::tick, interval.toMillis(), interval.toMillis(), TimeUnit.MILLISECONDS) 的工作流程如下：

1. 初始延迟: 在 interval.toMillis() 毫秒后开始第一次执行 tick 方法。
2. 固定速率执行: 每隔 interval.toMillis() 毫秒重复执行 tick 方法。
