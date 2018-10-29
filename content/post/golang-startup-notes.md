---
title: "golang 入门学习笔记"
date: 2018-10-29T23:10:21+08:00
draft: false
---

## 一、基础概念：工作区、GOROOT、GOPATH 等

第一步安装 go 的时候，在教程里了解到，golang 所有源码都在同一个工作区目录 (GOPATH?) 下。 GOROOT 则是 go 本身被安装到的本地目录。

// 印象就这些，总感觉还缺点什么，处于一知半解中。

## 二、Hello World

调试运行 `Hello World` 花了一些时间，因为自己执行 `go build`, `go install` 的结果和教程不太一样，要么教程里某处细节自己本地没有出现，要么直接报错。排查常见错误原因，该配置的地方却也都配置了。后来勉强能运行一些试验代码，就暂且不了了之了。

## 三、go 的特色：类型定义

go 语言的类型定义是印象较深的，据说这也是 go 的特色。

### 数组

将存储功能与使用功能区分开来，存储对应 `array`, 使用则对应非常灵活的 `slice`. 这样设计的原因也不难理解——对比 C, 其数组长度是不可变的；若是 C++ 则可以用 `vector` 实现变长数组；Java 的数组类型长度也不可变，但 Java 为提供变长数组而引入了 `ArrayList`; `array` 和 `slice` 的设计算是比较精巧的一种解决方式。

看 Effective Go 的介绍，多维数组在 go 中似乎不那么好用，想必这门语言也就不擅长做矩阵计算等咯？

### 函数

同样写起来非常灵活，尤其是 Multiple return value 和 Named variable return, 前者极大方便了错误信息的传递，后者则进一步增进了代码的简洁。

### Defer

说函数所以顺带提及 defer, 其实不是一个类型。感觉作用跟 Java 的 finally 类似，不过有点自己的小玄机：Defer 做 evaluation 的时机是 on execute, not on call. Effective Go 中概括其要点为：function-based, not block-based.

好处很明显：简化语法；令资源的申请与释放操作在编码中更为接近；不易出错（比如新增一个带有 `return` 语句的 `if` 分支，不必十分小心地在其中添加 `close` 语句）等。实践中的用法还未有更多见识。

### Pointer

类比 C 的指针粗浅理解，go 中不会有多级指针。

*更多内容有待学习。*

### interface

竟然也是一种类型。

*目前了解得还比较少*

### goroutine, channel

简单的说明/示例代码看了一些，暂时还说不出什么所以然。据 Effective Go 还是 gopl 描述，这部分的设计是另辟蹊径，其他语言是 "communicate by sharing vars", 而 go 则是 "share vars by communicating", 不太能明白。

## 四、控制流程

统一的循环书写方式（不用纠结 while, do while 了，只有 for）. 非常灵活的 switch. PS: 感觉 switch 的重新设计很有想象力。


