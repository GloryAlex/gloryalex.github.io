+++
categories = '技巧'
tags = ['cot', 'markdown']
date = '2025-11-27T16:16:34+08:00'
title = '使用 Marked 2 为 CotEditor 增加 markdown 预览功能'
+++

## 问题起源
CotEditor 虽然各种好，但毕竟功能比较轻量，用来编辑 markdown 甚至没有一个预览功能。不过好在它还支持脚本拓展，可以自己动手，丰衣足食。在 Github 上我找到[coteditor_markdown_set](https://github.com/mikejohnduran/coteditor_markdown_set/tree/master)这个仓库里有一系列预览脚本。不过它的预览能力依赖浏览器实现，每次预览都会打开一个浏览器窗口。而且这些脚本非常简单，事实上堪称手搓，很多 Markdown 特性都不支持。

最近突然想起来，我之前还买过 Marked 这个 Markdown 预览专用 app，正好用来解决这个需求。

## 临时预览
不过我使用 CotEditor 通常只在临时编辑，没有一个特定项目的场景。而官方仓库中的示例大部分都是通过`open file.md`的方式来预览的，也就是说必须有一个文件。考虑到只是临时预览，如果专门保存一个临时文件，后面还要定期清理，不太合适。

官方文档中也提到可以直接创建一个临时预览区，并提到了这个 url schema：

```url
x-marked://preview?text=Some%20text%20to%20%2A%2Apreview%2A%2A%0A
```

尝试了一下，这个方案比较适合临时预览。不过由于是通过 URL 发送文本，因此 url encode 是必须的。简单规划了一下，脚本内容分为以下几步：

- 读取所有输入
- url encode 编码
- 拼接成 url，并打开

具体内容如下，也可以直接在[这里](https://gist.github.com/GloryAlex/9c91a1dfd6b8267182fdd0bc6b4415d2)浏览。

```python3
#!/usr/bin/env python3
# %%%{CotEditorXInput=AllText}%%%
# %%%{CotEditorXOutput=Discard}%%%

import sys
import urllib.parse
import webbrowser

content = sys.stdin.read()
content = urllib.parse.quote(content)

webbrowser.open('x-marked://preview?text='+content)
```

然后把脚本文件保存到`~/Library/Application Scripts/com.coteditor.CotEditor`目录下，CotEditor 的脚本菜单下就会自动出现这个命令。

使用这个方案之前，我曾经担心完全通过参数传递文本，在内容过大时会不会卡顿。不过拿一段 1 万多字的文章试了下，基本是秒开。

![效果图](https://i.ibb.co/mCMc112K/11-27-19-12-24-2x.png)

这个方案的问题是没办法实时更新编辑器里的内容，每次预览都会生成一个新的预览窗口。

## 实时预览
如果要实时更新内容的话，得使用`open file.md`的方案。事实上，这也是可以做到的。CotEditor 在新建文件时，会自动保存到 iCloud 中的目录，因此的确存在这样一个**临时文件**可以用于预览。

