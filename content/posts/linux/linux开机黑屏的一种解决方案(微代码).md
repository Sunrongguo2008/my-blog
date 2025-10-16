+++
date = '2025-08-03'
draft = false
title = 'linux开机黑屏的一种解决方案'
tags = ["linux"]
+++
## 1. 特点

**linuxmint (ubuntu)** ：  
开机黑屏。切到高级设置进入，卡在 loading initial ramdisk 界面；但是可以进入高级设置-recovery mode，在里面选择 resume 可以进入系统。  
**Manjaro (arch)** ：  
安装完系统开机就黑屏，recovery mode 进不去  
**opensuse**：  
安装完系统开机就黑屏，其他未知  
**临时解决方案**：  
在 grub 中，对着启动项按下 e，找到以 ` linux /boot/vmlinuz...` 开头的行，在行尾添加 `dis_ucode_ldr` 可以开机
## 2. 原因
Intel 微代码（Microcode）更新与内核或硬件兼容性冲突
## 3. 解决方法
### 3.1 方法 1：改 grub（治标不治本，不过所有 linux 都可以用）：
1. **编辑 GRUB 配置文件**：

```bash
sudo nano /etc/default/grub
```
2. **在**​**​ `GRUB_CMDLINE_LINUX` ​**​**中添加禁用参数**：  
找到以 `GRUB_CMDLINE_LINUX=` 开头的行，在引号内添加 `dis_ucode_ldr` 参数：

```bash
GRUB_CMDLINE_LINUX="...原有参数... dis_ucode_ldr"
```

**示例**（修改后可能类似这样）：

```bash
GRUB_CMDLINE_LINUX="quiet splash dis_ucode_ldr"
```
3. **保存并更新 GRUB**：  
按 `Ctrl+X` → 输入 `Y` → 回车保存文件，然后执行：

```bash
sudo update-grub
```
4. **重启系统**：

```bash
sudo reboot
```

### 3.2 **方法 2：禁止微代码集成到 initramfs**（linuxmint 可行、Manjaro 不可行）

若降级无效，需彻底阻止微代码被加载到初始内存盘：

1. 编辑微代码配置文件：

```bash
sudo nano /etc/default/intel-microcode
```
2. 修改为以下内容：

```conf
IUCODE_TOOL_INITRAMFS=no
```
3. 重新生成 initramfs：

```bash
sudo update-initramfs -u  # Ubuntu/Debian
sudo dracut -f  # Fedora/openSUSE
```

### 3.3 **方法 3：降级 Intel 微代码**（没试过）

1. 进入 Recovery Mode 选择 `resume` 进入系统。
2. 执行以下命令安装旧版微代码包：

```bash
sudo apt install intel-microcode=3.20180312.0~ubuntu18.04.1  # Ubuntu 18.04 示例版本

```

### 3.4 试过没用的方法（别看这个，Manjaro 没用）

在 Arch Linux 中阻止 Intel 微代码打包到 initramfs，通过配置 `mkinitcpio` 工具实现。

**移除** **​ `microcode` ​**​ **钩子**

- 编辑 `/etc/mkinitcpio.conf`：

```bash
sudo nano /etc/mkinitcpio.conf
```
- 在 `HOOKS` 行中找到 `microcode` 并将其**删除**。例如修改前：

```
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)
```

修改后：

```
HOOKS=(base systemd autodetect modconf kms keyboard sd-vconsole block filesystems fsck)
```
- 保存文件（`Ctrl+O` → `Enter` → `Ctrl+X`）。

**重新生成 initramfs**

执行以下命令应用更改：

```bash
sudo mkinitcpio -P
```

此命令会为所有已安装的内核重新生成 initramfs 镜像（如 `initramfs-linux.img`），不再包含微代码。

**验证是否生效**

- 检查生成的 initramfs 内容：

```bash
lsinitcpio /boot/initramfs-linux.img | grep -i microcode
```

若输出中无 `intel-ucode.img` 等微代码文件，则表示已成功移除。

## 4.  **⚠️不要直接删除**​**​ `intel_ucode` ​**​**包**

### 4.1 如果已经删了

**开机会提示缺失**​**​ `ucode.img` ​**​**蓝屏（Manjaro）**

#### 4.1.1 原因

- 卸载 `intel-ucode` 后，`/boot/intel-ucode.img` 文件被删除。
- 但 GRUB 配置未更新，仍包含对该文件的引用，导致引导时因文件缺失而崩溃。

#### 4.1.2 解决方案：更新 GRUB 配置（没试过，来自 DeepSeek）

需手动移除对 `intel-ucode.img` 的依赖，并重新生成引导文件：

1. **使用 Arch Linux 安装介质启动**  
从 USB 安装盘启动，选择 “Troubleshooting” → “Boot Arch Linux” 进入临时系统。
2. **挂载原系统分区**  
假设根分区为 `/dev/nvme0n1p2`，EFI 分区为 `/dev/nvme0n1p1`：

```bash
mount /dev/nvme0n1p2 /mnt          # 挂载根分区
mount /dev/nvme0n1p1 /mnt/boot     # 挂载 EFI 分区
arch-chroot /mnt                   # 切换到原系统环境
```
3. **修改 GRUB 配置文件**  
编辑 `/boot/grub/grub.cfg` 或重新生成配置：

- **方法 1：重新生成 GRUB 配置（推荐）**

```bash
grub-mkconfig -o /boot/grub/grub.cfg  # 自动清理无效的 ucode 引用
```
- **方法 2：手动编辑（若自动生成无效）**   
找到所有包含 `initrd /boot/intel-ucode.img` 的行，删除该部分，仅保留对 `initramfs` 的引用。例如：  
原内容：

```ini
initrd  /boot/intel-ucode.img /boot/initramfs-linux.img
```

修改为：

```ini
initrd  /boot/initramfs-linux.img
```
4. **退出并重启**

```bash
exit
umount -R /mnt
reboot
```

### 4.2 可能的正确删除方法

若你坚持卸载微码（**一般不推荐**，除非确定 CPU 无需更新），需确保彻底清理：

1. **卸载包并清除残留**

```bash
sudo pacman -Rns intel-ucode  # 卸载并删除依赖配置
sudo rm /boot/intel-ucode.img # 确保文件被删除
```
2. **重建 initramfs**（部分内核需此步骤）

```bash
sudo mkinitcpio -P  # 重新生成 initramfs，避免依赖微码
```
