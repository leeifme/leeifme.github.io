---
title: go pprof 性能分析
date: 2019-05-20 14:15:15
description: "线上运行的基础平台文件管理服务进程出现内存泄露的现象，下图是 `grafana` 针对该服务的监控指标情况，可以发现服务刚起时，内存使用量为 **20M** 左右，经过操作后，内存会稳定在 **300M** 左右，不会持续上升，也不会下降，一开始找不到原因，所以尝试使用一下 golang pprof 性能分析工具分析一下程序到底哪出问题了"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
tocLevels: ["h2", "h3", "h4"]
author: 睡沙发の沙皮狗
authorEmoji: 👺
categories: 
tags:
- Go
---

## 起因

{{< notice warning >}} 
线上运行的基础平台文件管理服务进程出现内存泄露的现象，下图是 `grafana` 针对该服务的监控指标情况，可以发现服务刚起时，内存使用量为 **20M** 左右，经过操作后，内存会稳定在 **300M** 左右，不会持续上升，也不会下降，一开始找不到原因，所以尝试使用一下 golang pprof 性能分析工具分析一下程序到底哪出问题了
{{< /notice >}}

![](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/005TUXkXly1g3jdrwfy0zj31hc0r3tcp.jpg)

![](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/005TUXkXly1g3jdrwawt6j31hc0r3dhp.jpg)

## 添加 pprof 模块

现在最新版本的 go tool 分析工具已经很人性化了，pprof 采样数据主要有三种获取方式:

-   **runtime/pprof**: 手动调用`runtime.StartCPUProfile`或者`runtime.StopCPUProfile`等 API 来生成和写入采样文件，灵活性高，适用于`应用程序`
-   **net/http/pprof**: 通过 http 服务获取 Profile 采样文件，简单易用，适用于对应用程序的整体监控，通过 runtime/pprof 实现，适用于`web服务程序、服务进程`
-   **go test**: 通过 `go test -bench . -cpuprofile prof.cpu`生成采样文件 适用对函数进行针对性测试

{% pullquote right @leeifme , 睡沙发的沙皮狗 %} 

其实 net/http/pprof 中只是使用 runtime/pprof 包来进行封装了一下，并在 http 端口上暴露出来，让我们可以在浏览器查看程序的性能分析。可以自行查看 net/http/pprof 中代码，只有一个文件 pprof.go。

{% endpullquote %}

以上获取方式就不详细演示了，毕竟着重于解决当下问题，由于所要分析的服务程序依赖于 `gin web 框架` ，因此要在 `gin` 中集成 pprof；

**Example：**

```go
package main

import (
	"github.com/gin-contrib/pprof"  // step 1
	"github.com/gin-gonic/gin"
)

func main() {
  router := gin.Default()
  pprof.Register(router)   // step 2
  router.Run(":8080")
}
```

## 分析

启动程序，通过服务端口即可访问 pprof 的数据

查看当前总览：访问 `http://$HOSTIP:$PORT/debug/pprof`

![](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/005TUXkXly1g3ic456o71j313m0f70tl.jpg)

```
cpu（CPU Profiling）: $HOST/debug/pprof/profile，默认进行 30s 的 CPU Profiling，得到一个分析用的 profile 文件
block（Block Profiling）：$HOST/debug/pprof/block，查看导致阻塞同步的堆栈跟踪
goroutine：$HOST/debug/pprof/goroutine，查看当前所有运行的 goroutines 堆栈跟踪
heap（Memory Profiling）: $HOST/debug/pprof/heap，查看活动对象的内存分配情况
mutex（Mutex Profiling）：$HOST/debug/pprof/mutex，查看导致互斥锁的竞争持有者的堆栈跟踪
threadcreate：$HOST/debug/pprof/threadcreate，查看创建新 OS 线程的堆栈跟踪
```

这里，我更多的是做程序的内存分析，并通过交互式终端使用；

在 terminal 中使用 `go tool pprof http://$HOSTIP:$PORT/debug/pprof/heap` 可以进入 pprof 分析工具，比如输入 `top` 可以显示靠前的几项，go tool pprof 可以带上参数 `-inuse_space` (分析应用程序的常驻内存占用情况) 或者 `-alloc_space` (分析应用程序的内存临时分配情况)

现在 go tool 可以直接可视化结果，只需要带上 `-http=:8081` 参数即可，如：

```sh
$ go tool pprof -http=:8081 http://$HOSTIP:$PORT/debug/pprof/heap
```

之后就会在浏览器弹出 `http://$HOSTIP:8081/ui`，里面包含程序内存分析的 dot 格式的图、火焰图、top 列表、source 列表等，如下：



![](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/005TUXkXly1g3je0lzugdj31hc0r3q3u.jpg)



![](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/005TUXkXly1g3je0m0bjuj31hc0r3aby.jpg)



![](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/005TUXkXly1g3je0m0mu5j31hc0r341g.jpg)



![](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/005TUXkXly1g3je0m09v2j31hc0r30uc.jpg)

## 优化

内存消耗停滞在一个值时，比如上述问题描述，其实不用称之为内存泄漏，而是不主动 GC，需要主动释放内存；

导致这个问题的原因是由于上传文件时，采用 `multipart/form-data` 传输数据，`r.FormFile("file")` 将导致调用Request.ParseMultipartForm()，并将32 MB用作maxMemory参数的值，创建 32M 的缓冲区；由于`bytes.Buffer` 用于读取内容，因此读取过程将从一个小的或空的缓冲区开始，并在需要更大的时候重新分配；

有关详细信息，请参阅 [**multipart.Reader.ReadFrom()**](https://golang.org/pkg/mime/multipart/#Reader.ReadForm) 的实现。

```go
r.ParseMultipartForm(32 << 20) // 32 MB
file, _, err := r.FormFile("file")
// ... rest of your handler
```

{% pullquote @leeifme , 睡沙发的沙皮狗 %} 

详细描述可以参考 **stackoverflow** 上的回答，[Multipart form uploads + memory leaks in golang?](https://stackoverflow.com/questions/30657454/multipart-form-uploads-memory-leaks-in-golang)

{% endpullquote %}



这里优化的方案是在 `request`请求的 `body` 中只放文件数据，其余信息放到 `header` 中，这样就不需要使用 MultipartForm 去解析数据；

```go
file = r.Body
// ... rest of your handler
```

可以看到传输完成后，内存占用恢复到了服务初始的状态值

![](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/20190627151301.png)

## 参考

[Golang 大杀器之性能剖析 PProf](https://github.com/EDDYCJY/blog/blob/master/golang/2018-09-15-Golang%20%E5%A4%A7%E6%9D%80%E5%99%A8%E4%B9%8B%E6%80%A7%E8%83%BD%E5%89%96%E6%9E%90%20PProf.md)

[Multipart form uploads + memory leaks in golang?](https://stackoverflow.com/questions/30657454/multipart-form-uploads-memory-leaks-in-golang)