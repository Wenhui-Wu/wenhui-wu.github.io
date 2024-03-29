---
layout: post
title: 升级到Monterey后降级回BigSur的笔记
date: 2022-05-02 00:24 +0200
image:
tags:
- 生活
- 杂学
- 黑苹果
---

这篇笔记提供了这些问题的解决思路：
- Opencore更新后版本号没有变化？可以通过重置NVRAM解决；
- Opencore默认启动项被更改，并且选择界面被跳过？可以直接修改EFI中的`config.plist`文件来取消跳过；
- Win和Mac的睡眠相关问题。
 

之前提到过，我有一台XPS15 9550作为黑苹果使用。配置简而言之是：

- 屏幕分辨率是1080P；
- CPU是i7-6700HQ；
- 无线网卡是DW1830。

其他的硬件，同型号的机器应该都一样。

一切的折腾、折磨和心跳，都从iPad突然推送iOS15.4.1更新的那个瞬间开始……
通用控制它来了！
	iPad都花大钱买了，人家辛苦开发的新功能怎么能不试试呢！
于是就开启了漫漫折腾之路。

# OC更新后版本号不变就重置NVRAM

为了避免升级系统之后变砖，更新OC引导和和Kext驱动是必须的。

这一步，我参考了这个视频：[【入门级】黑苹果系统安装OpenCore引导版本升级教程Hackintosh|macOS全系列完整教程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Ga4y1j7eN?spm_id_from=333.880.my_history.page.click)

除了重做SSDT文件之外，都一一按照教程做好了。
但是更新完毕之后遇到了一个教程中没有提到的问题：**Hackintool中的版本号并没有变化**。
一开始我还以为是因为没有重做SSDT文件导致更新失败了，一直在查找SSDT文件相关的资料。
换了一个角度查资料之后决定尝试重置NVRAM，并且证实是有效的。

但是重置NVRAM又引起了另一个问题：OC的默认启动选项被清除，而OC被设置为不显示引导界面，无法进入Mac系统怎么办？

我的方案：想办法直接修改OC的`config.plist`文件。
要实现这个方法，需要有一个能工作的操作系统。
	如果修改过的新OC在U盘里（是所有教程都推崇的安全做法），那么可以直接把U盘插到正常的电脑上进行文件编辑；
	如果新OC在电脑硬盘里，~~那么也可以把硬盘拔下来插到其他电脑上编辑……~~那么可以通过同一台机器上的其他操作系统来完成。

我通过备用的Windows系统进入EFI分区，用记事板修改`config.plist`中的Misc-ShowPicker一项就把booting选单找回来了。
感谢DiskGenius，在给EFI分区分配盘符之后就可以访问了。

一切安排停当，启动OTA升级……

# 更新……成功？

漫长的等待之后终于进入了Monterey，但是问题多多。
	首先是开机时间变得异常长。
		这个问题在后来被发现是因为蓝牙的驱动崩溃了，于是启动过程超时；
	其次，触摸板和蓝牙挂了；
	最后，此外，系统变得不稳定，反复黑屏死机……

其中蓝牙的问题通过参考[这篇文档](https://dortania.github.io/OpenCore-Install-Guide/extras/monterey.html#bluetooth)得到了解决。
简单来说，是因为苹果重写了Montery的蓝牙，所以原先的蓝牙驱动失效了。
只要把原先的`BrcmBluetoothInjector.kext`驱动禁用掉（如果是Intel的网卡，应用的驱动是另一个），并启用`BlueToolFixup.kext`就好了。

蓝牙问题解决之后开机的速度变快了，可能是因为不会再在蓝牙驱动的地方卡住了。
但是新系统的反复崩溃还是让我决定挥手告别，退回到Big Sur。

因为此前没有备份，所以只能全新安装。
抹盘安装的过程参考了这个视频教程：[黑苹果抹盘重装教程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV124411w7zR/)

简单来说，重新安装黑苹果系统比初次安装要简单，只需要制作安装U盘，抹掉原来的系统，再按照引导安装即可。

# 回滚成功……装机推荐？

回滚成功，尘埃落定。
重新装机，记录的同时顺便安利一下软件
。
首先是xzhih的[one-key-hidpi](https://github.com/xzhih/one-key-hidpi)。
	它的操作非常简单，可以启用HiDPI缩放功能，让1080p屏幕上的字体再也不发虚；
其次是在`系统偏好设置 - 共享 - 文件共享`中启动samba服务器。
	这样就可以在我的主力Win电脑上随意管理Mac上的文件，还可以在两个系统之间快速传输文件了。
	顺便一提，我的Obsidian仓库是通过iCloud同步的，而Windows设备访问Obsidian就是以SMB为桥梁的。

使用一段时间过后，我的黑苹果似乎“睡得不好”。
	就是说睡眠功能并不是百分百在正常工作。
有的时候会在盒盖时神秘唤醒，有的时候合上盖子之后风扇还是转个不停。
于是我参考了一篇[教程博文](https://blog.skk.moe/post/thinkpad-e480-hackintosh/)，以及一篇[少数派上讲原理的文章](https://sspai.com/post/61379)。
这两篇文章谜之解决了我的神秘唤醒问题。
并且在后来，我发现合盖后风扇不停的原因是Win上的Obsidian会在Win睡眠之后依然保持与Mac的会话，阻止Mac进入睡眠。
	这个问题只需要在让Win进入睡眠之前随手关掉Obsidian就好。
	不过在解决Win神秘唤醒问题之后（参考了[V2EX的一篇帖子](https://v2ex.com/t/601966)），似乎Obsidian保持连接的问题就自己消失了？

结果好一切都好。
偶尔折腾一下，发现问题，解决问题，感觉挺好的。