---
layout:       post
title:        "使用Windows服务自动发送工作周报"
subtitle:     "VBA从模板生成每周周报"
date:         2018-02-02 12:35:54
author:       "Devdog"
header-img:   "img/in-post/post-bg/post-bg-random02.jpg"
header-mask:  0.3
catalog:      windows
multilingual: false
tags:
    - VBA
    - Windows服务
---

**每周都要写周报！**

**每周都要定时发送周报！**

因此我决定写个自动发送周报的小工具，自动发送周报，减轻这个上面花费的时间。

### 开发环境
- Excel 2013
- Visual Studio 2012

### 实现方式

- 使用VBA在Excel中定义一个方法，每次周报excel文件打开时都检测下是否以当前周五日期命名的sheet页面，如果没有则从周报模板sheet上拷贝一个新sheet，并以当前周五日期命名。每天工作完毕或周五前都可以填写该周报上工作内容。
- 使用Visual Studio 2012 创建一个windows服务，定时将以上周报excel文件以附件形式通过邮件发送到指定电子邮箱。

### VBA之于周报Excel

周报Excel中先生成一个周报模板sheet，作为Sheet(1)，并另存为“Excel启用宏的工作簿(*.xlsm)”。打开周报Excel，选择“开发工具”菜单（这个菜单默认不打开，可以在文件-高级-自定义功能区-主选项卡里面添加进来）下”查看代码“，选择“ThisWorkBook”，添加Workbook_Open方法，这个将在Excel文件打开时执行。

![WeekReport](/img/in-post/20180202/week-report-vba.png)

<pre><code>

Private Sub Workbook_Open()
Dim shname As String
Dim s As Date
Dim wd As Integer
Dim isFound As Boolean

isFound = False

'获取当前日期周五的日期
wd = weekday(Date, vbSunday)
s = Date - (wd - 1) + 5
shname = Format(s, "yyyymmdd")

'检查是否已有该sheet
For i = 1 To Sheets.Count
    If Sheets(i).Name = shname Then
        isFound = True
        Exit For
    End If
Next

If isFound = False Then

    ActiveWorkbook.Sheets("Template").Copy before:=Worksheets(1)  '新建的sheet在第一个前,总是在第一个
    'sheet name
    ActiveSheet.Name = shname
    
    MsgBox "已创建新一周周报，表名称为" & shname & ", 请开始键入新一周工作内容吧!", vbOKOnly, "提示"
    
End If

End Sub
</code></pre>

###Windows Service

