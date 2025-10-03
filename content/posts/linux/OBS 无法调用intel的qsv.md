+++
date = '2025-08-11'
draft = false
title = 'OBS 无法调用intel的qsv（Arch）'
tags = ["linux"]
+++

在 [Arch Linux 中文论坛询问](https://forum.archlinuxcn.org/t/topic/14285)，有人说：根据[ArchWiki](https://wiki.archlinux.org/title/FFmpeg#Intel_QuickSync_(QSV))，需要安装 libvpl 和 vpl-gpu-rt

```bash
sudo pacman -S libvpl
sudo pacman -S vpl-gpu-rt
```

