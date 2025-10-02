+++
date = '2025-08-07'
draft = false
title = 'timeshift无法删除快照'
tags = ["linux"]
+++

## 发现问题

```bash
$ sudo timeshift --list #列出快照
[Warning] Deleted invalid lock
Mounted '/dev/nvme0n1p6' at '/run/timeshift/47565/backup'
btrfs: Quotas are not enabled
Device : /dev/nvme0n1p6
UUID   : da4fd0a6-78d7-464c-8e24-b38ee2d483b7
Path   : /run/timeshift/47565/backup
Mode   : BTRFS
Status : OK
3 snapshots, 9.9 GB free

Num     Name                 Tags       Description                                 
------------------------------------------------------------------------------
0    >  2025-08-05_22-44-05  O          Before restoring '2025-08-05 10:18 下午'  
1    >  2025-08-07_14-31-24  B H D W M                                              
2    >  2025-08-07_15-08-42  O H                                                    

```

我无法删除的快照是 `2025-08-05_22-44-05`，尝试用命令行删一下

```bash
$ sudo timeshift --delete --snapshot 2025-08-05_22-44-05                                       ✔ 
Mounted '/dev/nvme0n1p6' at '/run/timeshift/46581/backup'
btrfs: Quotas are not enabled
------------------------------------------------------------------------------
Removing snapshot: 2025-08-05_22-44-05
Deleting subvolume: @ (Id:309)
E: ERROR: Could not destroy subvolume/snapshot: Directory not empty

E: Failed to delete snapshot subvolume: '/run/timeshift/46581/backup/timeshift-btrfs/snapshots/2025-08-05_22-44-05/@'
E: Failed to remove snapshot: 2025-08-05_22-44-05
```

删除失败了。观察到快照文件在 `/run/timeshift/47326/backup/timeshift-btrfs/snapshots/2025-08-05_22-44-05/`，咱看一下这个目录

```bash
$ ls -la "/run/timeshift/47326/backup/timeshift-btrfs/snapshots/2025-08-05_22-44-05/"
ls: 无法访问 '/run/timeshift/47326/backup/timeshift-btrfs/snapshots/2025-08-05_22-44-05/': 没有那个文件或目录
```

好家伙，目录不存在

**问题发现了：快照目录在文件系统中已经不存在了，但是 Timeshift 仍然认为它存在。这是一个典型的 Timeshift 状态与实际文件系统不一致的问题。这说明 Timeshift 的内部数据库与实际文件系统状态不同步。**

## 解决问题

使用 `sudo timeshift --list`：

查看 `Device` 字段（每个人的都不一样）：`Device : /dev/nvme0n1p6` ​

查看想要删除的快照的 `NAME` 字段（每个人的都不一样）：`2025-08-05_22-44-05` ​

```bash
# 手动挂载到 /mnt
$ sudo mount /dev/nvme0n1p6 /mnt

# 检查实际存在的快照
$ ls -la /mnt/timeshift-btrfs/snapshots/
总计 0
drwxr-xr-x 1 root root 114  8月 7日 16:00 .
drwxr-xr-x 1 root root 210  8月 7日 16:00 ..
drwxr-xr-x 1 root root  32  8月 7日 16:39 2025-08-05_22-44-05
drwxr-xr-x 1 root root  20  8月 7日 15:00 2025-08-07_14-31-24
drwxr-xr-x 1 root root  20  8月 7日 16:00 2025-08-07_15-08-42

# 删除不存在的快照条目
sudo rm -rf /mnt/timeshift-btrfs/snapshots/2025-08-05_22-44-05 2>/dev/null

# 检查是否删除
$ ls -la /mnt/timeshift-btrfs/snapshots/
总计 0
drwxr-xr-x 1 root root  76  8月 7日 16:45 .
drwxr-xr-x 1 root root 210  8月 7日 16:00 ..
drwxr-xr-x 1 root root  20  8月 7日 15:00 2025-08-07_14-31-24
drwxr-xr-x 1 root root  20  8月 7日 16:00 2025-08-07_15-08-42
# `2025-08-05_22-44-05`没有了！删了！

# 卸载
sudo umount /mnt
```

‍
