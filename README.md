# zcore_migration_notes

## 组内分工

* [郑权](https://github.com/VitalyAnkh)：
* [许善朴](https://github.com/xushanpu123)：
* [郑昱笙](https://github.com/yunwei37)：
  * [ ] 完善文件系统
  * [ ] 进程间通信机制
  * [ ] libc-test
  * [ ] 系统调用的单元测试
* [李宇](https://github.com/wfly1998)：
  * [ ] 移植 shell
  * [ ] 实现 `sys_fork`
* [曾广仕](https://github.com/NameAvailable319)：

## 已完成工作

## 每日进度

---

### 20200806

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

目前已完成系统调用 `sys_getrandom` 和 stdin

---

[郑昱笙](https://github.com/yunwei37)：

- 加了 linux-loader 的文档和一点测试；
- 目前已完成系统调用 `sys_pipe`，等等测试好了再发 pull-request