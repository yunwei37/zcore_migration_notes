# rCore 到 zCore 的功能迁移组相关文档记录

本仓库内容包含：

- 相关链接，如每日记录文档、相关 issue 和 pull-request，相关资料目录；
- 每人已完成的目标和未来的计划；
- 简单的工作记录，可以让同伴大致了解每天完成了什么工作；
- 相关的分析和报告文档等

Group wiki: [2020年操作系统专题训练大实验-移植rCore内核功能到zCore](http://os.cs.tsinghua.edu.cn/oscourse/OsTrain2020/g3)

## 组员每日记录文档

- [李宇](https://github.com/wfly1998/DailySchedule)
- [郑权](https://github.com/VitalyAnkh/learn/blob/master/Notebook/org/20200804025006-zcore_journal.org)
- [郑昱笙](https://github.com/yunwei37/os-summer-of-code-daily)
- [许善朴](https://github.com/xushanpu123)
- [曾广仕](https://github.com/NameAvailable319)

## 组内分工和计划 checklist

* [郑权](https://github.com/VitalyAnkh)：
  * [ ] 实现更多 zircon 系统调用
    * [ ] 实现 `zx_thread_read_state`
  * [ ] (尝试）用 Rust 实现的 [relibc](https://gitlab.redox-os.org/redox-os/relibc) 替换预编译的 libc
* [许善朴](https://github.com/xushanpu123)：
  * [ ] 为系统调用编写测试
  * [ ] 完善文件系统
* [郑昱笙](https://github.com/yunwei37)：
  * [x] 给linux相关的三个模块增加文档
    * [x] linux-loader
    * [x] linux-syscall
    * [x] linux-object 
  * [ ] 完善文件系统相关
  * [x] 进程间通信机制
    * [x] pipe 系统调用
    * [x] 信号量系统调用
    * [x] 共享内存系统调用（大概能跑，继续完善）
  * [ ] 系统调用的单元测试
    * [x] 使用 busybox 的命令进行一定的测试（已完成部分）
    * [x] 完善系统调用单元测试的框架
    * [ ] libc-test
* [李宇](https://github.com/wfly1998)：
  * [x] 移植 shell
    * [x] 实现 stdin
      * [x] 实现 `EventBus`
    * [x] 实现 `sys_poll`
  * [ ] 移植 GNU Make
  * [ ] 移植 Rust 工具链
  * [x] 移植 GCC
* [曾广仕](https://github.com/NameAvailable319)：
 * [ ] 对接文件系统

## 参考链接

感觉这一组做的工作可能会对我们有不少参考意义：
[https://github.com/rcore-os/rCore/tree/master/docs/2020\_OS/g2](https://github.com/rcore-os/rCore/tree/master/docs/2020_OS/g2)

## 当前进度简要记录

[郑权](https://github.com/VitalyAnkh):

在读 zircon-object 代码，尝试实现 `zx_thread_read_state`， 还没有成果。计划：

 zircon-object 中大量使用了 rust 的 async 语法，学习之

---

[李宇](https://github.com/wfly1998)：

记一点东西给大家参考，也是我前几天的研究成果

1. zCore 文件系统位置：`zCore/rootfs/`

2. 交叉编译到 zCore 的命令：

   ```sh
   gcc -Wl,--dynamic-linker=/lib/ld-musl-x86_64.so.1
   ```
   
   或者
   
   ```sh
   musl-gcc -pie -fpie
   ```

   或者直接在 Alpine Linux 中编译

3. 将程序迁移到 zCore 的方法：

   1. 安装 docker 或虚拟机
   2. 在 docker 或虚拟机内安装 Alpine Linux 操作系统
   3. 在 Alpine 操作系统内使用 `apk add` 命令安装需要的包
   4. 将该程序二进制及其依赖的库复制出来即可，具体文件可以去 Alpine 官网的 Packages 部分查询

20200806：

- 在 LibOS 中实现 stdin 成功，也添加了一些必要的系统调用：[#131](https://github.com/rcore-os/zCore/pull/131)

20200807：

- 在 LibOS 中移植 shell 勉强成功，可以运行内置命令

20200812：

- 在 QEMU 中实现 stdin 成功：[#143](https://github.com/rcore-os/zCore/pull/143)
- 此外，我把可以迁移的程序放到仓库里了：[migration](https://github.com/yunwei37/zcore_migration_notes/tree/master/migration)

20200813：

- GCC 在 LibOS 中移植成功，编译时需要加 `-pie -fpie` 参数

20200816：

- 修复了 `sys_wait4` 的 bug，现在 shell 在 QEMU 中可以勉强正常运行，不会因为 `sys_wait4` 而阻塞：[#150](https://github.com/rcore-os/zCore/pull/150)

20200817：

- 添加了信号相关的系统调用，一点尝试：[#155](https://github.com/rcore-os/zCore/pull/155)

20200819：

- 将 `sys_pipe` 改为 `sys_pipe2`，一点尝试：[#157](https://github.com/rcore-os/zCore/pull/157)
- GCC 在 QEMU 中移植成功，使用前需要：
  1. 修改 `linux-syscall/src/lib.rs`，让 `Sys::VFORK` 执行 `sys_fork`
  2. 修改 `zCore/src/memory.rs` 第 20 行，把 16M 的内存改为至少 64 M
  3. 将 `x86_64.img` 调整到合适大小，方法：
     ```bash
     dd if=/dev/zero of=tmp.img count=800000
     dd if=x86_64.img of=tmp.img conv=notrunc
     cp tmp.img x86_64.img
     ```
     其中 800000 可根据需求改为任意数值
  
  编译时需添加 `-pie -fpie` 参数

20200820：

- 修复了 `sys_poll` 的缺陷，现在 rustc 不会再死循环调用 `sys_poll` 了：[#158](https://github.com/rcore-os/zCore/pull/158)

---

[郑昱笙](https://github.com/yunwei37)：

20200806：

- 加了 linux-loader 的文档和一点 busybox 相关测试；

20200807：

- 目前已完成系统调用 `sys_pipe`，添加了测试和文档；
- 增加了单元测试可能的框架

20200808：

- 编译运行 libc-test 成功，思考如何进行自动化测试
- 增加了部分 time 相关系统调用，正在完善

202008010：

- 感觉这部分其实做了一点无用功（虽然也不完全是
- 另外补上了 sys_utimensat，现在可以正常使用 touch 创建文件；

202008011：

- 修改了一点之前犯蠢的测试代码，现在可以使用c语言进行正常的单元测试；
- pull-request: [#135](https://github.com/rcore-os/zCore/pull/135) 被合并啦

希望之后有人能一起来完善单元测试：

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

2020/8/12

- 一些文档和杂七杂八的系统调用接口以及测试的打杂的活,具体参考这里：[#142](https://github.com/rcore-os/zCore/pull/142) 
- 已经被合并

2020/8/13

- 这两天也有些时候在解决测量学实验和计算机组成课程的内容，以及其他杂七杂八的考试，所以时间有可能会稍微不那么多一点...

2020/8/14

- 着手修复 pipe 的相关问题，加上了 async-poll 但还是不能正常使用；

2020/8/15

- 提交了一个相关的pr，解决了一下 shell 里面命令执行 path 的问题：[#149](https://github.com/rcore-os/zCore/pull/149) ，顺带加了一下pipe

2020/8/16

- 今天去试着跑了一下 libc-test：
  - [#152](https://github.com/rcore-os/zCore/pull/152) 解决了一个小测试，顺带补了一点文档；
  - 似乎很多没办法通过的测试还是和 symlink 的绝对路径相关；
  - 浮点数运算差异应该可以不用管
  - 部分是系统调用还没有实现或者实现有部分差异；

2020/8/17 - 2020/8/18

- add semaphores related syscalls
- [#156](https://github.com/rcore-os/zCore/pull/156)

2020/8/20 - 

- pull request: [#160](https://github.com/rcore-os/zCore/pull/160)
