---
title: Linux >> Tail
author: Jiaqi Gao
description: 
date: 2023-06-01T23:48:42+08:00
lastmod: 2023-06-01T23:48:42+08:00
slug: linux-tail
image: 
math: false
license: 
hidden: false
comments: true
draft: false
categories:
  - Linux
tags:
  - Linux
series:
aliases:
---
## 简介

> 显示文件的最后一部分

## 语法

```shell
tail [OPTION] FILE...
```

## 选项

```shell
-c, --bytes=NUM                    # 输出文件尾部的NUM（NUM为整数）个字节内容。
-f, --follow[={name|descript}]     # 显示文件最新追加的内容。“name”表示以文件名的方式监视文件的变化。
-F                                 # 与 “--follow=name --retry” 功能相同。
-n, --line=NUM                     # 输出文件的尾部NUM（NUM位数字）行内容。
--pid=<进程号>                      # 与“-f”选项连用，当指定的进程号的进程终止后，自动退出tail命令。
-q, --quiet, --silent              # 当有多个文件参数时，不输出各个文件名。
--retry                            # 即是在tail命令启动时，文件不可访问或者文件稍后变得不可访问，都始终尝试打开文件。使用此选项时需要与选项“--follow=name”连用。
-s, --sleep-interal=<秒数>          # 与“-f”选项连用，指定监视文件变化时间隔的秒数。
-v, --verbose                      # 当有多个文件参数时，总是输出各个文件名。
--help                             # 显示指令的帮助信息。
--version                          # 显示指令的版本信息。
```

## 参数

文件列表：指定要显示尾部内容的文件列表。

## 补充说明

* 默认在屏幕上显示指定文件的末尾10行。
* 处理多个文件时会在各个文件之前附加含有文件名的行。
* 如果没有指定文件或者文件名为`-`，则读取标准输入。
* 如果表示字节或行数的`NUM`值之前有一个`+`号，则从文件开头的第`NUM`项开始显示，而不是显示文件的最后`NUM`项。
* `NUM`值后面可以写单位：
  * `b`：512
  * `kB`：1000
  * `k`：1024
  * `MB`：1000**2
  * `M`：1024**2
  * `GB`：1000**3
  * `G`：1024**3
  * `T`、`P`、`E`、`Z`、`Y`等以此类推。