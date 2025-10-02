+++
date = '2025-08-09'
draft = false
title = 'linux应用程序无法开机自启可能的解决方案'
tags = ["linux", "blog"]
+++

最近遇到了一个的 Linux 启动问题：明明设置了开机自启动的应用程序，结果每次开机后都没运行起来。手动启动是正常的，但就是自动启动失效。

## 发现问题：应用程序启动不了

### 症状

我开机进入桌面后发现：

- ABDownloadManager 没有自动启动
- Notification\_monitor 脚本也没运行
- 这些应用都放在外接的 SSD 上（路径是 `/run/media/s/SSD/...`）
- 奇怪的是，手动双击或者命令行运行都是正常的

## 二、分析问题

### 查看启动日志

```bash
journalctl -b -e
```

这个命令会显示本次启动的日志，`-e` 参数直接跳到最后面，方便查看最新的信息。

在日志中找到了这些关键信息：

```
8月 09 11:54:56 s-pc systemd-xdg-autostart-generator[893]: Exec binary '/run/media/s/SSD/Linux_Tools/ABDownloadManager/bin/ABDownloadManager' does not exist: No such file or directory

8月 09 11:54:56 s-pc systemd-xdg-autostart-generator[893]: /home/s/.config/autostart/com.abdownloadmanager.desktop: not generating unit, executable specified in Exec= does not exist.

8月 09 11:54:58 s-pc udisksd[951]: **Mounted /dev/nvme0n1p4 at /run/media/s/SSD on behalf of uid 1000**
```

- 11:54:56 - 系统尝试启动应用，发现路径不存在
- 11:54:58 - udisksd 完成 SSD 挂载，应用启动需要的路径被创建

### 找到原因了

系统想启动应用时，SSD 还没挂载好，路径不存在，启动失败。

## 解决方案

### 方法一：fstab 自动挂载

##### 让 SSD 开机就挂载

这个 SSD 每天都要用，不如让它开机就自动挂载。这样就能提前解决挂载问题，而不是等待挂载完成。

##### 简单配置

首先获取 SSD 的 UUID：

```bash
sudo blkid /dev/nvme0n1p4
```

输出类似：

```
/dev/nvme0n1p4: LABEL="SSD" BLOCK_SIZE="512" UUID="9FAB0FC4562CB861" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="af50b57f-7ccb-42bb-a3b5-7256ef6e3933"
```

安装 ntfs-3 g：

```bash
sudo pacman -S ntfs-3g
```

然后编辑 fstab：

```bash
sudo nano /etc/fstab
```

在文件末尾添加：

```
UUID=9FAB0FC4562CB861 /run/media/s/SSD ntfs-3g defaults,uid=1000,gid=1000,windows_names,nofail 0 0
```

```bash
systemctl daemon-reload
```

配置完成后，重启系统，SSD 就会在开机时自动挂载，autostart 也能正常工作了。

### 方法二：创建 systemd 用户服务

这个方法有个缺点：很多应用自带的开机自启并不是**创建 systemd 用户服务**的方案，需要对每个应用进行修改

而且以下内容由 AI（Claude）撰写，我也没试过，不知道是否可行

#### Systemd 特点

传统的 autostart 机制比较"死板"：到时间就启动，不管条件是否满足。而 systemd 服务更智能，可以"等条件满足再启动"。

关键是 `ExecStartPre` 指令，它让程序在启动前先执行一个预检查脚本。我们可以用它来主动等待挂载完成：

```bash
ExecStartPre=/bin/bash -c 'while [ ! -d "/run/media/s/SSD" ]; do sleep 1; done'
```

这行代码的意思是：不断检查 SSD 挂载目录是否存在，不存在就等 1 秒再检查，直到目录存在才继续执行主程序。

#### 创建服务配置

以 ABDownloadManager 为例，创建 systemd 用户服务：

```bash
# 创建服务目录
mkdir -p ~/.config/systemd/user

# 创建服务配置文件
cat > ~/.config/systemd/user/abdownloadmanager.service << EOF
[Unit]
Description=AB Download Manager
After=graphical-session.target
Wants=graphical-session.target

[Service]
Type=simple
ExecStartPre=/bin/bash -c 'while [ ! -d "/run/media/s/SSD" ]; do sleep 1; done'
ExecStart=/run/media/s/SSD/Linux_Tools/ABDownloadManager/bin/ABDownloadManager
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
EOF
```

配置说明：

- ​ `After=graphical-session.target`：确保在图形界面准备好后再启动
- ​ `ExecStartPre`：启动前等待挂载完成
- ​ `Restart=on-failure`：如果程序崩溃会自动重启
- ​ `RestartSec=5`：重启前等待 5秒

同样为 notification\_monitor 创建服务：

```bash
cat > ~/.config/systemd/user/notification_monitor.service << EOF
[Unit]
Description=Notification Monitor
After=graphical-session.target
Wants=graphical-session.target

[Service]
Type=simple
ExecStartPre=/bin/bash -c 'while [ ! -d "/run/media/s/SSD" ]; do sleep 1; done'
ExecStart=/run/media/s/SSD/Linux_Tools/notification_monitor/notification_monitor.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
EOF
```

#### 部署步骤

1. **重载 systemd 配置**：

    ```bash
    systemctl --user daemon-reload
    ```
2. **启用服务**：

    ```bash
    systemctl --user enable abdownloadmanager.service
    systemctl --user enable notification_monitor.service
    ```
3. **清理旧的 autostart 配置**：

    ```bash
    rm ~/.config/autostart/com.abdownloadmanager.desktop
    rm ~/.config/autostart/notification_monitor.sh.desktop
    ```
4. **测试效果**（可选）：

    ```bash
    # 立即启动测试
    systemctl --user start abdownloadmanager.service

    # 查看服务状态
    systemctl --user status abdownloadmanager.service

    # 查看服务日志
    journalctl --user -u abdownloadmanager.service -f
    ```

重启系统后，应用程序就能正常自启动了。
