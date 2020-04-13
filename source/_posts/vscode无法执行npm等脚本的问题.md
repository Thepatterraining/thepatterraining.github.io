---
title: vscode无法执行npm等脚本的问题
date: 2020-04-12 11:38:00
tags: ['php','vscode', 'javascript', 'npm']
category: 工具
article: vscode无法执行npm等脚本的问题
---

# vscode无法执行npm等脚本的问题

这是因为在windows系统上面powershell的执行策略问题。

可以运行下面的命令查看当前执行策略

> Get-ExecutionPolicy

使用下面的命令更改执行策略

> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

`-scope`参数是 限制在当前用户下面更改 策略更改为`RemoteSigned`策略。

##### RemoteSigned

- Windows服务器计算机的默认执行策略。
- 脚本可以运行。
- 需要从可信任的发布者处获得从互联网下载的脚本和配置文件的数字签名，其中包括电子邮件和即时消息传递程序。
- 不需要在本地计算机上编写且未从Internet下载的脚本上进行数字签名。
- 如果脚本不受阻碍（例如使用Unblock-Filecmdlet），则运行从Internet下载且未签名的脚本。
- 可能会运行来自Internet以外的其他来源的未签名脚本和可能有害的已签名脚本。

##### Restricted

- Windows客户端计算机的默认执行策略。
- 允许使用单个命令，但不允许使用脚本。
- 阻止运行所有脚本文件，包括格式和配置文件（.ps1xml），模块脚本文件（.psm1）和PowerShell配置文件（.ps1）。

##### Undefined

- 当前范围中未设置执行策略。
- 如果所有作用域中的执行策略均为Undefined，则有效的执行策略为Restricted，这是默认的执行策略。

##### Unrestricted

- 非Windows计算机的默认执行策略，不能更改。
- 未签名的脚本可以运行。有运行恶意脚本的风险。
- 在运行非本地Intranet区域中的脚本和配置文件之前警告用户。

更多问题可以参考`powershell`的官方文档

> https://docs.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7