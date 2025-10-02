+++
date = '2025-08-06'
draft = false
title = '我遇到的winapps问题&解决方案'
tags = ["linux", "blog"]
+++

2025-08-06 21:43

都是我折腾 [winapps](https://github.com/winapps-org/winapps) 时遇到的问题，费了三整天才慢慢解决，故分享出来让大家少踩坑。

## 1. 1. RDP 花费的连接时间太长，导致脚本超时

### 1.1 特点

运行 50 秒才启动

```bash
$ xfreerdp3 /u:"Administrator" /p:"1" /v:192.168.122.122 /cert:tofu
[08:48:09:871] [26120:0000660b] [WARN][com.freerdp.client.x11] - [load_map_from_xkbfile]:     : keycode: 0x08 -> no RDP scancode found
[08:48:09:871] [26120:0000660b] [WARN][com.freerdp.client.x11] - [load_map_from_xkbfile]: ZEHA: keycode: 0x5D -> no RDP scancode found
[08:48:09:893] [26120:0000660b] [WARN][com.freerdp.crypto] - [verify_cb]: Certificate verification failure 'self-signed certificate (18)' at stack position 0
[08:48:09:893] [26120:0000660b] [WARN][com.freerdp.crypto] - [verify_cb]: CN = DESKTOP-G8N9MT9
[08:48:32:930] [26120:0000660b] [ERROR][com.winpr.sspi.Kerberos] - [kerberos_AcquireCredentialsHandleA]: krb5glue_get_init_creds (Cannot contact any KDC for realm 'ATHENA.MIT.EDU' [-1765328228])
[08:48:55:956] [26120:0000660b] [ERROR][com.winpr.sspi.Kerberos] - [kerberos_AcquireCredentialsHandleA]: krb5glue_get_init_creds (Cannot contact any KDC for realm 'ATHENA.MIT.EDU' [-1765328228])
[08:48:55:967] [26120:0000660b] [WARN][com.freerdp.core.connection] - [rdp_client_connect_auto_detect]: expected messageChannelId=1008, got 1003
[08:48:55:967] [26120:0000660b] [WARN][com.freerdp.core.license] - [license_read_binary_blob_data]: license binary blob::type BB_ERROR_BLOB, length=0, skipping.
[08:48:55:992] [26120:0000660b] [WARN][com.freerdp.core.connection] - [rdp_client_connect_auto_detect]: expected messageChannelId=1008, got 1003
[08:48:55:999] [26120:0000660b] [INFO][com.freerdp.gdi] - [gdi_init_ex]: Local framebuffer format  PIXEL_FORMAT_BGRX32
[08:48:55:999] [26120:0000660b] [INFO][com.freerdp.gdi] - [gdi_init_ex]: Remote framebuffer format PIXEL_FORMAT_BGRA32
[08:48:55:000] [26120:0000660b] [INFO][com.freerdp.channels.rdpsnd.client] - [rdpsnd_load_device_plugin]: [static] Loaded fake backend for rdpsnd
[08:48:55:000] [26120:0000660b] [INFO][com.freerdp.channels.drdynvc.client] - [dvcman_load_addin]: Loading Dynamic Virtual Channel ainput
[08:48:55:000] [26120:0000660b] [INFO][com.freerdp.channels.drdynvc.client] - [dvcman_load_addin]: Loading Dynamic Virtual Channel rdpgfx
[08:48:55:001] [26120:0000660b] [INFO][com.freerdp.channels.drdynvc.client] - [dvcman_load_addin]: Loading Dynamic Virtual Channel disp
[08:48:55:001] [26120:0000660b] [INFO][com.freerdp.channels.drdynvc.client] - [dvcman_load_addin]: Loading Dynamic Virtual Channel rdpsnd
[08:48:55:123] [26120:000066fe] [INFO][com.freerdp.channels.rdpsnd.client] - [rdpsnd_load_device_plugin]: [dynamic] Loaded fake backend for rdpsnd
[08:48:56:947] [26120:000066fe] [WARN][com.freerdp.channels.drdynvc.client] - [check_open_close_receive]: {Microsoft::Windows::RDS::DisplayControl:9} OnOpen=(nil), OnClose=0x7f02c8b5ecc0
[08:48:56:965] [26120:000066fe] [INFO][com.freerdp.channels.rdpsnd.client] - [rdpsnd_load_device_plugin]: [dynamic] Loaded fake backend for rdpsnd
[08:48:58:969] [26120:000066fe] [WARN][com.freerdp.channels.drdynvc.client] - [check_open_close_receive]: {Microsoft::Windows::RDS::DisplayControl:9} OnOpen=(nil), OnClose=0x7f02c8b5ecc0
[08:48:58:979] [26120:00006608] [ERROR][com.freerdp.core] - [freerdp_abort_connect_context]: ERRCONNECT_CONNECT_CANCELLED [0x0002000B]

```

### 1.2 问题分析

日志中的关键错误：

```bash
[ERROR][com.winpr.sspi.Kerberos] - [kerberos_AcquireCredentialsHandleA]: krb5glue_get_init_creds (Cannot contact any KDC for realm 'ATHENA.MIT.EDU' [-1765328228])
```

FreeRDP 试图使用 Kerberos 认证，但无法联系到域控制器，导致多次超时重试。

### 1.3 解决方案

明确指定本地认证

#### 1.3.1 对于[测试RDP](https://github.com/winapps-org/winapps?tab=readme-ov-file#step-4-test-freerdp) 的命令

修改成：

```bash
xfreerdp3 /u:"Administrator" /p:"1" /v:192.168.122.122 /cert:tofu /d:WORKGROUP
```

#### 1.3.2 对于安装脚本的超时失败

通过阅读安装脚本，发现 `/d:` 参数由 `RDP_DOMAIN` 决定

对应到 `~/.config/winapps/winapps.conf` 上，也就是修改成：

```vim
# [WINDOWS DOMAIN]
# DEFAULT VALUE: '' (BLANK)
RDP_DOMAIN="WORKGROUP"
```

### 1.4 备注

#### 1.4.1 关于 AI

大部分 AI（DeepSeek Qwen）会给出 `xfreerdp3 /u:"Administrator" /p:"1" /v:192.168.122.122 /cert:tofu /sec:rdp` 或者 `xfreerdp3 /u:"Administrator" /p:"1" /v:192.168.122.122 /cert:tofu /auth-only:ntlm` 一个命令会报错，一个仍然需要等待大量时间

还是 Claude 牛逼，虽然仍然给出上面的错误指令，但是也给出了正确方案 `xfreerdp3 /u:"Administrator" /p:"1" /v:192.168.122.122 /cert:tofu /d:WORKGROUP` ​

#### 1.4.2 Claude 给出的其他方案（不知道是否可行）

在 Windows 虚拟机中：

1. 打开"组策略编辑器" (gpedit. Msc)
2. 导航到：计算机配置 → 管理模板 → Windows 组件 → 远程桌面服务 → 远程桌面会话主机 → 安全
3. 禁用"要求使用网络级别身份验证对远程连接的用户进行身份验证"

或者直接 cmd 命令：

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0
```

## 2. 中文无法输入

### 2.1 特点

无法显示输入法候选词窗口

### 2.2 原因&解决方案

1. [Windows原生输入法兼容性不好](https://github.com/winapps-org/winapps/issues/43#issuecomment-2124032568)。**建议换为微信输入法，百度输入法等**

2. Windows 开机后在 winapps 连接前使用了其他连接（KVM 的、[测试RDP](https://github.com/winapps-org/winapps#step-4-test-freerdp) 时的），导致输入法候选词窗口跑到其他连接上没回来（我是这样理解的）。**一般重启可解决**

## 3. 中文输入乱码

### 3.1 特点

**按键映射错误：**

类似这样

- 打开记事本，输入 `qw`，结果记事本上的内容为<u>​`s'd`​</u>，候选词为“上的”
- 同理，`Backspace`, `Delete` 等按键也映射异常，导致无法退格删除等
- 英文就能正常输入

### 3.2 原因&解决方案

你被 [README](https://github.com/winapps-org/winapps?tab=readme-ov-file#step-3-create-a-winapps-configuration-file) 的一句话骗了：

> ### 3.2.1 Step 3: Create a WinApps Configuration File 第 3 步：创建 WinApps 配置文件
>
> ......
>
> #### 3.2.1.1 Configuration Options Explained
>
> - .........
> - **<u>To enable non-English input and seamless language switching, you can try adding</u>**   **<u>​`/kbd:unicode`​</u>**​  **<u>to</u>**  **<u>​`RDP_FLAGS`​</u>**​ **<u>. This ensures client inputs are sent as Unicode sequences.</u>**
> - ......

里面的

> **To enable non-English input and seamless language switching, you can try adding**   **​ `/kbd:unicode` ​**​  **to**  **​ `RDP_FLAGS` ​**​ **. This ensures client inputs are sent as Unicode sequences.**
>
> 为了启用非英语输入并实现无缝的语言切换，您可以尝试在 `/kbd:unicode` 添加到 `RDP_FLAGS` 中。这确保客户端输入以 Unicode 序列发送。

就是它干的！不要加 `/kbd:unicode`！血的教训！

明明是

> To ~~enable~~ **prevent** non-English input and ~~seamless~~ **difficult** language switching, you can try adding  `/kbd:unicode`  to  `RDP_FLAGS`. This ensures client inputs are sent as Unicode ~~sequences~~ **garbled**.
>
> 为了~~启用~~​**阻止**非英语输入并实现~~无缝~~​**困难**的语言切换，您可以尝试在 `/kbd:unicode` 添加到 `RDP_FLAGS` 中。这确保客户端输入以 Unicode ~~序列~~​**乱码**发送。

可能对其他语言有用，对中文还是算了吧

## 4. 已经被我修复 ~~待官方修复~~ ~~没解决~~ 的问题

**注：已经**​**[提交了PR](https://github.com/winapps-org/winapps/pull/629)**​ **，**​~~**等待官方修复 ing...**~~  **已合并，BUG已修复**

### 4.1 发现问题

直接双击打开文件（文件挂载在 `/run/media/s/松果U盘/DeepSeek.pptx`）

![屏幕截图_20250806_213526](https://img2024.cnblogs.com/blog/3684112/202508/3684112-20250809104931535-65971359.png "图 1")

但是用①选择文件打开，正常。之后直接点②也能直接打开。

奇了怪了

![image](https://img2024.cnblogs.com/blog/3684112/202508/3684112-20250809104932461-21259550.png "图 2")

### 4.2 分析问题

仔细观察发现：

图一的路径是 `\\tsclient\media\松果U盘\DeepSeek.pptx` ​

图二的路径是 `\\tsclient\media\s\松果U盘\DeepSeek.pptx` ​

也就是说，winapps 启动的时候挂载的是 `\\tsclient\media\s\松果U盘\`, 但是启动 PowerPoint 的命令行却指向了 `\\tsclient\media\松果U盘\`，导致 PowerPoint 找不到文件

花了一晚上时间，发现是 `~/.local/bin/winapps` 的正则匹配有问题

第 668 行的 `-e 's|<sup>('"${REMOVABLE_MEDIA//|/\|}"')/[</sup>/]*|\\\\tsclient\\media|' \` ​

应该改成 `-e 's|^'"${REMOVABLE_MEDIA}"'|\\\\tsclient\\media|' \` ​

**已经**​**[提交了PR](https://github.com/winapps-org/winapps/pull/629)**​ **，**​~~**等待官方修复 ing...**~~   **已合并**

### 4.3 解决问题

编辑 `~/.local/bin/winapps`，把第 668 行改成：

```bash
                -e 's|^'"${REMOVABLE_MEDIA}"'|\\\\tsclient\\media|' \
```

大功告成！
