# rCore 到 zCore 功能迁移组汇报——程序使用方法

## 移植成功的程序

* shell (busybox)
* GCC

## 使用方法

### 环境配置

按照 zCore 仓库的 `README.md` 进行配置，使其可正常执行如下命令：

```bash
cargo run --release -p linux-loader /bin/busybox ls
```

### LibOS

#### 复制程序

注意：

由于 LibOS 与 QEMU 的 fork 问题，LibOS 中仅能使用 `sys_vfork`，不能使用 `sys_fork`；而 QEMU 中仅能使用 `sys_fork`，不能使用 `sys_vfork`；

所以，LibOS 与 QEMU 使用的 shell (busybox) 不同，LibOS 中需使用王润基学长添加 Force-NOMMU 参数编译的 `busybox`，QEMU 中需使用原版 `busybox`

而以下操作可能涉及 `busybox` 的替换，因此建议进行操作前先将 `rootfs/bin/` 中的原版 `busybox` 备份一下

```bash
git clone https://github.com/yunwei37/zcore_migration_notes.git
cd zcore_migration_notes/migration
cp bin/* path_to_zcore/rootfs/bin
cp -d lib/* path_to_zcore/rootfs/lib
cp -rd x86_64-linux-musl/* path_to_zcore/rootfs
```

#### `shell` 使用方法 (LibOS)

将程序复制成功后，即可执行：

```bash
cargo run --release -p linux-loader /bin/busybox sh
```

注：

1. 由于 LibOS 及 HostFS 的限制，在 `shell` 中不能直接执行如 `ls`, `rm` 等命令，需使用 `busybox ls`, `busybox rm` 等命令执行
2. 由于未知原因，`shell` 不会显示正常 shell 会显示的如 `/ #`, `~ $` 等表示 shell 的符号
3. 由于未知原因，`shell` 除内置命令，如 `cd`, `pwd` 外，只能执行一条外部命令，解决方法如下（**不建议**）：
   1. 打开 `linux-object/src/process.rs`，找到 `wait_child` 和 `wait_child_any` 函数
   2. 将以上两函数结尾前执行 `signal_clear` 的代码注释掉，即可正常执行 **LibOS 中的** shell
      **注**：
      * 将这两行代码注释掉后仅可正常执行 LibOS 中的 shell，但会影响 QEMU 中的 shell 及 GCC 等程序正常执行，**所以不建议修改**
      * 如已修改，执行 GCC 前或使用 QEMU 前请取消掉以上两行的注释

#### GCC 使用方法 (LibOS)

将程序复制成功后，即可执行：

```bash
cargo run --release -p linux-loader /bin/x86_64-linux-musl-gcc
```

也可在 `rootfs/` 目录，或者 `rootfs/root/` 目录中创建 `*.c` 文件进行编译，如创建 `rootfs/root/hello.c`：

```bash
cargo run --release -p linux-loader /bin/x86_64-linux-musl-gcc -pie -fpie /root/hello.c -o /root/hello.out
```

之后即可在 `rootfs/root/` 中编译出 `hello.out` 文件，使用如下命令执行：

```bash
cargo run --release -p linux-loader /root/hello.out
```

注：编译时必须添加 `-pie -fpie` 参数，因为 zCore 目前无法执行 PIE-disabled 的程序

### QEMU

#### 生成镜像

0. 将 `rootfs/bin/` 中的 `busybox`  替换为原版

1. 使用如下命令安装 `rcore-fs-fuse`

   ```bash
   cargo install rcore-fs-fuse --git https://github.com/rcore-os/rcore-fs
   ```

2. 在 zCore 仓库目录中执行如下命令打包镜像：

   ```bash
   rcore-fs-fuse zCore/x86_64.img rootfs zip
   ```

   打包前记得提前创建需用 GCC 编译的 C 程序，如果之前已创建并编译，请提前删除 `hello.out` 以避免影响后续 GCC 的使用

3. `cd zCore`，即可进行如下操作

#### `shell` 使用方法 (QEMU)

由于在 zCore 中运行 `shell` 的代码已经写好，所以直接运行如下命令即可启动 `shell`：

```bash
make run mode=release accel=1 linux=1
```

注：`shell` 中目前不会显示 `/ ＃` 等符号，但可直接输入 `ls` 等命令执行

#### GCC 使用方法 (QEMU)

1. 修改 `src/memory.rs` 第 20 行，将内存大小从 16M 改为 至少 64M：

   ```rust
   const KERNEL_HEAP_SIZE: usize = 64 * 1024 * 1024;
   ```

2. 由于 GCC 需调用 `sys_vfork`，而 运行在 QEMU 中的 zCore 不支持 `sys_vfork`，所以需修改 `../linux-syscall/src/lib.rs`，将 `Sys::VFORK` 所在一行注释掉，并添加如下一行：

   ```rust
   Sys::VFORK => self.sys_fork(),
   ```

   将 `sys_vfork` 重定向到 `sys_fork`

3. *假定*已在 `root/` 中创建 `hello.c` 并启动 QEMU：

   ```bash
   cd root
   x86_64-linux-musl-gcc -pie -fpie hello.c -o hello.out
   ./hello.out
   ```

注：

1. 编译时需添加 `-pie -fpie` 参数
2. 编译不一定可以成功，若失败会导致 QEMU 卡死，这时需重启 QEMU 并重新编译

### 未移植成功的 Rust 工具链

1. 从 [x86\_64-unknown-linux-musl](https://static.rust-lang.org/dist/rust-1.45.2-x86_64-unknown-linux-musl.tar.gz) 下载 Rust 工具链安装包并解压，解压后：

   ```bash
   cd rust-1.45.2-x86_64-unknown-linux-musl
   ./install.sh --prefix=path_to_zcore/rootfs
   ```

2. 之后即可运行 rustc：

   ```bash
   cargo run --release -p linux-loader /bin/rustc
   ```

   能够看到，`rustc` 可以正常输出帮助信息，但编译仅可编译出 `*.o` 的中间结果，目前还不能编译出可执行程序
