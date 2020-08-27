# rCore 到 zCore 功能迁移组报告

郑昱笙、李宇

## 目录

<!-- TOC -->

- [rCore 到 zCore 功能迁移组报告](#rcore-到-zcore-功能迁移组报告)
  - [目录](#目录)
  - [实验目标描述](#实验目标描述)
    - [概述](#概述)
    - [目标分解：](#目标分解)
  - [已有相关工作介绍](#已有相关工作介绍)
  - [小组成员分工](#小组成员分工)
  - [主要成果描述](#主要成果描述)
    - [李宇](#李宇)
    - [郑昱笙](#郑昱笙)
      - [补全 zCore 中 Linux 相关的三个模块的文档和单元测试：](#补全-zcore-中-linux-相关的三个模块的文档和单元测试)
      - [完善文件系统和IO相关系统调用](#完善文件系统和io相关系统调用)
      - [完善进程间通信机制](#完善进程间通信机制)
  - [对实验的后续开发内容建议或设想](#对实验的后续开发内容建议或设想)
  - [实验总结](#实验总结)
  - [实验过程日志](#实验过程日志)

<!-- /TOC -->


## 实验目标描述 

### 概述

rCore 是用 Rust 语言实现的兼容 Linux 内核。它支持四种指令集，能够运行比较丰富的应用程序。但是随着时间的积累，rCore 的代码越堆越多，很多内部实现缺乏推敲，需要优化和重构。后来我们从头开始实现了 zCore 项目，采用了更加清晰的分层结构，同时复用 Zircon 微内核的内核对象实现了 Linux 内核的部分功能（如内存管理和进程管理）。目前 zCore 中的 linux 模块已经能够运行基础的 Busybox 等小程序，但仍有大量原本 rCore 支持的功能没有实现。本项目希望将 rCore 的功能迁移到 zCore 当中，并借此机会进行重构。其中一些代码可以直接搬过来，剩下的可能需要调整适配（例如涉及到 async），还有一些可以直接基于 Zircon 内核对象进行实现（例如 epoll）。

### 目标分解：

- 李宇：


- 郑昱笙：
  - 给linux的三个模块补全文档和测试
  - 完善文件系统相关系统调用；
  - 完善进程间通信机制


## 已有相关工作介绍 

- zCore 中 Linux 相关的三个模块的文档和单元测试集合都相对缺失；
- 文件相关系统调用 zCore 中除了以下部分已经基本完成，但以下功能在rCore中都能找到：
  - stdin 尚未实现；
  - io多路复用（如 select poll epoll）尚未实现
  - touch 不能创建文件，时间相关模块暂时缺失；
- 进程间通信机制暂时缺失，相比 rCore 实现了三种机制：pipe、信号量、共享内存
- 信号机制暂时缺失，相比 rCore 中也有对应的实现
- zCore 中 linux 模块并没有像 zircon 那样的系统调用测试程序集合，rCore 则可用 libc-test 进行测试；
- rCore 中移植相关用户态程序留下了不少文档可供参考；

## 小组成员分工 

- 李宇：尝试从 Linux 用户程序入手，在 zCore 上运行 rCore 支持的 GCC，Nginx，Rustc 等，并修复和移植相关的功能；
- 郑昱笙：尝试完善 linux 系统调用的单元测试和 libc-test, 并以测试驱动尝试尽可能多地移植 rCore 的相关功能

## 主要成果描述 

### 李宇



### 郑昱笙


#### 补全 zCore 中 Linux 相关的三个模块的文档和单元测试：

相关PR：

- [#125 Improve docs for linux-syscall and fix "uname" command](https://github.com/rcore-os/zCore/pull/125)
- [#128 Improve doc and add some simple trivial test in linux-loader](https://github.com/rcore-os/zCore/pull/128)
- [#132 Add simple pipe syscall and unit test method for linux syscall](https://github.com/rcore-os/zCore/pull/132)
- [#142 Improve doc in linux-object and migrate some simple syscalls](https://github.com/rcore-os/zCore/pull/142)

在开始之前，相关单元测试仅仅使用了运行 busybox 作为一个简单的测试，并且不检测用户程序执行的返回值；

在这段时间中，我们添加了大量基于用户态程序的单元测试，并完善了相应测试代码，补全了用户态返回值判断和自动编译测试用例的部分，使得现在可以简单地通过这样的方法使用 c 语言对系统调用进行单元测试：

1. 在 linux-syscall/test 文件夹里面编写 c 语言的测试程序，可以使用 assert 函数判断是否正确；
2. 在 linux-loader 的 main.rs 里面可以这样写：

```rs
    #[async_std::test]
    async fn test_pipe() {
        assert_eq!(test("/bin/testpipe1").await, 0);
    }
```

3. 运行 `make rootfs` 命令
4. run test

三个模块的单元测试覆盖率变化如下：

- linux-loader  74.63 -> 87.88
- linux-syscall 18.68 -> 61.55
- linux-object  41.84 -> 56.17

除此之外，linux 相关三个模块的文档均已补齐，并均能通过 `#[deny(missing_docs)]` 编译，主要参考linux相关文档；另外也参与了一点 libc-test 移植相关的工作。

#### 完善文件系统和IO相关系统调用

具体修改和完善如下

- [#135 Add more time syscalls and fix touch command in busybox](https://github.com/rcore-os/zCore/pull/135)
  - 添加了时间相关模块（timeval timespec）和相关系统调用：
    - `time`( )
    - `gettimeofday`( )
    - `gettusage`( )
    - `times`( )
  - 添加了 utimensat 系统调用，使 touch 命令可以正常创建文件；

- [#142 Improve doc in linux-object and migrate some simple syscalls](https://github.com/rcore-os/zCore/pull/142)
  - 添加了 `sysinfo` 和 `flock` 系统调用；

- [#149 fix dynamic link path for envs in shell and add async poll in pipe](https://github.com/rcore-os/zCore/pull/149)
  - 部分修复了shell环境变量无法正常运行程序的问题；

- [#152 fix regression/rlimit-open-files.exe in libc-test](https://github.com/rcore-os/zCore/pull/152)
  - 添加了 `dup` 系统调用；
  - 添加了进程最大打开文件数量限制；

- [#164 Add async select syscall](https://github.com/rcore-os/zCore/pull/164)
  - 添加了使用 async/await 的 `select` 系统调用；
  - 完善了 poll 和 select 系统调用的超时机制；

目前文件系统相关方面除了符号链接存在的问题和权限控制机制，以及 epoll 系统调用，应该已经基本完善；

#### 完善进程间通信机制

1. 添加和完善 pipe 系统调用：
   - 添加了 `pipe` `pipe2` 系统调用，以及相关的文档和用户态单元测试；
   - 相关PR：
     - [#132 Add simple pipe syscall and unit test method for linux syscall](https://github.com/rcore-os/zCore/pull/132)
     - [#149 fix dynamic link path for envs in shell and add async poll in pipe](https://github.com/rcore-os/zCore/pull/149)

2. 信号量相关系统调用；
   - 在 linux-object::sync 中添加了信号量同步机制；
   - 添加了 `semget` `semop` `semctl` 系统调用，以及相关的文档和用户态单元测试（基于 libc-test）；
   - 相关PR：
     - [#156 Add semaphores syscalls](https://github.com/rcore-os/zCore/pull/156)

3. 共享内存相关系统调用；
   -  添加了 `shmget` `shmat` `shmdt` `shmctl` 系统调用，以及相关的文档和用户态单元测试（基于 libc-test）；
   - 相关PR：
     - [#160 Add shared memory ipc syscalls](https://github.com/rcore-os/zCore/pull/160)
  
目前 rCore 中已有的进程间通信机制已经全部完成移植，另外也修复和增添了一些功能；

## 对实验的后续开发内容建议或设想 

- symlink 系统调用尚未完成，需要硬件抽象层配合进行修复；
- 信号机制实现还不够完善，kill 系统调用还未实现；
- epoll 还未实现，要涉及到部分文件相关概念转换；
- 网络通信部分还未实现，但工作量比较大；
- 可以将 zCore 中部分已经重构或实现更完善的代码整合回 rCore 中；

## 实验总结 



郑昱笙：

- 感觉收获好多

## 实验过程日志 

链接：

- 李宇： [https://github.com/wfly1998/DailySchedule/blob/master/README_official.md](李宇： https://github.com/wfly1998/DailySchedule/blob/master/README_official.md)
- 郑昱笙：[https://github.com/yunwei37/os-summer-of-code-daily](https://github.com/yunwei37/os-summer-of-code-daily)