# zcore_migration_notes

## 组员每日记录文档

- [李宇](https://github.com/wfly1998/DailySchedule)
- [郑权](https://github.com/VitalyAnkh)
- [郑昱笙](https://github.com/yunwei37/os-summer-of-code-daily)
- [许善朴](https://github.com/xushanpu123)
- [曾广仕](https://github.com/NameAvailable319)

## 组内分工

* [郑权](https://github.com/VitalyAnkh)：
  * [ ] 实现更多 zircon 系统调用
    * [ ] 实现 `zx_thread_read_state`
  * [ ] (尝试）用 Rust 实现的 [relibc](https://gitlab.redox-os.org/redox-os/relibc) 替换预编译的 libc
* [许善朴](https://github.com/xushanpu123)：
* [郑昱笙](https://github.com/yunwei37)：
  * [ ] 给linux相关的三个模块增加文档
    * [x] linux-loader
    * [x] linux-syscall
    * [ ] linux-object 
  * [ ] 完善文件系统相关
  * [ ] 进程间通信机制
    * [x] pipe 系统调用（正在进行测试）
    * [ ] 可能也需要对 linux-object 线程和进程进行一定改进
  * [ ] 系统调用的单元测试
    * [ ] 使用 busybox 的命令进行一定的测试（已完成部分）
    * [ ] 完善系统调用单元测试的框架
    * [ ] libc-test
* [李宇](https://github.com/wfly1998)：
  * [ ] 移植 shell
    * [x] 实现 stdin
      * [ ] 实现 `Condvar`
    * [ ] 实现 `sys_poll`
  * [ ] 实现 `sys_fork`
* [曾广仕](https://github.com/NameAvailable319)：
 *[ ] 对接文件系统

## 已完成工作

## 每日进度

### 20200806

[郑权](https://github.com/VitalyAnkh):

在读 zircon-object 代码，尝试实现 `thread_read_state`， 还没有成果。计划：

 zircon-object 中大量使用了 rust 的 async 语法，学习之

[李宇](https://github.com/wfly1998)：

记一点东西给大家参考，也是我前几天的研究成果

1. zCore 文件系统位置：`zCore/rootfs/`

2. 交叉编译到 zCore 的命令：

   ```sh
   gcc -Wl,--dynamic-linker=/lib/ld-musl-x86_64.so.1
   ```

3. 将程序迁移到 zCore 的方法：

   1. 安装 docker 或虚拟机
   2. 在 docker 或虚拟机内安装 alpine 操作系统
   3. 在 alpine 操作系统内使用 `apk add` 命令安装需要的包
   4. 将该程序二进制及其依赖的库复制出来即可，具体文件可以去 alpine 官网的 Packages 部分查询

目前已完成系统调用 `sys_getrandom` 和 stdin，已提交 [Pull Request #131](https://github.com/rcore-os/zCore/pull/131)

---

[郑昱笙](https://github.com/yunwei37)：

- 加了 linux-loader 的文档和一点测试；
- 目前已完成系统调用 `sys_pipe`，但由于 fork 暂时没办法用可能比较难以通过用户态程序测试...
