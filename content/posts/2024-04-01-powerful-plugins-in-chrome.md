---
title: "如何像使用终端一样使用浏览器？"
date: 2024-04-01T16:57:45+08:00
draft: false
comment: true
description: "今天写一篇小短文，推荐两个 Chrome 插件，用于程序员们提高浏览器的操作效率，像使用终端一样使用浏览器。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-01-awesome-plugins-in-browser-01.png)

今天写一篇小短文，推荐两个 Chrome 插件，用于程序员们提高浏览器的操作效率，让我能像使用终端一样使用浏览器。

## CTRL+N/P

一个是 CTRL+N/P，如果你习惯于终端上的上下翻页历史的记录的操作，这个工具挺适合你的。它的作用也非常简单，但你在搜索引擎上搜索记录时，如果遇到刻在骨子里的情不自禁使用 CTRL+P/N  上下翻页搜索相关条目是，它就是你需要的。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-01-awesome-plugins-in-browser-04.png)

默认情况下，Chrome 浏览器顶部搜索框是支持这个操作的。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-01-awesome-plugins-in-browser-02.gif)

但如果进入搜索页面，这个搜索框没有这个能力。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-01-awesome-plugins-in-browser-03.png)

于是我就尝试安装了这个 Ctrl+N/P 插件。

我在 Google、baidu、bing 和 sogou 进行了测试，结果除了 baidu 其他的都可以正常使用。这对我没有影响，毕竟我基本不用 baidu。

## Vimium

另一个插件是 Vimium，适合于 Vim 用户的 Chrome 插件，浏览器上的 vim 模拟器。如果你习惯于日常使用 Vim, 然而发现和浏览器交互时, 就必须要使用鼠标或者触控板，可以考虑使用它。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-01-awesome-plugins-in-browser-05.png)

它的使用不复杂。默认情况下，它是在 Vim 的 Normal 模式，如果要想把浏览器恢复到和没有安装 Vimium 一样的效果，只要输入 i 到插入模式即可。在 

Normal 模式下的大部分操作都是 Vim 风格的。输入 `?` 就可以打开它的帮助文档。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-01-awesome-plugins-in-browser-06.png)

我最喜欢的还是 Vim 的导航能力，这也是平时用的最多的。阅读文档或者阅读测试代码时，直接 jk 实现上下滚动，与双屏配合，大部分情况下无需用到鼠标。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-01-awesome-plugins-in-browser-07.gif)

如果想快速点击或访问链接快速，输入 f，它会把所有可点击标识出来。我测试过，它也不能把所有的点都能标识出来，有它的局限性。

具体效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-01-awesome-plugins-in-browser-08.gif)

Vimium 已经很大程度上提高了键盘操作浏览器的效率，它还有更多能力，在它的帮助中都能找到，输入 `?` 可自行探索吧。

## 最后

我觉得也不要过分迷恋这种操作，毕竟它无法实现完全的全键盘操作，GUI 毕竟和字符界面是不一样的，找到适合自己操作方式。

但其实我挺迷恋的，毕竟不迷恋就不会研究这操作。

## 附上帮助翻译

快捷键      | 功能
----------- | -------
j, \<c-e>	  | 向下滚动
k, \<c-y>   |	向上滚动
gg          | 滚动到页面顶部
G	          | 滚动到页面底部
d	          | 向下滚动半页
u	          | 向上滚动半页
h	          | 向左滚动
l	          | 向右滚动
zH	        | 全部向左滚动
zL	        | 全部向右滚动
r	          | 重新加载页面
yy	        | 复制当前URL到剪贴板
p	          | 在当前标签页中打开剪贴板的URL
P	          | 在新标签页中打开剪贴板的URL
gu	        | 上升到URL层级结构
gU	        | 转到当前URL层级结构的根
i	          | 进入插入模式
v	          | 进入视觉模式
V	          | 进入视觉行模式
gi	        | 聚焦页面上的第一个文本输入框
f	          | 在当前标签页中打开一个链接
F	          | 在新标签页中打开一个链接
\<a-f>	    | 在新标签页中打开多个链接
yf	        | 复制链接URL到剪贴板
[[	        | 跟随标记为previous或<的链接
]]	        | 跟随标记为next或>的链接
gf	        | 选择页面上的下一个框架
gF	        | 选择页面的主要/顶部框架
m	          | 创建一个新的标记
`	          | 跳转到一个标记
o	          | 打开URL、书签或历史条目
O	          | 在新标签页中打开URL、书签或历史条目
b	          | 打开一个书签
B	          | 在新标签页中打开一个书签
T	          | 搜索你打开的标签页
ge	        | 编辑当前的URL
gE	        | 编辑当前的URL并在新标签页中打开
/	          | 进入查找模式
n	          | 向前循环到下一个查找匹配
N	          | 向后循环到上一个查找匹配
H	          | 后退历史
L	          | 前进历史
t	          | 创建新标签页
J, gT	      | 向左移动一个标签页
K, gt	      | 向右移动一个标签页
^	          | 跳转到之前访问的标签页
g0	        | 跳转到第一个标签页
g$	        | 跳转到最后一个标签页
yt	        | 复制当前标签页
\<a-p>	    | 固定或取消固定当前标签页
\<a-m>	    | 静音或取消静音当前标签页
x	          | 关闭当前标签页
X	          | 恢复关闭的标签页
W	          | 将标签页移动到新窗口
\<<	        | 将标签页向左移动
\>>	        | 将标签页向右移动
?	          | 显示帮助
gs	        | 查看页面源代码

如有错误，请指正。感谢！
