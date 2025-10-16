+++
date = '2025-10-02T06:20:10+08:00'
draft = false
title = 'Blowfish 文档外自定义指南'
tags = ["blog"]
+++
## 0.用[Blowfish](https://blowfish.page/zh-cn/)创建了博客！泰裤辣！😄
## 1.博客修改
{{< alert "circle-info" >}}
所有例子中，`/`指主仓库根目录

我的网站目录在`~/文/my-blog` ，因此`/themes/blowfish/`指的是`~/文/my-blog/themes/blowfish/`

{{< /alert  >}}

### 1.1修改侧栏(TOC)和标题颜色
效果：
![image.png](https://cdn.jsdelivr.net/gh/Sunrongguo2008/picture/obsidian/20251016224355331.png)
新建 `/assets/css/custom.css`

写入以下代码：
```CSS
/* 浅色模式下的彩虹色标题 */  
:where(h2):not(:where([class~="not-prose"],[class~="not-prose"] *)) {  
  color: #dc2626 !important; /* 红色 */  
}  
  
:where(h3):not(:where([class~="not-prose"],[class~="not-prose"] *)) {  
  color: #ea580c !important; /* 橙色 */  
}  
  
:where(h4):not(:where([class~="not-prose"],[class~="not-prose"] *)) {  
  color: #16a34a !important; /* 绿色 */  
}  
  
:where(h5):not(:where([class~="not-prose"],[class~="not-prose"] *)) {  
  color: #2563eb !important; /* 蓝色 */  
}  
  
:where(h6):not(:where([class~="not-prose"],[class~="not-prose"] *)) {  
  color: #9333ea !important; /* 紫色 */  
}  
  
/* 深色模式下的彩虹色标题 */  
.dark :where(h2):not(:where([class~="not-prose"],[class~="not-prose"] *)) {  
  color: #f87171 !important; /* 浅红色 */  
}  
  
.dark :where(h3):not(:where([class~="not-prose"],[class~="not-prose"] *)) {  
  color: #fb923c !important; /* 浅橙色 */  
}  
  
.dark :where(h4):not(:where([class~="not-prose"],[class~="not-prose"] *)) {  
  color: #4ade80 !important; /* 浅绿色 */  
}  
  
.dark :where(h5):not(:where([class~="not-prose"],[class~="not-prose"] *)) {  
  color: #60a5fa !important; /* 浅蓝色 */  
}  
  
.dark :where(h6):not(:where([class~="not-prose"],[class~="not-prose"] *)) {  
  color: #a78bfa !important; /* 浅紫色 */  
}



/* 浅色模式下的TOC彩虹色 */  
.toc ul ul li a { color: #dc2626 !important; } /* H2对应红色 */  
.toc ul ul ul li a { color: #ea580c !important; } /* H3对应橙色 */  
.toc ul ul ul ul li a { color: #16a34a !important; } /* H4对应绿色 */  
.toc ul ul ul ul ul li a { color: #2563eb !important; } /* H5对应蓝色 */  
.toc ul ul ul ul ul ul li a { color: #9333ea !important; } /* H6对应紫色 */  
  
/* 深色模式下的TOC彩虹色 */  
.dark .toc ul ul li a { color: #f87171 !important; } /* H2对应浅红色 */  
.dark .toc ul ul ul li a { color: #fb923c !important; } /* H3对应浅橙色 */  
.dark .toc ul ul ul ul li a { color: #4ade80 !important; } /* H4对应浅绿色 */  
.dark .toc ul ul ul ul ul li a { color: #60a5fa !important; } /* H5对应浅蓝色 */  
.dark .toc ul ul ul ul ul ul li a { color: #a78bfa !important; } /* H6对应浅紫色 */

```
### 1.2字体修改
#### 1.2.1优点
此方法和文档中的相比，能让访问者从国内第三方网站下载字体，提高字体加载速率

#### 1.2.2操作方法

从网站找到代码，比如：https://fonts.zeoseven.com/items/442/#embed

选中 `HTML强化`，用里面的代码进行导入

{{< alert "circle-info" >}}
在我这里，`常规CSS`无法使用，所以用`HTML强化`

`英文子集化`可以让所有英文显示为指定字体

`全部版本`可以展示所用可用的字型，可与选择适合的字型链接替代`HTML强化`中的链接
{{< /alert  >}}
![image.png](https://cdn.jsdelivr.net/gh/Sunrongguo2008/picture/obsidian/20251016125133509.png)

新建 `/themes/blowfish/layouts/partials/extend-head.html`

把内容放进去

最后差不多这个样：
```HTML
<!--导入LXGW WenKai Mono字体-->
<link rel="preload" as="style" crossorigin
    href="https://fontsapi.zeoseven.com/292/medium/result.css"
    onload="this.rel='stylesheet'"
    onerror="this.href='https://fontsapi-storage.zeoseven.com/292/medium/result.css'" />
<noscript>
    <link rel="stylesheet" href="https://fontsapi.zeoseven.com/292/medium/result.css" />
</noscript>

<!--导入Maple Mono NF-CN字体-->
<link rel="preload" as="style" crossorigin
    href="https://fontsapi.zeoseven.com/442/main/result.css"
    onload="this.rel='stylesheet'"
    onerror="this.href='https://fontsapi-storage.zeoseven.com/442/main/result.css'" />
<noscript>
    <link rel="stylesheet" href="https://fontsapi.zeoseven.com/442/main/result.css" />
</noscript>

<style>
/*
无需专门导入字体，让所有英文显示为Maple Mono NF-CN字体
@font-face {
    font-family: "ZSFT-ENMin-442";
    src: url("https://fontsapi.zeoseven.com/442/main.woff2") format('woff2'),
        url("https://fontsapi-storage.zeoseven.com/442/main.woff2") format('woff2');
    unicode-range: U+0061-007A, U+0041-005A, U+0030-0039, U+002E, U+002C, U+0021, U+003F, U+003A, U+003B, U+002D;
}
*/

/* (需专门导入字体)让所有正文显示为LXGW字体 */
    body {
        font-family: "LXGW WenKai";
        font-weight: normal;
    }

/* (需专门导入字体)让所有代码块显示为Maple Mono NF-CN字体 */
code, kbd, samp, pre {
    font-family: "Maple Mono NF CN";
}

</style>
```

#### 1.2.3注意事项
如果你用git下载/管理的blowfish主题，修改`/themes/blowfish`（添加了`/themes/blowfish/layouts/partials/extend-head.html`）会遇到子模块冲突
```bash
下列子模组路径所包含的修改在任何远程源中都找不到：
  themes/blowfish

请尝试

	git push --recurse-submodules=on-demand

或者进入到子目录执行

	git push

以推送至远程。

致命错误：正在终止。
~/文/my-blog (main|↑1|✔) [128]$ 
```
你有两个选择:

**选项 A**: 保留你的字体修改,推送到子模块的 fork/分支

请见站内文章 《优雅地自定义 Hugo Blowfish 主题（或任何 Git 子模块）》

**选项 B**: 删除子模块，把字体修改直接保存到主仓库中
```bash
# 1. 回到主仓库根目录
cd ~/文/my-blog

# 2. 移除子模块配置(但保留文件)
git rm --cached themes/blowfish
rm -rf .git/modules/themes/blowfish

# 3. 编辑 .gitmodules 文件,删除 blowfish 相关配置
nano .gitmodules  # 或用你喜欢的编辑器删除 blowfish 部分

# 4. 如果 .gitmodules 为空,删除它
rm .gitmodules  # 如果没有其他子模块的话

# 5. 将 themes/blowfish 作为普通目录添加
git add themes/blowfish
git add .gitmodules  # 如果修改了的话

# 6. 提交更改
git commit -m "Convert blowfish theme from submodule to regular directory"
git push