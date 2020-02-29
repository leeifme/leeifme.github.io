---
title: LaTeX + Sublime 制作一份优雅的简历
date: 2017-09-02 16:12:19
tags:
  - 安利系列
  - 百宝箱
categories:
  - 
---

>## 前言
>
>阿一古，终究是抓到了学期末的尾巴，
>
>时间越近心越慌，到了大三最后，周围拥有革命友谊的同志们，也各自选择了不知道适不适合自己的道路，
>
>又到了人生分叉路上，不知是不是只有我在路口惶恐不安

## 简历错误的打开方式

**使用 PDF 以外的格式**，如 PPT、DOCX、HTML。PDF 是我目前看到兼容性最好的格式，因为你不能保证你花里胡哨的简历在其他人的电脑上打开不会有乱码的问题。要知道 Windows 上写的 TXT 在 Mac OS 上打开都会有编码问题。DOCX 就更不用说了，有些公司标配的 Office 是 LibreOffice 。PPT 和 HTML 就更不想说了，虽然转场很帅，当你考虑过 HR 打开它的心情吗。

还有，

简历的文件命名尽量包含重要信息，突出重点，如：李小明-Android工程师-简历. pdf；

## LaTeX + Sublime

`LaTeX`，是一种基于TEX的排版系统，较多的运用在论文编写排版中，网上有很多介绍，我就不介绍了，

为什么要选择LaTeX，因为我喜欢它的一套简历模版`modercv `,显示效果是这样：

- casual

  ![casual](http://orj2jcr7i.bkt.clouddn.com/leeifme/blog/techcausl.png)

<!--more -->

- assic

  ![classic](http://orj2jcr7i.bkt.clouddn.com/leeifme/blog/techclassic.png)

- oldstyle

  ![oldstyle](http://orj2jcr7i.bkt.clouddn.com/leeifme/blog/techoldstyle.png)

- banking

  ![banking](http://orj2jcr7i.bkt.clouddn.com/leeifme/blog/techbanking.png)

**文末，有模版下载地址。**

### 安装MikTex

`LateX`,有很多编译套件，如CTEX，TeXLive，MikTeX等。我推荐MikTeX，才100M+。

官网地址：[MikTex](http://www.miktex.org/)

### 安装LaTeXTools

这是 Sublime 上的一个插件，它可以使用 MikTeX 或者 TeXlive 作为引擎，如果你使用 MikTeX，那么基本上不用设置，因为 LaTeXTools 默认使用的是 MikTeX；

**设置：**

在Sublime中，`Preferences -> Package Settings -> LaTeXTools -> Settings-User`修改为：

```
“texpath” : “D:\Program Files (x86)\MiKTeX 2.9\miktex\bin:$PATH”  //MikTeX安装路径下的bin文件夹
```

## 遇坑必填

### 制作中文简历

模版下载后，编译中文发现报错，需要去掉三行注释

```
\usepackage{CJKutf8}
\begin{CJK*}{UTF8}{gbsn} 
\clearpage\end{CJK*} 
```

完成上述步骤后，你会发现依然报错，但此时，报错为`not find CJKutf8.styl`,

其实，我们在编译时，MikTeX会自动帮我们下载运行需要的组件和脚本，但是我发现它下载CJK组件时会报错，这是中文支持包。

然后我到 MikeTeX Package Manager 里搜索后下载，发现需要`FQ`,所以下载时更改下代理。

完成上述步骤，你就可以开始你的简历制作了

## 彩蛋

- [模版下载地址](https://github.com/leeifme/Moderncv-LateX)
- [LateX在线编辑地址](https://www.sharelatex.com)（不需要外网访问，但是效率较慢，推荐本地编辑）