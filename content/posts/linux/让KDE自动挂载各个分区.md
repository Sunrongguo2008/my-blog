+++
date = '2025-08-04'
draft = false
title = '让KDE自动挂载各个分区'
tags = ["linux"]
+++

## 1. 前景提要

首先，KDE 有这个界面

![](https://cdn.jsdelivr.net/gh/Sunrgongguo2008/picture/obsidian/202510030927580.png)

但是用过就会知道，即使你都勾选上了，它照样不会开机自动挂载

[forum.ubuntu.com.cn/viewtopic.php?t=50539](https://forum.ubuntu.com.cn/viewtopic.php?t=50539) 提及了这个问题，虽然没有提到治本的解决方法，但是一个哥们提醒了我：

![](https://cdn.jsdelivr.net/gh/Sunrongguo2008/picture/obsidian/202510030927699.png)

调查发现是 `uudisksctl mount -b xxxx` 这个命令需要 root，而 KDE 似乎没有这个权限，也不会弹窗索要，导致部分分区（可能只限**不可移动硬盘**上的**NTFS 分区**）无法开机自动挂载

## 2. 解决方案

### 2.1 换一种方式挂载（省心）

常见方法有（Deepseek 给的）：

- 通过 `/etc/fstab` 配置
- 使用 `systemd` 自动挂载
- 使用 `pmount` 工具

### 2.2 让 `udisksctl mount -b xxxx` 这类命令无需 root 权限执行

#### 2.2.1 失败的方案

**我也不知道为啥不行，网上搜的都这样干**

1. 将用户加入 `storage` 用户组

```bash
sudo usermod -aG storage $(whoami)  # 将当前用户加入 storage 组
```

2. 验证组是否生效：运行 `groups` 命令，检查输出是否包含 `storage`。

3. 注销并重新登录使组权限生效

#### 2.2.2 可行的方案：配置 PolicyKit 规则（推荐）

1. 创建 PolicyKit 规则文件：

```bash
sudo nano /etc/polkit-1/rules.d/50-udisks2.rules
```

2. 添加以下内容：

```javascript
// 允许所有用户挂载/卸载块设备
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.udisks2.filesystem-mount" ||
        action.id == "org.freedesktop.udisks2.filesystem-unmount") {
        return polkit.Result.YES;
    }
});

```

或者下面这个（没试过）：

```javascript
// 更精细的控制，只允许特定用户组
polkit.addRule(function(action, subject) {
    if ((action.id == "org.freedesktop.udisks2.filesystem-mount" ||
         action.id == "org.freedesktop.udisks2.filesystem-unmount") &&
        subject.isInGroup("plugdev")) {
        return polkit.Result.YES;
    }
});
```

重启以后就能发现分区被自动挂载了！

‍
