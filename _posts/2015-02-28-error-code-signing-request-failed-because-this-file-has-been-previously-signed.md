---
layout: post
title:  "Error:Code signing request failed because this file has been previously signed"
date:   2015-02-28 19:24:43 +0800
categories: windows
permalink: /windows/2015/02/28/error-code-signing-request-failed-because-this-file-has-been-previously-signed.html
---



今天更新PTCInquiry，在Momentics IDE环境下更改bar-descriptor.xml中Localization增加了中文和英文，然后就出现“Code signing request failed because this file has been previously signed”的错误了。原因是因为这个版本已经签名过了，不能再次签名，需要升级版本。提示中给出解决方法了，需要更新Package Version或Package Build ID， 支持论坛上也是这样说的，参见：?0?2http://supportforums.blackberry.com/t5/Testing-and-Deployment/Code-signing-request-failed-because-this-file-has-been/ta-p/798291

但我这边更新Package Version或Package Build ID还是出现同样的错误，原来版本之后的buildnum每次build之后都是自动递增的，可以肯定的是修改了Localization前后不知道动了什么地方破坏了bar-descriptor.xml文件。 我后来的解决办法是在上次备份里面的bar-descriptor.xml拿出来，更改bar-descriptor.xml中Localization增加了中文和英文就OK了。