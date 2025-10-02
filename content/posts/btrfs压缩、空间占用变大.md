+++
date = '2025-08-05'
draft = false
title = 'btrfs压缩、空间占用变大'
tags = ["linux", "blog"]
+++


## 1. Btrfs 压缩

### 1.1 实施步骤：

**修改**  **​ `/etc/fstab` ​**：

```bash
UUID=da4fd0a6-78d7-464c-8e24-b38ee2d483b7 /              btrfs   subvol=/@,defaults,compress=zstd:3 0 0
UUID=da4fd0a6-78d7-464c-8e24-b38ee2d483b7 /home          btrfs   subvol=/@home,defaults,compress=zstd:3 0 0
UUID=da4fd0a6-78d7-464c-8e24-b38ee2d483b7 /var/cache     btrfs   subvol=/@cache,defaults,compress=zstd:1 0 0
UUID=da4fd0a6-78d7-464c-8e24-b38ee2d483b7 /var/log       btrfs   subvol=/@log,defaults,compress=zstd:2 0 0
```

**重新挂载所有分区**：

```bash
# 重新加载 systemd 配置
sudo systemctl daemon-reload

# 然后再重新挂载
sudo mount -o remount,compress=zstd:3 /
sudo mount -o remount,compress=zstd:3 /home
sudo mount -o remount,compress=zstd:1 /var/cache
sudo mount -o remount,compress=zstd:2 /var/log
```

**重新压缩现有数据**（可选，需要一些时间）：

```bash
#在 defragment 命令中，压缩参数的语法不同。
sudo btrfs filesystem defragment -r -v -czstd /  # 注意这里只需要指定算法，不需要级别
sudo btrfs filesystem defragment -r -v -czstd /home
sudo btrfs filesystem defragment -r -v -czstd /var/cache
sudo btrfs filesystem defragment -r -v -czstd /var/log
```

## 2. 空间占用变大：

### 2.1 问题分析：

```bash
sudo btrfs filesystem usage /
```

Data 部分 `Data,single` 变大

### 2.2 解决方案：

这种情况通常是由于重新压缩过程中产生的碎片或者空间分配不均造成的。需要执行平衡操作来重新整理数据：

```bash
# 执行平衡来优化空间使用
sudo btrfs balance start -dusage=50 /
# 如果上面的完成，可以再执行更彻底的平衡(更有效)
sudo btrfs balance start /

# 监控平衡进度
sudo btrfs balance status /
# 五秒一查
watch -n 5 'sudo btrfs balance status /'
```

重启就好了

或者试试删除快照
