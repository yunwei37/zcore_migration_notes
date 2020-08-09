# 第二阶段目标描述 - 李宇

## 移植 shell

- [ ] 移植 shell
  - [x] 编写 stdin 使其在 LibOS 中工作
  - [ ] 修改 stdin 使其在 QEMU 中工作
  - [ ] 完善 `sys_fork` 使其在 QEMU 中工作

### 备注

1. （20200808）现有的 stdin 目前只能在 LibOS 中工作，不能在 QEMU 中工作
2. 经王润基学长提示，`sys_fork` 在 LibOS 中难以实现，所以后续工作均需要在 QEMU 中进行

## 移植 GNU Make

- [ ] 移植 make
  - [ ] 待补充

## 移植 Rust 工具链

- [ ] 移植 Rust 工具链
  - [ ] 待补充

## 移植 GCC

- [ ] 移植 GCC
  - [ ] 待补充

## 移植 Nginx

- [ ] 移植 Nginx
  - [ ] 编写网络相关系统调用

