---
title: macOS逆向初学者的第一篇文章
date: 2023-12-14 15:14:43
tags:
  - macOS逆向
  - 软件破解
---

刚入门 macOS 逆向的一位萌新，借一个简单的应用记录一下成果。

首先准备个两个软件，ida pro 和 hopper（demo 版即可），用来做静态分析。应该具备一些基本 C 语言知识，我的电脑是 m1，所以文章内展示的都是 ARM 系列的，之前的 intel 版本就选择 X86_64

今天的目标是 Macs Fan Control 用这个软件来做个例子，因为足够简单。

## 1. 初步准备

在应用程序里面右键软件包 Macs Fan Control，显示包内容，可以看到这个程序的内部文件，我们的目光注意在 MacOS 这个文件夹内的和软件同名的二进制文件，这个文件就是主程序。我们把他复制一份到其他地方，下载或文档等其他目录里面。

![Alt text](image.png)

将复制出来的二进制文件拉到 ida 和 hopper 中，选择自己的系统架构。先用 hopper 加载，因为他的字符串搜索功能比较好用（个人观点）。

![Alt text](image-1.png)

## 2. 开始分析

我们打开目标软件看一下关于里面的信息

![Alt text](image-2.png)

可以看到有一个 Free version 的字样，我们在 hopper 中搜索一下，切换到 Str 这个地方搜索，这个是搜索字符串。

![Alt text](image-3.png)

我们可以看到搜索 Free 的时候，第一个出来的结果是 `AboutDialog/staticFreeVersion` 对应的就是关于窗体的免费版本，红框的地方我们可以看到还有一个 pro 版本，我们按下 x 跳转过去看看。

![Alt text](image-4.png)

![Alt text](image-5.png)

伪代码里面我们随便翻看一下，可以看到需要的 Qxxxxx 的字样，应该是一个 Qt 程序哈，伪代码我更倾向于用 ida 查看，这俩软件相互配合使用。

![Alt text](image-6.png)

ida 里面的伪代码我们可以看到，ProVersion 这个块是 LABEL72，这个块里面，第一行是一个判断 `if ( !(unsigned int)sub_100028588(QCoreApplication::self) )` 我们进到这个函数里面看一下。

![Alt text](image-7.png)

可以看到函数里面有一个 pro-version 到字样，或许是关键点，并且这个函数返回一个 int 类型，结合判断条件 `if ( !(unsigned int)sub_100028588(QCoreApplication::self) )` 来说，如果返回值是 0，取反条件成立，就执行下面的 `goto LABEL_91` 我们不让他跳走，让他继续往下走到 ProVersion 这个地方。就把这个函数返回值写死成 1，这样就可以了。

## 3. 开始修改程序

![Alt text](image-8.png)

这是我们改完之后的样子，先到函数第一行，按 tab 切换到汇编界面，然后按下 <kbd>control</kbd>+<kbd>option</kbd>+<kbd>k</kbd>，跳出修改窗口

![Alt text](image-9.png)

第一行改成 `mov x0, #1` 第二行改成 `ret`

然后保存我们的修改

![Alt text](image-10.png)

看到输出窗口有提示信息 Applied 8/8 patch(es) 说明就已经改完了，我们把修改后的文件，也就是最开始复制出来的那个文件，替换软件包 Macs Fan Control 里面的 MacOS 下的同名文件，然后签名。

```bash
sudo codesign -f -s - --timestamp=none /Applications/Macs\ Fan\ Control.app
```

签名后打开软件在看下。

![Alt text](image-11.png)

可以看到已经成功成为 pro 版了，至此，软件破解完成。

参考文章： https://www.52pojie.cn/forum.php?mod=viewthread&tid=1705732&extra=page%3D1%26filter%3Dtypeid%26typeid%3D377
