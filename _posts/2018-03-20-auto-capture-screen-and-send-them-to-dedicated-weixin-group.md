---
layout:       post
title:        "利用AutoIt脚本自动发送浏览器页面截屏图片到微信群"
subtitle:     "AutoIt脚本的威力"
date:         2018-03-20 19:55:37
author:       "Devdog"
header-img:   "img/post-bg-geek.jpg"
header-mask:  0.3
catalog:      windows
multilingual: false
tags:
    - AutoIt
---

我固定每个周日或周一上午都要将上一周设备上汇总数据截屏发送到微信群,以便领导了解情况。已经连续手动截屏并发送到微信群快一年了，虽然一共只有3张页面而已，但我还是想可不可以通过AutoIt脚本自动完成呢？我此前曾经用AutoIt做了一些相关的桌面小工具，但没有试过操作浏览器页面，截取屏幕和向微信群发送消息。以下是我完成过程，来个小总结吧。

**AutoIt，要不要了解下！** 

以下是百度百科上对[AutoIt](https://baike.baidu.com/item/autoit/4327423)的介绍。

>AutoIt 目前最新是v3版本，这是一个使用类似BASIC脚本语言的免费软件,它设计用于Windows GUI(图形用户界面)中进行自动化操作。它利用模拟键盘按键，鼠标移动和窗口/控件的组合来实现自动化任务。而这是其它语言不可能做到或无可靠方法实现的(例如VBScript和SendKeys).

>*功能*
>
- 运行Windows和Dos程序 
- 模拟键击动作(支持大多数键盘布局)
- 模拟鼠标移动和点击动作
- 对窗口进行移动,调整大小和其它操作
- 直接与窗口的“控件“交互(设置/获取文本,移动,关闭等等)
- 配合剪贴板进行剪切/粘贴文本操作
- 对注册表进行操作

>不同于AutoIt v2版本,新的v3版本有更多标准语法-类似于VBScript和BASIC-而且支持更复杂的表达式,用户函数,循环以及脚本编写老手们所期待的其它所有内容.
正如以前版本一样,AutoIt设计得尽可能的小(大约115KB)并且不用依赖外部DLL文件或添加注册表项目即可独立运行.此外使用 Aut2Exe 这个工具还可以把脚本文件编译为独立的可执行程序.
同时升级了ActiveX和DLL版本在AutoIt里称为 AutoItX - 与v2版本不同的是它将是一个组合控件 (COM组件对象模型和同一DLL文件中的标准DLL函数).AutoItX 将允许您加入一些AutoIt独有的特性到您最常用的脚本语言或程序设计语言中去!请查看这AutoItX帮助文件 (开始 \ 程序\ AutoIt v3 \ Extras \ AutoItX \ AutoItX Help File) 以获得更多信息和示例.
最重要的是,AutoIt 将继续是免费的 - 但是如果您打算支持我们花在此工程的时间,金钱以及所作努力和网站主机运作的话,那么您可以到AutoIt的主页上进行捐赠.

**打开浏览器**

先引用IE.au3，使用RunWait打开一个URL，即调用系统默认浏览器打开网站页面。
<pre><code>
#include "IE.au3"
#include "AutoItConstants.au3"

;打开默认浏览器-
RunWait('"' & @ComSpec & '" /c explorer.exe http://124.72.48.52:8080/mainpage', '', @SW_HIDE)
</code></pre>

**删除图片文件**

删除指定保存图片文件夹下的图片，以后截屏图书也是保存在此目录。
<pre><code>
;删除Images目录下3个文件
$myFile = "D:\WorkSpace\...\Images\ImageCapture1.jpg"
If FileExists($myFile) Then
    FileDelete($myFile)
EndIf
Sleep(2000)

$myFile = "D:\WorkSpace\...\Images\ImageCapture2.jpg"
If FileExists($myFile) Then
    FileDelete($myFile)
EndIf
Sleep(2000)

$myFile = "D:\WorkSpace\...\Images\ImageCapture3.jpg"
If FileExists($myFile) Then
    FileDelete($myFile)
EndIf
Sleep(2000)
</code></pre>

**移动鼠标和点击**

用MouseMove移动鼠标至屏幕指定位置，比如某个按钮，文字框等等页面控件，然后通过MouseClick进行点击动作。
<pre><code>
;
MouseMove(940,122+100,3);
MouseClick($MOUSE_CLICK_LEFT);
Sleep(1000);
;
MouseMove(1123,122+100,3);
MouseClick($MOUSE_CLICK_LEFT);
Sleep(1000);
;
MouseMove(1251,135+100,3);
MouseClick($MOUSE_CLICK_LEFT);
Sleep(3000);
</code></pre>

**截屏**

对屏幕操作，截取屏幕，引用ScreenCapture.au3，使用对指定矩形位置截屏并保存图片文件

<pre><code>
#include "ScreenCapture.au3"

_ScreenCapture_Capture("D:\WorkSpace\...\Images\ImageCapture1.jpg",344,100+100,1354,624+100);
</code></pre>

**打开文件夹并拷贝图片文件**

和打开浏览器一样，还是使用RunWait打开一个路径，直接打开文件夹。在文件夹中点击，然后通过Send命令模拟送出全选(CTRL+A)和拷贝(CTRL+C)快捷键，拷贝至系统剪贴板。
<pre><code>
;打开Imges文件夹，选中并拷贝3个图片文件
RunWait('"' & @ComSpec & '" /c explorer.exe D:\WorkSpace\...\Images', '', @SW_HIDE)
Sleep(1000);
$handle = WinGetHandle("Images","");
WinActivate($handle);
Sleep(1000);
$pos = WinGetPos($handle);
;
MouseMove($pos[0]+657,$pos[1]+556,3);
MouseClick("");
Sleep(1000);
;
Send("^a")
Sleep(1000);
;
Send("^c")
Sleep(1000);
</code></pre>

**打开微信群**

微信已经打开，WinActivate等待微信窗口激活，通过WinGetPos获取该窗口的位置，然后鼠标定位到搜索框，Send送出要搜索的微信群名称，然后点击第一个搜索结果的位置，即打开该微信群。鼠标定位到该微信群聊天窗口，点击。
<pre><code>
;微信上搜索并点开相应微信群
$handle = WinGetHandle("[CLASS:WeChatMainWndForPC]", "");
WinActivate($handle);
Sleep(1000);
$pos = WinGetPos($handle);
MouseMove($pos[0]+100,$pos[1]+30,3);
MouseClick("");
Sleep(1000)
;
Send("微信群名称填写在这里");
Sleep(1000);
MouseMove($pos[0]+160,$pos[1]+100,3);
MouseClick("");
Sleep(1000);
;
MouseMove($pos[0]+640,$pos[1]+520,3);
MouseClick("");
;
Sleep(1000);
</code></pre>

**粘贴图片文件到聊天窗口，发送**

通过Send命令模拟送出粘贴(CTRL+V)快捷键，将前序系统剪贴板上的三个图片文件粘贴到微信群聊天窗口，然后送出Enter键，发送以上图片文件。
<pre><code>
Send("^v");
Sleep(1000)
;点发送，相当于键盘按Enter;
Send("{Enter}");
Sleep(3000)
</code></pre>

**发送文字**

和发送图片一样，发送文字到微信群聊天窗口，然后通过Enter键发送。

<pre><code>
Send("以上是XX市图试运行XX柜和XX柜上一周借阅、上下架及当前在架图书统计。{Enter}");
</code></pre>

**大功告成**

来个图片看看吧

![AutoIt_WechatGroup](/img/in-post/20180320/autoitwechatgroup.png)