在Visual Studio中创建Windows Service，参考:[VS2012下开发Windows服务](https://blog.csdn.net/huangcailian/article/details/42237013)。创建系统时钟，设定时间间隔为10分钟，即每10分钟检查一次时间是否满足条件，满足的话执行发送电子邮件方法。

<pre><code>
public MainService()
        {
            InitializeComponent();
            timer1 = new System.Timers.Timer();
            timer1.Elapsed += new System.Timers.ElapsedEventHandler(this.timer1_Elapsed);
        }

        protected override void OnStart(string[] args)
        {
            Load_Config();
            //
            timer1.Enabled = true;
            timer1.Interval = 1000 * 60 * 10; //10分钟间隔
            timer1.Start();
            logger.Info("SendWeekRptAutoService Service Start!");
        }

        protected override void OnStop()
        {
            timer1.Enabled = false;
            timer1.Stop();
            logger.Info("SendWeekRptAutoService Service Stop!");
        }
</code></pre>

###Timer

检查时间是否满足，我设定的是周五17:00以后到周日凌晨。时间满足的话则发送邮件。还设置了一个发送标记，这个发送标记将更新到本地配置文件，如果已经发送则不会再次发送邮件。
<pre><code>
private void timer1_Elapsed(object sender, System.Timers.ElapsedEventArgs e)
        {

            DateTime dt = DateTime.Now;
            if (((dt.DayOfWeek != DayOfWeek.Friday) || (dt.Hour < 17)) && (dt.DayOfWeek != DayOfWeek.Saturday) && (dt.DayOfWeek != DayOfWeek.Sunday))
            {
                if (isSent == true)
                {
                    isSent = false;
                    UpdateSentFlag(isSent);
                }
                logger.Info("SendWeekRptAutoService-->timer1_Tick 1");
            }
            else
            {
                if (isSent == false)
                {
                    if (sendmail() == true)
                    {
                        isSent = true;
                        UpdateSentFlag(isSent);
                        logger.Info("SendWeekRptAutoService-->timer1_Tick 2");
                    }
                    else
                    {
                        logger.Info("SendWeekRptAutoService-->timer1_Tick 3");
                    }
                }
                else
                {
                    logger.Info("SendWeekRptAutoService-->timer1_Tick 4");
                }
            }
        }
</code></pre>

###发送邮件

<pre><code>
private bool sendmail()
        {
            //logger.Info("SendWeekRptAutoService-->sendmail");

            string host = "smtp.qq.com";// 邮件服务器smtp.qq.com表示腾讯邮箱服务器    
            string userName = "hxjjju@yourdomain.com";// 发送端账号   
            string password = "yourpassword";// 发送端密码(这个客户端重置后的密码)      

            SmtpClient client = new SmtpClient();
            client.DeliveryMethod = SmtpDeliveryMethod.Network;//指定电子邮件发送方式    
            client.Host = host;//邮件服务器
            client.Port = 25;  //465(ssl), 25(default)
            client.UseDefaultCredentials = true;
            client.Credentials = new System.Net.NetworkCredential(userName, password);//用户名、密码
            //client.EnableSsl = true;

            string strfrom = userName;
            string strto = "user1@anotherdomain.com";   //
            string strcc = "user2@anotherdomain2.com";//抄送

            string subject = "周报-XXX";   //邮件的主题             
            string body = "\nXX部门-周报-XX, 请查阅!\nsent by  SendWeekRptAutoService";//发送的邮件正文  

            string mailfile = "C:\\Users\\qurete\\Desktop\\XX部门-周报-XX.xlsm";
            Attachment mailattached = new Attachment(mailfile, MediaTypeNames.Application.Octet);

            System.Net.Mail.MailMessage msg = new System.Net.Mail.MailMessage();
            msg.From = new MailAddress(strfrom, "jjjj");
            msg.To.Add(strto);
            msg.CC.Add(strcc);
            strcc = "18833335953@163.com";
            msg.CC.Add(strcc);
            msg.Attachments.Add(mailattached);
            msg.Subject = subject;//邮件标题   
            msg.Body = body;//邮件内容   
            msg.BodyEncoding = System.Text.Encoding.UTF8;//邮件内容编码   
            msg.IsBodyHtml = false;//是否是HTML邮件   
            msg.Priority = MailPriority.High;//邮件优先级   

            try
            {
                client.Send(msg);
                logger.Info("SendWeekRptAutoService-->sendmail send ok bang bang bang");
            }
            catch (System.Net.Mail.SmtpException ex)
            {
                logger.Info("SendWeekRptAutoService-->sendmail send fail, error: " + ex.Message);
                return false;
            }
            return true;
        }
</code></pre>

###注册和注销Windows服务

使用InstallUtil.exe实用工具（需要使用管理员权限），一般写成批处理文件备用。

- 注册Windows服务
<pre><code>
%~dp0\InstallUtil.exe %~dp0\SendWeekRptAutoService.exe
rem 这里是安装完成之后启动服务
Net Start SendWeekRptAutoService
rem 这里是将服务设置为自动启动
sc config SendWeekRptAutoService start= auto
pause
</code></pre>

- 注销Windows服务

<pre><code>
%~dp0\InstallUtil.exe /u %~dp0\SendWeekRptAutoService.exe
pause
</code></pre>

###Enjoy

注册服务以后，我只要在每周五前不定时更新周报Excel文件就好了，不用担心错过发送周报的时间。
