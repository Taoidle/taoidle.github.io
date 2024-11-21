---
layout: Win11找不到本地组策略编辑器(gpedit.msc)
title: Win11找不到本地组策略编辑器(gpedit.msc)
categories:
  - - 折腾记录
tags: 
  - windows
date: 2024-11-21 11:08:40
---
# 解决方式

以管理员方式打开cmd，运行以下命令：

```bat
FOR %F IN ("%SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientTools-Package~*.mum") DO ( DISM /Online /NoRestart /Add-Package:"%F" )

FOR %F IN ("%SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientExtensions-Package~*.mum") DO ( DISM /Online /NoRestart /Add-Package:"%F" )
```

等命令运行完成，Win + R 输入gpedit.msc，回车即可打开本地组策略编辑器