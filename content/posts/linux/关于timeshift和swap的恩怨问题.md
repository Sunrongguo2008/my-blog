+++
date = '2025-08-09'
draft = false
title = '关于timeshift和swap的恩怨问题'
tags = ["linux"]
+++


在 btrfs 下，创建 swap 文件而不是 swap 分区时：

**把 swap 文件放到**​ **​ `/swap` ​**​**子卷中：**  因为 timeshift 不会备份 `/swap` 子卷，导致恢复快照时，swap 文件 `/swap/swapfile` 会消失  
**不把 swap 文件放到子卷中：**  timeshift 备份时，swapfile 被内核使用，会导致 `E: ERROR: Could not create subvolume: Text file busy`，导致无法创建快照，出现[这个情况](https://forum.archlinuxcn.org/t/topic/14195)

所以，最佳实践可能是**创建 swap 分区**而不是**swap 文件** ([参考来源](https://forum.archlinuxcn.org/t/topic/14276))

或者仍然**把 swap 文件放到**​ **​ `/swap` ​**​**子卷中**，timeshift 恢复快照后，手动把 /swap 目录删掉、把原来的 /swap 子卷移动过来

