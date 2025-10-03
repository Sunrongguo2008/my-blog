+++
date = '2025-08-07'
draft = false
title = 'btrfs下不显示timeshift创建的快照／创建快照失败'
tags = ["linux", "blog"]
+++


## 1. 原因

[英雄在这里](https://bbs.deepin.org/zh/post/270814)

> **btrfs 文件系统玩 swapfile，如果还想用 timeshift 或者 snapper 备份快照，就必须把 swap 文件放到不需要备份的子卷里面（比如创建 @swap 子卷）。
> 不然的话，如果直接放在需要快照的子卷，会在创建快照的时候会失败（提示设备正在被写入啥的/ 在 timeshift GUI 上不显示创建的快照）
> 要是像我一样，哪天需要恢复快照了，突然发现一个快照都没建出来就搞笑了 ......**

**所以需要把 swapfile 放到@swap 子卷里面**

## 2. 解决方案

 **⚠️注意：**

在 btrfs 下，创建**swap 文件**而不是 swap 分区时：  
**把 swap 文件放到**​ **​ `/swap` ​**​**子卷中：**  因为 timeshift 不会备份 `/swap` 子卷，导致恢复快照时，swap 文件 `/swap/swapfile` 会消失  
**不把 swap 文件放到子卷中：**  timeshift 备份时，swapfile 被内核使用，会导致 `E: ERROR: Could not create subvolume: Text file busy`，导致无法创建快照 ([参考来源](https://forum.archlinuxcn.org/t/topic/14276))

最佳实践可能是**创建 swap 分区**而不是**swap 文件**

或者仍然**把 swap 文件放到**​ **​ `/swap` ​**​**子卷中**，timeshift 恢复快照后，手动把 /swap 目录删掉、把原来的 /swap 子卷移动过来

以下是**创建 swap 文件+把 swap 文件放到**​ **​ `/swap` ​**​**子卷中**的方案（**创建 swap 分区**的方法网上一搜一大把，就不写了）：

### 2.1 创建交换空间（设置子卷）

```bash
# 创建交换子卷，位置是/swap（其他位置也可以，但是下面的都得换掉）
sudo btrfs subvolume create /swap

# 进入交换子卷目录
cd /swap

# 创建一个0大小的文件
sudo truncate -s 0 /swap/swapfile

# 设置No_COW属性
sudo chattr +C swapfile

# 禁用压缩
sudo btrfs property set swapfile compression none
ERROR: failed to set compression for swapfile: Invalid argument #不知道为什么，不过不影响

# 将文件填充、扩容
sudo dd if=/dev/zero of=/swap/swapfile bs=1G count=6

# 设置swapfile权限
sudo chmod 600 swapfile

# 格式化swapfile
sudo mkswap swapfile

# 启用swapfile
sudo swapon swapfile
```

### 2.2 验证交换空间是否启用

```bash
# 查看交换文件是否已激活
sudo swapon --show

# 查看系统内存和交换使用情况
free -h
```

你应该能看到类似这样的输出：

```bash
$ sudo swapon --show
NAME           TYPE SIZE USED PRIO
/swap/swapfile file   6G   0B   -2

$ free -h
                总计        已用        空闲        共享   缓冲/缓存        可用
内存：          15Gi       6.3Gi       621Mi       1.3Gi       9.5Gi       9.0Gi
交换：         6.0Gi          0B       6.0Gi   
```

### 2.3 设置开机自动挂载 swapfile

```bash
# 编辑 fstab 文件
sudo nano /etc/fstab

# 在文件末尾添加这一行：
/swap/swapfile none swap sw 0 0
```

**重启后验证**交换文件是否自动启用：

```bash
# 查看交换文件是否已激活
sudo swapon --show

# 查看系统内存和交换使用情况
free -h
```

你应该能看到类似这样的输出：

```bash
$ sudo swapon --show
NAME           TYPE SIZE USED PRIO
/swap/swapfile file   6G   0B   -2

$ free -h
                总计        已用        空闲        共享   缓冲/缓存        可用
内存：          15Gi       6.3Gi       621Mi       1.3Gi       9.5Gi       9.0Gi
交换：         6.0Gi          0B       6.0Gi   
```

‍
