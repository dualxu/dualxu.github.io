---
layout:       post
title:        "麦步手表上的terminal"
subtitle:     "麦步论坛活动"
date:         2017-04-12 22:05:32
author:       "Devdog"
header-img:   "/img/in-post/20170302-arduino-maibu2.jpg"
header-mask:  0.3
catalog:      windows
multilingual: false
tags:
    - 麦步
    - Arduino
---


在麦步手表上实现的一个类似Ubuntu上terminal的表盘界面。最开始有个系统bootloader和os的版本号，然后是杜撰的一个si命令用于获取系统信息（system information）。这个表盘界面上获取的麦步用户名作为登陆用户，其他信息还有日期，时间，农历信息，步数和楼层，海拔和气压，温度等。最后再回到命令提示符。
麦步论坛上看[[表盘发布] 装逼范ubuntu表盘[附件已可下载，黑白两色背景]](http://bbs.maibu.cc/forum.php?mod=viewthread&tid=1382&highlight=ubuntu)。
---
做个ubuntu界面的表盘，还只是模拟的界面其中si是杜撰的一个命令，获取system information（si），然后是该命令给出的一些应答数据，最后回到提示符界面。第一行是标题，第二行是bootloader和OS的版本，第三行是命令提示符及命令si，下面依次是日期时间，农历，步数和楼层，电量和温度，最后是海拔高度和气压。最后一行提示符界面的那个光标可是会闪烁等待你输入命令的哦
随便看看，很好的学习过程

