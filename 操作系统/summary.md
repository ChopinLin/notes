# 基本概念

在用户角度，操作系统是一个控制软件。资源管理，分配资源。主要是CPU，内存，磁盘三种资源。层次在硬件之上，应用软件之下。为应用程序提供服务。分为 shell 和kernel。我们关注kernel。

- CPU调度
- 物理内存管理
- 虚拟内存管理
- 文件系统管理
- 中断系统与设备驱动

特征

- 并发

  并发与并行不同。并发是一段时间内，多个程序顺序执行。并行是一个时间点，多个程序同时运行。要多核cpu支持。

- 共享

  同时共享，并发共享

- 虚拟

- 异步

  程序不是一步到底，而是走走停停，但只要输入相同，结果相同

## 操作系统的启动



## 中断、异常、系统调用



## 内存管理

目标：抽象、保护、共享、虚拟化

内存架构（金字塔）

管理内存的不同方法：程序重定位、分段、分页、虚拟内存、按需分页虚拟内存

MMU的作用：MMU是CPU里的一个硬件，主要是地址映射，安全检查

地址空间分为物理地址空间和逻辑地址空间，有物理地址和逻辑地址的区别。

逻辑地址的生成与 编译、汇编、链接、载入有关。与物理地址空间的映射与MMU有关。

内存碎片：内碎片与外碎片

连续内存分配：first fit、best fit、worst fit

问题：内存利用率低，碎片问题，只能大块分配连续内存

非连续内存：分段、分页、页表

允许代码数据共享，支持动态加载等。但是需要有映射，开销大。映射有硬件和软件两种方案。

## 进程与线程

## 调度

## 同步、互斥

## 文件系统

## I/O 子系统



