---
title: Go 并发模型
date: 2019-01-22 14:15:15
categories: 
tags:
- Go
---

## 前言

> Go 语言是为并发而生的语言，Go 语言是为数不多的在语言层面实现并发的语言；也正是 Go 语言的并发特性，吸引了全球无数的开发者

## 并发 (concurrency) 和并行(parallellism)

在了解 **Go** 的并发原理之前，先了解什么是并发什么是并行；

- 并发 **( concurrency )**

  两个或两个以上的任务在一段时间内被执行。我们不必 care 这些任务在某一个时间点是否是同时执行，可能同时执行，也可能不是，我们只关心在一段时间内，哪怕是很短的时间（一秒或者两秒）是否执行解决了两个或两个以上任务

- 并行  **( parallellism )**

  两个或两个以上的任务在同一时刻被同时执行

并发说的是逻辑上的概念，而并行，强调的是物理运行状态；并发 “包含” 并行；( 详情请见：**Rob Pike** 的 [PPT](https://talks.golang.org/2012/concurrency.slide#1))

## CSP 并发模型

Go 实现了两种并发形式。第一种是大家普遍认知的：多线程共享内存。其实就是 Java 或者 C++ 等语言中的多线程开发。另外一种是 Go 语言特有的，也是 Go 语言推荐的：`CSP`（communicating sequential processes）并发模型

请记住下面这句话：

> **Do not communicate by sharing memory; instead, share memory by communicating.**
> “不要以共享内存的方式来通信，相反，要通过通信来共享内存”

普通的线程并发模型，就是像 Java、C++、或者 Python，他们线程间通信都是通过共享内存的方式来进行的。非常典型的方式就是，在访问共享数据（例如数组、Map、或者某个结构体或对象）的时候，通过锁来访问，因此，在很多时候，衍生出一种方便操作的数据结构，叫做 “线程安全的数据结构”

Go 的 CSP 并发模型，是通过`goroutine`和`channel`来实现的。

- `goroutine:` 是 Go 语言中并发的执行单位，有点抽象，其实就是和传统概念上的`线程 `类似，但它比线程更为轻量，称之为`协程`
- `channel：` 是 Go 语言中各个并发结构体 (`goroutine`) 之前的通信机制。 通俗的讲，就是各个`goroutine`之间通信的” 管道 “，有点类似于 Linux 中的管道

创建一个 `goroutine` 很简单，只要使用 `go` 关键字就可以了，如下；

```go
go func()
```

通信机制 `channel` 也很方便，传数据用 `channel <- data` ，取数据用 `<-channel`

在通信过程中，传数据 `channel <- data` 和取数据 `<-channel` 必然会成对出现，因为这边传，那边取，两个`goroutine`之间才会实现通信，而且不管传还是取，必阻塞，直到另外的`goroutine`传或者取为止

<img src="https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/go-cps.png">

这便是 `Golang CSP` 并发模型最基本的形式，本问内容不详细阐述并发原理

## 并发模型的运用

### 流水线模型

>Golang 并发核心思路是关注数据流动。数据流动的过程交给 channel，数据处理的每个环节都交给 goroutine，把这些流程画起来，有始有终形成一条线，那就能构成流水线模型

流水线并不是什么新奇的概念，它能极大的提高生产效率，在当代社会流水线非常普遍，我们用的几乎任何产品（手机、电脑、汽车、水杯），都是从流水线上生产出来的；**在 Golang 中，流水线由多个阶段组成，每个阶段之间通过 channel 连接，每个节点可以由多个同时运行的 goroutine 组成**

![](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/20190124110810.png)

如上图，从最简单的流水线入手，由 3 个阶段组成，分别是 A、B、C；第一个阶段的协程是**生产者**，它们只生产数据，最后一个阶段的协程是**消费者**，A 是生成者，C 是消费者，而 B 只是中间过程的处理者；A 和 B 之间是通道`aCh`，B 和 C 之间是通道`bCh`，A 生成数据传递给 B，B 生成数据传递给 C

> 举个例子，设计一个程序：计算一个整数切片中元素的平方值并把它打印出来。非并发的方式是使用 for 遍历整个切片，然后计算平方，打印结果

我们使用流水线模型实现这个简单的功能，从流水线的角度，可以分为 3 个阶段：

1. 遍历切片，这是生产者。
2. 计算平方值。
3. 打印结果，这是消费者。

具体代码，参考 [**simple.go**](https://github.com/leeifme/Go-test/blob/master/go-concurrent/assembly-line/simple.go)

- `producer()`负责生产数据，它会把数据写入通道，并把它写数据的通道返回
- `square()`负责从某个通道读数字，然后计算平方，将结果写入通道，并把它的输出通道返回
- `main()`负责启动 producer 和 square，并且还是消费者，读取 suqre 的结果，并打印出来

**流水线的特点**

1. 每个阶段把数据通过 channel 传递给下一个阶段
2. 每个阶段要创建 1 个 goroutine 和 1 个通道，这个 goroutine 向里面写数据，函数要返回这个通道
3. 有 1 个函数来组织流水线，我们例子中是 main 函数

### 流水线 FAN 模式

> 流水线模型进阶，FAN-IN 和 FAN-OUT 模式，FAN 模式可以让我们的流水线模型更好的利用 Golang 并发，提高软件性能

这里还是以生产汽车的流水线为例，汽车生产线上有个阶段是给小汽车装 4 个轮子，可以把这个阶段任务交给 4 个人同时去做，这 4 个人把轮子都装完后，再把汽车移动到生产线下一个阶段；这个过程中，就有任务的分发，和任务结果的收集；**其中任务分发是 FAN-OUT，任务收集是 FAN-IN**

- **FAN-OUT 模式：多个 goroutine 从同一个通道读取数据，直到该通道关闭；**OUT 是一种张开的模式，所以又被称为扇出，可以用来分发任务
- **FAN-IN 模式：1 个 goroutine 从多个通道读取数据，直到这些通道关闭；**IN 是一种收敛的模式，所以又被称为扇入，用来收集处理的结果

如下图所示，

![](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/20190124112954.png)

依然延用上面的案例需求，计算一个整数切片中元素的平方值并把它打印出来，这次我们修改一下，添加 `merge()` 方法，具体代码参考 [**fan.go**](https://github.com/leeifme/Go-test/blob/master/go-concurrent/assembly-line/fan.go)

- `producer()`保持不变，负责生产数据
- `squre()`也不变，负责计算平方值
- 修改`main()`，启动 3 个 square，这 3 个 squre 从 producer 生成的通道读数据，**这是 FAN-OUT**
- 增加`merge()`，入参是 3 个 square 各自写数据的通道，给这 3 个通道分别启动 1 个协程，把数据写入到自己创建的通道，并返回该通道，**这是 FAN-IN**