---
title: Select -- 无阻塞读写 channel
date: 2019-02-02 14:15:15
categories: 
tags:
- Go
---

## 通道阻塞

在之前的 [**Go 的并发模型**](https://leeif.me/2019/01/go-concurrent-model.html) 可以了解到，FAN 流水模型可以多个 Goroutine 读一个 Channel 中的数据(FAN-OUT)，或者多个 Chanel 将数据发送到一个 Goroutine 中接收(FAN-IN)，**但是无论是无缓冲通道，还是有缓冲通道，都存在阻塞的情况**

### 无缓冲通道

> 特点：发送的数据需要被读取后，发送才会完成

**阻塞场景：**

- 通道中无数据，但执行都通道
- 通道中无数据，向通道中写数据，但无其他协程读取该通道中的数据

**代码示例：**

```go
// 场景 1
func ReadNoDataFromNoBufCh(){
    noBufCh := make(chan int)
    // 通道中无数据
    <- noBufCh
    fmt.Println("read from no buffer channel success")

    // Output:
    // fatal error: all goroutines are asleep - deadlock!
}

// 场景 2
func WriteNoBufCh(){
    ch := make(chan int)
    ch <- 1
    fmt.Println("write success no block")

    // Output:
    // fatal error: all goroutines are asleep - deadlock!
}
```

### 有缓冲通道

> 特点：有缓存时可以向通道中写入数据后直接返回，缓存中有数据时可以从通道中读到数据直接返回，这时有缓存通道是不会阻塞的

**阻塞场景：**

- 通道的缓存无数据，但执行读通道
- 通道的缓存已经占满，向通道写数据，但无协程读

**代码示例：**

```go
// 场景 1
func ReadNoDataFromBufCh(){
    bufCh := make(chan int, 1)
    // 通道中无数据
    <- bufCh
    fmt.Println("read from no buffer channel success")

    // Output:
    // fatal error: all goroutines are asleep - deadlock!
}

// 场景 2
func WriteBufChButFull(){
    ch := make(chan int, 1)
    // 写数据，占满缓冲
    ch <- 1
    
    // 无协程读取缓冲中的数据，继续写，阻塞
    ch <- 2
    fmt.Println("write success no block")

    // Output:
    // fatal error: all goroutines are asleep - deadlock!
}
```

## Select 功能

**Select** 由关键字 `select` 和 `case` 组成，`default` 不是必须的，如果没其他事可做，可以省略 `default`，**在多个通道上进行读或写操作，让函数可以处理多个事情，但 1 次只处理 1 个**，有以下特性：

- 每次执行 `select`，都会只执行其中 1 个 `case` 或者执行 `default` 语句
- 当没有 case 或者 default 可以执行时，select 则阻塞，等待直到有 1 个 case 可以执行
- 当有多个 case 可以执行时，则随机选择 1 个 case 执行
- case 后面跟的必须是读或者写通道的操作，否则编译出错

