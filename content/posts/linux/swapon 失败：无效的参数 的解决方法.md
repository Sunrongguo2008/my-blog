+++
date = '2025-08-05'
draft = false
title = 'swapon 失败：无效的参数 的解决方法'
tags = ["linux"]
+++

## 1. 特点
```bash
$ sudo swapon /swapfile    

swapon: /swapfile：swapon 失败: 无效的参数
或者 swapon: swapfile: swapon failed: Invalid argument
```

## 2. 原因

在**Btrfs**上建立**交换文件**需要设置 `No_COW` 属性 + 禁用压缩

## 3. 解决方案

 **⚠️注意：**

在 btrfs 下，创建**swap 文件**而不是 swap 分区时：  
**把 swap 文件放到**​ **​ `/swap` ​**​**子卷中：**  因为 timeshift 不会备份 `/swap` 子卷，导致恢复快照时，swap 文件 `/swap/swapfile` 会消失  
**不把 swap 文件放到子卷中：**  timeshift 备份时，swapfile 被内核使用，会导致 `E: ERROR: Could not create subvolume: Text file busy`，导致无法创建快照 ([参考来源](https://forum.archlinuxcn.org/t/topic/14276))

最佳实践可能是**创建 swap 分区**而不是**swap 文件**

或者仍然**把 swap 文件放到**​ **​ `/swap` ​**​**子卷中**，timeshift 恢复快照后，手动把 /swap 目录删掉、把原来的 /swap 子卷移动过来

以下是**创建 swap 文件+不把 swap 文件放到子卷中**的方案（**创建 swap 分区**的方法网上一搜一大把，就不写了）：

### 3.1 **错误**示范：

```bash
$ sudo truncate -s 6G /swapfile # 创建一个6G大小的文件
$ sudo chattr +C /swapfile # 设置No_COW属性
$ sudo btrfs property set /swapfile compression none # 禁用压缩
$ sudo chmod 600 /swapfile # 设置swapfile权限
$ sudo mkswap /swapfile # 格式化swapfile

mkswap: /swapfile 包含空洞或者其它不支持的扩展。
        此交换文件可能会在启用时被内核拒绝！
        使用 --verbose 显示更多细节。

正在设置交换空间版本 1，大小 = 6 GiB (6442446848  个字节)
无标签，UUID=0c11bd0e-061e-419f-88b9-b13504f612c4
$ sudo swapon /swapfile # 启用swapfile
swapon: /swapfile：将跳过 - 它似乎有空洞。
$
```

错误行为：先写入 6 G，再设置 `No_COW` 属性 + 禁用压缩

❌这样不行，6 G 内容被压缩了

[热心网友的提醒（在评论区）：](https://bbs.deepin.org/zh/post/270814)

>  **⚠️注意**
>
> **btrfs 文件系统玩 swapfile，如果还想用 timeshift 或者 snapper 备份快照，就必须把 swap 文件放到不需要备份的子卷里面（比如创建 @swap 子卷）。
> 不然的话，如果直接放在需要快照的子卷，会在创建快照的时候会失败（提示设备正在被写入啥的/ 在 timeshift GUI 上不显示创建的快照）
> 要是像我一样，哪天需要恢复快照了，突然发现一个快照都没建出来就搞笑了......**

设置子卷的 swap 创建我会放在[另一个博客](https://www.cnblogs.com/sunong/p/19026935.html)里面

**如果你要用到 timeshift 等 btrfs 快照（大概率你会用到），就去那个博客里面看！**

博客地址：https://www.cnblogs.com/sunong/p/19026935.html

### 3.2 **正确**示范（不设置子卷）

 **⚠️再次提醒：如果你要用到 timeshift 等 btrfs 快照（大概率你会用到），就去那个博客里面看！**

```bash
# 创建一个0大小的文件
sudo truncate -s 0 /swapfile

# 设置No_COW属性
sudo chattr +C /swapfile

# 禁用压缩
sudo btrfs property set /swapfile compression none
ERROR: failed to set compression for /swapfile: Invalid argument #不知道为什么，不过不影响

# 将文件填充、扩容
sudo dd if=/dev/zero of=/swapfile bs=1G count=6

# 设置swapfile权限
sudo chmod 600 /swapfile

# 格式化swapfile
sudo mkswap /swapfile

# 启用swapfile
sudo swapon /swapfile
```

### 3.3 验证交换空间是否启用

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

### 3.4 设置开机自动挂载 swapfile

```bash
# 编辑 fstab 文件
sudo nano /etc/fstab

# 在文件末尾添加这一行：
/swapfile none swap sw 0 0
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
/swapfile file   6G   0B   -2

$ free -h
                总计        已用        空闲        共享   缓冲/缓存        可用
内存：          15Gi       6.3Gi       621Mi       1.3Gi       9.5Gi       9.0Gi
交换：         6.0Gi          0B       6.0Gi   
```

‍
