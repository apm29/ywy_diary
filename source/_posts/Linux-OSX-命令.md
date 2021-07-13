---
title: Linux/OSX 命令
tags:
  - Linux
  - Shell
categories: []
toc: false
date: 2020-01-07 11:05:29
---

查看进程占用

`lsof -i tcp:8080` 

该命令会显示占用8080端口的进程，有其 pid ,可以通过pid关掉该进程

杀死进程 

`kill pid`

删除当前目录下除xx、xxx文件外的所有文件

`rm -rf !(xx|xxx)`