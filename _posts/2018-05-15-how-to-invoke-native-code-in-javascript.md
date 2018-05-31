---
layout:       post
title:        "如何在UWP环境下WPF控件Webview中使用Javascript调用类库接口"
subtitle:     "How to Invoke Native Code in WPF webview with Javascript"
date:         2018-05-15 21:43:11
author:       "Devdog"
header-img:   "img/post-bg-rwd.jpg"
header-mask:  0.3
catalog:      windows
multilingual: false
tags:
    - UWP
    - JavaScript
    - webview
---

>需求

最近我需要将蓝牙iBeacon上的数据获取出来，然后将对应的JavaScript所需接口注入到我们应用内的浏览器里，便于合作方在webview页面上调用。Javascript的接口具体细节参考微信上摇一摇业务开发的[H5页面获取设备信息 JS API](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1443448133)。

>实现方式

在Javascript中调用平台接口，一般可以采用COM或ActiveX组件的方式注册到系统中然后在Javascript中调用或者标准接口直接使用HTML5来实现。但我发现在Visual Studio 2017中UWP平台下没有看到生成COM或ActiveX组件的方式，参考了MSDN上这篇[How to invoke Javascript (JS) at Native, and invoke native at JS in WebView of Universal Windows Platform (UWP)](https://code.msdn.microsoft.com/windowsapps/How-to-invoke-JS-at-Native-df3fd459),我们了解到需要先创建一个RuntimeComponent（universal windows）项目，生成.winmd文件，然后在UWP（universal windows）项目中引用这个.winmd文件，添加一个webview控件，然后通过AddWebAllowedObject绑定RuntimeComponent中提供的方法注入到webview中，供javascript调用。

>开发环境：

- Windows 10
- Visual Studio 2017
- C#

>具体步骤


**创建RuntimeComponent项目**

在Visual Studio 2017中新建项目，选择“Windows运行时组件(通用Windows)”，用于为通用Windows平台UWP应用创建托管Windows运行时组件.winmd。
![RuntimeComponet](/img/in-post/20180515/createruntimecomponent.png)


**AllowForWeb**

项目中类SHRFIDBeaconService必须为sealed，并且必须要引用[Windows.Foundation.Metadata.AllowForWeb]中的[AllowForWeb]属性。定义方法GetPlusResult和GetVersion。

<pre><code>
//This project type is RuntimeComponent(Universal Windows)
//Three times for important thing,^_^
namespace SHRFIDBeaconRuntimeComponent
{
    //this class must be sealed, and must have attribuite [Windows.Foundation.Metadata.AllowForWeb]
    [AllowForWeb]
    public sealed class SHRFIDBeaconService
    {
        private static Logger logger = LogManager.GetLogger("SHRFIDBeaconService");

        private static bool _restartingBeaconWatch;
        private static string _errBleMessage;        

        private readonly WindowsBluetoothPacketProvider _provider;
        private BeaconManager _beaconManager;

        public SHRFIDBeaconService()
        {
            // Construct the Universal Bluetooth Beacon manager
            _provider = new WindowsBluetoothPacketProvider();
            _beaconManager = new BeaconManager(_provider);

            // Subscribe to status change events of the provider
            _provider.WatcherStopped += WatcherOnStopped;
            _beaconManager.BeaconAdded += BeaconManagerOnBeaconAdded;
            //
	......
        public int GetPlusResult(int param1, int param2)
        {
            return param1 + param2;
        }

        public string GetVersion()
        {
            return "1.0.0.0";
        }
	......
</code></pre>

**创建UWP项目**

创建空白UWP项目

![CreateUWP](/img/in-post/20180515/createemptyuwp.png)

**WPF中webveiw**

空白页面中添加webview，source设置为测试页面



        <!-- Source file is on the web.         >>>> http://www.microsoft.com-->
        <!-- Source file is in local storage.   >>>> ms-appdata:///local/intro/welcome.html-->
        <!-- Source file is in the app package. >>>> ms-appx-web:///InvokeNativeCode/InvokeNativeCode.html-->
        <!-- WebView Grid.Row="1" Name="MainWebView" NavigationStarting="MainWebView_NavigationStarting" Source="http://www.microsoft.com" /-->
        <WebView Grid.Row="1" Name="MainWebView" NavigationStarting="MainWebView_NavigationStarting" Source="ms-appx-web:///InvokeNativeCode/InvokeNativeCode.html" />

**WPF页面实现MainWebView_NavigationStarting方法**

WPF页面，添加MainWebView_NavigationStarting实现，使用AddWebAllowedObject在页面载入时将Runtimecomponent中的服务SHRFIDBeaconService注入到webview中。

<pre><code>
	namespace SHRFIDBeaconServiceTest
	{

    public sealed partial class MainPage : Page
    {
        public MainPage()
        {
            this.InitializeComponent();
        }

        //when start Navigation, Add Native object to webview,and then the object can be invoke in html.
        private void MainWebView_NavigationStarting(WebView sender, WebViewNavigationStartingEventArgs args)
        {
            //This way can invoke native method and get result from native method.
            //But because this native is from orther project in this solution, so when you operator UI, it will be very difficult.
            sender.AddWebAllowedObject("SHRFIDBeaconService", new SHRFIDBeaconRuntimeComponent.SHRFIDBeaconService());
        }
</code></pre>

		

**HTML测试页面**

测试页面中引用SHRFIDBeaconService各种方法实现，例如SHRFIDBeaconService.GetPlusResult(),SHRFIDBeaconService.GetVersion()等。

	<!DOCTYPE html>
	<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
	<head>
    <meta charset="utf-8" />
    <title>SHRFIDBeaconServiceTest</title>
	</head>
	<body style="background:#007acc">
    <h1>Invoke native code in WebView</h1>
    <h2>Blue region is WebView</h2>
    <h3 id="ifName"></h3>
    <h3 id="resultContainer"></h3>
    <br />
    <br />
    <br />
    <div>
        <button id="btn1" width="200" onclick="GetPlusResult()">GetPlusResult</button>
        <button id="btn2" onclick="GetVersion()">GetVersion</button>
    </div>
    <br />
    <div>
        <button id="btn4" onclick="StartSearchBeacons()">StartSearchBeacons</button>
        <button id="btn6" onclick="OnSearchBeacons()">OnSearchBeacons</button>
        <button id="btn5" onclick="StopSearchBeacons()">StopSearchBeacons</button>
    </div>
    <br />
    <div>

    </div>

    <script>
        window.onload = function () {
            //Notice that The Native method name is GetPlusResult, but in js it's name is getPlusResult, the first letter had to to lower.
            var a = 117;
            var b = 140;
            var result = SHRFIDBeaconService.getPlusResult(a, b);
            document.getElementById("ifName").innerHTML = "getPlusResult: ";
            document.getElementById("resultContainer").innerHTML =  a + "+" + b + "= " + result;
        }

        function GetPlusResult() {
            var a = 123;
            var b = 234;
            var sum = SHRFIDBeaconService.getPlusResult(a, b);
            document.getElementById("ifName").innerHTML = "getPlusResult: ";
            document.getElementById("resultContainer").innerHTML =  a + "+" + b + "= " + sum;
        }
        function GetVersion() {
            var version = SHRFIDBeaconService.getVersion();
            document.getElementById("ifName").innerHTML = "getVersion: ";
            document.getElementById("resultContainer").innerHTML =  version;
        }
        function StartSearchBeacons() {
            var startb = SHRFIDBeaconService.startSearchBeacons();
            document.getElementById("ifName").innerHTML = "startSearchBeacons: ";
            document.getElementById("resultContainer").innerHTML = startb;
        }
        function StopSearchBeacons() {
            var stopb = SHRFIDBeaconService.stopSearchBeacons();
            document.getElementById("ifName").innerHTML = "stopSearchBeacons: ";
            document.getElementById("resultContainer").innerHTML =  stopb;
        }
        function OnSearchBeacons() {
            var searchb = SHRFIDBeaconService.onSearchBeacons();
            document.getElementById("ifName").innerHTML = "onSearchBeacons: ";
            document.getElementById("resultContainer").innerHTML = searchb;
        }
    </script>
	</body>
	</html>

**实际测试**

生成，部署，启动......

![test](/img/in-post/20180515/test.png)
