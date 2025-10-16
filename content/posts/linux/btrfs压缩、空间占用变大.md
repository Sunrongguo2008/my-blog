+++
date = '2025-08-05'
draft = false
title = 'btrfs压缩、空间占用变大'
tags = ["linux"]
+++


## 1. 启用 Btrfs 压缩

### 1.1 配置步骤

要为 Btrfs 文件系统启用透明压缩，需在 `/etc/fstab` 中为挂载选项添加 `compress` 参数。

**编辑 `/etc/fstab`：**

```bash
UUID=da4fd0a6-78d7-464c-8e24-b38ee2d483b7 /              btrfs   subvol=/@,defaults,compress=zstd:3      0 0
UUID=da4fd0a6-78d7-464c-8e24-b38ee2d483b7 /home          btrfs   subvol=/@home,defaults,compress=zstd:3 0 0
UUID=da4fd0a6-78d7-464c-8e24-b38ee2d483b7 /var/cache     btrfs   subvol=/@cache,defaults,compress=zstd:1 0 0
UUID=da4fd0a6-78d7-464c-8e24-b38ee2d483b7 /var/log       btrfs   subvol=/@log,defaults,compress=zstd:2   0 0
```

> **说明**：`zstd` 压缩级别范围通常为 1–15（部分内核版本可能支持更高），级别越高压缩率越好但 CPU 开销更大。`zstd:3` 是兼顾性能与压缩率的常用选择。

**重新挂载文件系统以应用新选项：**

```bash
# 重新加载 systemd 配置（非必需，但推荐）
sudo systemctl daemon-reload

# 重新挂载各子卷
sudo mount -o remount,compress=zstd:3 /
sudo mount -o remount,compress=zstd:3 /home
sudo mount -o remount,compress=zstd:1 /var/cache
sudo mount -o remount,compress=zstd:2 /var/log
```

> **注意**：重新挂载仅对**新写入**的文件生效。已有文件仍为未压缩状态。

**（可选）对现有文件重新压缩：**

Btrfs 通过 `defragment` 操作可对已有文件应用压缩。但需注意：**`btrfs filesystem defragment` 命令不支持指定 zstd 压缩级别**，仅能指定算法。

```bash
# 重新压缩根目录下的所有文件（递归）
sudo btrfs filesystem defragment -r -v -czstd /

# 对其他子卷执行相同操作
sudo btrfs filesystem defragment -r -v -czstd /home
sudo btrfs filesystem defragment -r -v -czstd /var/cache
sudo btrfs filesystem defragment -r -v -czstd /var/log
```

> ⚠️ **重要提示**：  
> - `defragment` 会创建文件的新副本（受 CoW 影响），**短期内可能使磁盘使用量翻倍**。  
> - 该操作 I/O 和 CPU 开销较大，建议在系统空闲时执行。  
> - 某些内核版本中，`-czstd` 实际使用默认级别（通常为 3），无法指定具体级别。


## 2. 为何启用压缩后空间占用反而变大？

### 2.1 问题分析

启用压缩并执行 `defragment` 后，观察到 `btrfs filesystem usage /` 显示 **`Data,single` 使用量显著增加**，而 **`Device unallocated`（未分配空间）却很小**，甚至系统提示磁盘空间不足。

可能是以下原因导致：

1. **写时复制（CoW）机制**：  
   Btrfs 在修改文件时不会覆盖原数据，而是写入新位置。执行 `defragment` 时，每个文件都会生成一个压缩后的新副本，**旧数据块不会立即释放**。

2. **空间未回收**：  
   删除或覆盖文件后，Btrfs 不会立即将空间返还给文件系统，而是标记为“可回收”。只有在执行 **balance** 或 **sync + trim** 等操作后，空间才会真正释放。



### 2.2 解决方案：执行 Balance 操作

`btrfs balance` 可重新组织数据块，合并碎片，并释放未使用的空间。

```bash
# 推荐：仅平衡使用率低于 50% 的数据块（更快）
sudo btrfs balance start -dusage=50 /

# 或执行完整平衡（耗时较长，更彻底）
sudo btrfs balance start /
```

> **建议**：首次可先用 `-dusage=50` 快速回收空间；若效果不佳，再考虑完整平衡。

**监控平衡进度：**

```bash
# 查看当前状态
sudo btrfs balance status /

# 每 5 秒刷新一次
watch -n 5 'sudo btrfs balance status /'
```

**平衡完成后，强制同步并释放空间：**

```bash
sudo btrfs filesystem sync /
```

> 此命令确保所有挂起的写操作完成，并触发空间回收。

### 2.3 建议

- **定期执行 balance**：对于写入频繁的系统（如日志、缓存目录），可设置定时任务每月执行一次轻量级 balance。
- **检查快照**：Btrfs 快照会阻止空间释放。使用 `btrfs subvolume list /` 查看是否有冗余快照，并用 `btrfs subvolume delete` 清理。（或者用timeshift）


## 附：数据完整性检查

为确保压缩和平衡操作未损坏数据，可运行 scrub：

```bash
# 启动后台校验
sudo btrfs scrub start /

# 查看结果
sudo btrfs scrub status /
```

Scrub 会读取所有数据块并验证校验和，自动修复可恢复的错误（需有冗余配置，如 RAID1）。

