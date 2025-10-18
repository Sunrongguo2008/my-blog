+++
date = '2025-10-16'
draft = false
title = '优雅地自定义 Hugo Blowfish 主题（或任何 Git 子模块）'
tags = ["blog"]
description = "教你如何优雅地自定义 Hugo Blowfish 主题：通过 Fork 子模块保存修改、保持与上游同步，解决 Git 子模块推送冲突。适用于所有需要自定义第三方主题或依赖的场景。"
+++

{{< alert "circle-info" >}}少数内容由AI生成{{< /alert  >}}
## 1 前言

使用 Hugo 的 Blowfish 主题时，我们经常需要调整一些样式、字体或配置文件，以满足个性化需求。  
然而，当主题是以 **Git 子模块（submodule）** 的形式存在时，直接修改主题文件会带来一个常见问题：  
**这些改动无法正常推送到远程仓库**，从而导致更新或协作困难。

本文将介绍一种通用、优雅的解决方案：  
在保留自定义修改的同时，仍能轻松同步上游更新。

> 本文方法不仅适用于 Blowfish，也适用于任何以 Git 子模块引入的第三方项目。

---

## 2 问题场景

假设你修改了 Blowfish 主题（如调整字体），随后在推送主仓库时遇到错误：

```bash
下列子模组路径所包含的修改在任何远程源中都找不到：
  themes/blowfish

请尝试
	git push --recurse-submodules=on-demand
	
或者进入到子目录执行

	git push

以推送至远程。

致命错误：正在终止。
```

原因很简单：  
**子模块中的修改仅存在于本地**，Git 无法在远程找到对应的提交。

这通常发生在以下情况：

- 你在子模块中直接修改了文件；
    
- 子模块处于 `detached HEAD` 状态；
    
- 没有将修改推送到任何可访问的远程仓库。
    

---

## 3 解决方案：Fork 并建立自定义子模块

核心思路是：

> 为主题（或子模块）创建一个属于你自己的 Fork，  
> 在其中保存自定义修改，  
> 同时保持从官方仓库获取更新的能力。

{{< alert "circle-info" >}}
还有个方法：删除子模块，把字体修改直接保存到主仓库中

不在本文中介绍，请移步至《Blowfish 文档外自定义指南》-字体修改 的偏后部分 
{{< /alert  >}}

---

### 3.1 第一步：检查当前子模块状态

进入主题目录，确认修改情况：

```bash
cd themes/blowfish
git status
git log --oneline -5
```

若出现类似输出：

```
头指针分离自 3e652b37
无文件要提交，干净的工作区

770a2a67 (HEAD) 自定义字体
d0db9190 (origin/main, origin/HEAD) fix user.json
```

`分离自`说明当前处于 **detached HEAD** 状态（即未在任何分支上）。  
我们需要将这个提交移动到自己的分支中。

---

### 3.2 第二步：Fork 官方主题仓库

1. 打开 Blowfish 官方仓库  
    [https://github.com/nunocoracao/blowfish](https://github.com/nunocoracao/blowfish)
    
2. 点击右上角 **Fork** 按钮
    
3. 将仓库 Fork 到自己的 GitHub 账号
    

---

### 3.3 第三步：配置子模块使用你的 Fork

在子模块目录中执行以下命令（这里以`custom-font`为分支名称）：

```bash
# 确认当前远程
git remote -v

# 添加你的 Fork
git remote add myfork https://github.com/你的用户名/blowfish.git

# 创建分支保存自定义修改
git checkout -b custom-font 770a2a67

# 推送到你的 Fork
git push myfork custom-font
```

---

### 3.4 第四步：让主仓库指向你的 Fork

回到 Hugo 博客根目录：

```bash
cd ../..
```

编辑 `.gitmodules` 文件，将内容改为：

```ini
[submodule "themes/blowfish"]
	path = themes/blowfish
	url = https://github.com/你的用户名/blowfish.git
```

同步配置：

```bash
git submodule sync
cd themes/blowfish
git remote set-url origin https://github.com/你的用户名/blowfish.git
```

---

### 3.5 第五步：提交并推送修改

```bash
cd ../..
git add .gitmodules themes/blowfish
git commit -m "Switch Blowfish submodule to personal fork"
git push
```

如果子模块的修改尚未推送，补上：

```bash
cd themes/blowfish
git push --set-upstream origin custom-font
cd ../..
git add themes/blowfish
git commit -m "Update submodule reference"
git push
```

---

## 4 主题更新策略

有了自己的 Fork 后，你可以随时同步官方更新，而不丢失自定义内容。


### 4.1 推荐方法：合并上游更新

```bash
cd themes/blowfish
git remote add upstream https://github.com/nunocoracao/blowfish.git
git fetch upstream
git merge upstream/main
git push origin custom-font
```

这种方式最安全，能保留你的修改记录。



### 4.2 其他方法：使用变基保持线性历史

```bash
cd themes/blowfish
git fetch upstream
git rebase upstream/main
git push --force origin custom-font
```

适合对 Git 操作较熟悉的用户，能保持干净的历史，但注意变基会改写提交记录。


## 5 F&Q

### 5.1 如何查看上游更新？

```bash
cd themes/blowfish
git fetch upstream
git log HEAD..upstream/main --oneline
```

### 5.2 如何测试更新是否正常？

```bash
hugo server -D
```

确认主题在本地运行正常后再推送。

### 5.3 如何回退到官方版本？

```bash
cd themes/blowfish
git checkout main
cd ../..
git submodule sync
git add .gitmodules themes/blowfish
git commit -m "Revert to official blowfish theme"
git push
```

---

## 6 最佳实践建议

1. **定期同步上游**：每月更新一次主题，防止版本落后。
    
2. **尽量少改主题代码**：优先通过配置或自定义 CSS 实现修改。
    
3. **记录修改内容**：在 Fork 的 README 中注明改动目的。
    
4. **使用语义化分支名**：如 `custom-font`、`custom-layout`，方便区分用途。
    
5. **备份关键改动**：重要样式或配置可单独备份，防止冲突丢失。
    

---

## 7 总结

通过 Fork 子模块的方式，我们可以：

- **保留自定义修改**
    
- **保持上游更新能力**
    
- **避免 Git 推送冲突**
    

这种做法不仅适用于 Hugo Blowfish 主题，也同样适用于任何以 Git 子模块引入的第三方项目。

**核心步骤回顾：**

1. Fork 原仓库
    
2. 新建分支保存自定义修改
    
3. 修改 `.gitmodules` 指向你的 Fork
    
4. 定期同步 upstream 保持更新
    
