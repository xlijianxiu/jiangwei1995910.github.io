---
layout:     post
title:      "关于netsh winsock reset"
subtitle:   ""
date:       2016-12-17 10:00:00
author:     ""
header-img: "img/18.jpg"
catalog: true
tags:
    - Windows
---
>转载于网络

<p>
winsock是Windows网络编程接口，winsock工作在应用层，它提供与底层传输协议无关的高层数据传输编程接口 netsh winsock reset 是把它恢复到默认状态
</p>

<h2> 目录:</h2>
<table>
<tr>
	<td><a href="#jieshao">介绍</a></td>
	<td><a href="#vista">Vista</a></td>
	<td><a href="#win10">Win10</a></td>
</tr>
<tr>
	<td><a href="#xp">WinXP</a></td>
	<td><a href="#win7">Win7</a></td>
	<td><a href="#other">修复方法</a></td>
</tr>
</table>
---

<p id="jieshao"></p>
<h2> 介绍</h2>
<p>netsh winsock reset命令，作用是重置 Winsock 目录。如果一台机器上的Winsock协议配置有问题的话将会导致网络连接等问题，就需要用netsh winsock reset命令来重置Winsock目录借以恢复网络。这个命令可以重新初始化网络环境，以解决由于软件冲突、病毒原因造成的参数错误问题。 netsh是一个能够通过命令行操作几乎所有网络相关设置的接口，比如设置IP，DNS，网卡，无线网络等，Winsock是系统内部目录，Winsock是Windows网络编程接口，winsock工作在应用层，它提供与底层传输协议无关的高层数据传输编程接口，reset是对Winsock的重置操作。当执行完winsock的命令重启计算机后，需要重新配置IP。</p>
---
<p id="xp"></p>
<h2> Win XP</h2>

<p>
要为 Windows XP 重置 Winsock，请按照下列步骤操作：<br>
1.单击“开始”，运行中输入cmd。<br>
2.然后输入命令 netsh winsock reset。<br>
3.重启计算机。<br>
</p>
---

<p id="vista"></p>
<h2>  Vista</h2>
<p>
要为 Windows Vista 重置 Winsock，请按照下列步骤操作：<br>
1.单击“开始”，在开始搜索框中键入cmd，右键单击“cmd.exe”，单击“以管理员身份运行”，然后按“继续”。<br>
2.在命令提示符处键入 netsh winsock reset，然后按 Enter。<br>
3.注意：如果该命令键入错误，则出现一条错误消息。重新键入该命令。当该命令成功完成时，出现一条确认消息，后跟一个新的命令提示符。然后转到步骤4。<br>
4.键入 exit，然后按 Enter。
</p>

---
<p id="win7"></p>
<h2>  Win7</h2>
<p>要为 Windows 7 重置 Winsock，请按照下列步骤操作：<br>
1.单击“开始”，在开始搜索框中键入cmd，右键单击“cmd.exe”，单击“以管理员身份运行”，然后按“继续”。<br>
2.在命令提示符处键入 netsh winsock reset，然后按 Enter(回车键)。<br>
3.注意：如果该命令键入错误，则出现一条错误消息。重新键入该命令。当该命令成功完成时，出现一条确认消息，后跟一个新的命令提示符。<br>
4.然后键入 exit，然后按 Enter，退出命令行对话框。<br>
</p>
---
<p id="win10"></p>
<h2>  Win10</h2>
<p>
部分用户在升级至Windows 10 系统后，会遇到除了自带的Microsoft Edge 浏览器外，其他应用都不能联网的情况，也可以通过该命令解决，具体操作如下：<br>
1、按Win+X，选择“命令提示符（管理员）”，注意这个不要选择到上面的那个“命令提示符”，不然你在输入命令后，可能会收到“请求的操作需要提升”的提示。<br>
2、在弹出的CMD窗口中输入“netsh winsock reset”（注意，不带双引号），然后回车；<br>
3、回车后，你将收到“成功重置 Winsock 目录，你必须重新启动计算机才能完成重置”的提示。这时重启你的计算机，网络即可恢复正常。 <br>
</p>
---




