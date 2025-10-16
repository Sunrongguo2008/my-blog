+++
date = '2025-10-02T06:20:10+08:00'
draft = false
title = 'Hello Blowfish'
tags = ["测试"]
+++
## 用[Blowfish](https://blowfish.page/zh-cn/)创建了博客！泰裤辣！😄

## 字体修改
从网站找到代码，比如：https://fonts.zeoseven.com/items/442/#embed

选中 `HTML强化`，用里面的代码进行导入

{{< alert "circle-info" >}}
在我这里，`常规CSS`无法使用，所以用`HTML强化`

`英文子集化`可以让所有英文显示为指定字体

`全部版本`可以展示所用可用的字型，可与选择适合的字型链接替代`HTML强化`中的链接
{{< /alert  >}}
![image.png](https://cdn.jsdelivr.net/gh/Sunrongguo2008/picture/obsidian/20251016125133509.png)

新建 /themes/blowfish/layouts/partials/extend-head.html

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
## 修改侧栏(TOC)和标题颜色
新建 /assets/css/custom.css
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