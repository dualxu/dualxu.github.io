---
layout:       post
title:        "如何在C#类库中调用OCX控件及解决InvalidActiveXStateException异常"
subtitle:     "解决调用时InvalidActiveXStateException异常"
date:         2018-05-25 11:51:42
author:       "Devdog"
header-img:   "img/post-bg-geek.jpg"
header-mask:  0.3
catalog:      windows
multilingual: false
tags:
    - C#
    - OCX
---


关于在C#中调用OCX控件这里有个很好的例子[C#调用OCX控件的常用方法](http://developer.huawei.com/ict/forum/thread-21687.html),结合最近我实际开发中碰到的问题，我总结下并具体描述下我这里的解决方式。

开发环境：Windows 10，Visual Studio 2012，.Net Framework 4.5

>注册OCX控件

将要用的.ocx文件拷贝到Visual Studio 2012安装目录下的VC文件夹，例如我的是：D:\Applications\Microsoft Visual Studio 11.0\VC，使用regsvr32.exe工具进行注册(需要管理员权限，即在管理员权限下打开cmd.exe)

![RegisterOCX](/img/in-post/20180525/registerocx.png)


>生成AxDll

打开Visual Studio 2012 命令提示，例如我的在：C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Microsoft Visual Studio 2012\Visual Studio Tools\目录下，使用aximp.exe实用工具生成2个dll文件，其中一个是Ax开头的dll文件，在后续流程中使用此dll工具引用到类库中。

![AxImp](/img/in-post/20180525/aximp.png)

>添加引用

在Visual Studio 2012中创建C#类库项目，添加上一步生成的Ax开头的dll引用。

![AddAxDll](/img/in-post/20180525/addaxdll.png)

在类库中调用Dll中的方法。
<pre><code>
    public partial class TiCaiCut
    {
        AxTiCaiCut _tcc = null;
        /// <summary>
        /// 构造函数
        /// </summary>
        public TiCaiCut()
        {
            _tcc = new AxTiCaiCut();         
        }

        /// <summary>
        /// 打开设备
        /// </summary>
        /// <param name="sPort">端口号</param>
        /// <param name="iBPS">波特率，9600</param>
        /// <returns>返回：0-成功，其他失败</returns>
        public short Open(string sPort, short iBPS)
        {
            return _tcc.Open(sPort, iBPS);
        }
	......
</code></pre>

>调用类库

在上一个解决方案中添加一个Console测试程序，引用以上生成的类库中的方法,例如以上的Open。

<pre><code>
            TiCaiCut tcc = new TiCaiCut();
            //
            label_menu:
            Console.WriteLine("");
            Console.WriteLine("----------------------------------");
            Console.WriteLine("1.打开设备");
            ......
            Console.WriteLine("0.退出");
            Console.WriteLine("----------------------------------");

            key = Console.ReadLine();
            switch (key)
            {
                case "1":
                    Console.WriteLine("串口号:");
                    string StrCom = Console.ReadLine();
                    Console.WriteLine("波特率(9600):");
                    string StrBaud = Console.ReadLine();
                    short iBaud = 9600;
                    Int16.TryParse(StrBaud,out iBaud);
                    short RetOpen = tcc.Open(StrCom, iBaud);
                    Console.WriteLine("打开设备："+ RetOpen.ToString());
                    break;
                case "2":
			......
</code></pre>

>异常问题解决

好了，运行。这个时候可能会看到一个异常。如下图：

![InvalidActiveXException](/img/in-post/20180525/invalidativexexception.png)

这个问题在[C#中引用第三方ocx控件引发的问题以及解决办法](https://blog.csdn.net/minjunyu/article/details/5627908)中已经给出了Winform中的解决方法，但并没有提及到类库中的解决方法。

受这个帖子[C# 在类文件中调用ocx控件,引发System.Windows.Forms.AxHost异常，求助](https://bbs.csdn.net/topics/370061237)的启发， 在类库中构造函数初始化时添加CreateControl方法，强制创建可见控件，此问题得到解决。

<pre><code>
    public partial class TiCaiCut
    {
        AxTiCaiCut _tcc = null;
        /// <summary>
        /// 构造函数
        /// </summary>
        public TiCaiCut()
        {
            _tcc = new AxTiCaiCut();
            //
            _tcc.BeginInit();
            _tcc.CreateControl();
            _tcc.EndInit();
        }

        /// <summary>
        /// 打开设备
        /// </summary>
        /// <param name="sPort">端口号</param>
        /// <param name="iBPS">波特率，9600</param>
        /// <returns>返回：0-成功，其他失败</returns>
        public short Open(string sPort, short iBPS)
        {
            return _tcc.Open(sPort, iBPS);
        }
	......
</code></pre>

希望可以有所帮助！