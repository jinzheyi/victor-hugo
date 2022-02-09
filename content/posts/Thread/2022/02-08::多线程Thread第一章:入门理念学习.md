+++
title = "多线程Thread第一章；入门理念学习"
date = "2022-02-08 14:21:52"
url = "archives/1"
tags = ["Thread"]
categories = ["后端","Thread多线程学习"]
featuredImage = "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.yuncode.net%2Fupload%2Fimage%2F201304%2F04%2
F20130404170819_95029.jpg&refer=http%3A%2F%2Fwww.yuncode.net&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1647003374&t=8fb27e35b54cbfc53d5a856ec29dac9d"
+++

## 进程与线程 ##

### 进程 ###

- 程序由指令和数据组成，但这些指令要运行，数据要读写，就必须将指令加载至CPU，数据加载至内存。在指令运行过程中还需要用到磁盘、网络等设备。进程
就是用来加载指令、管理内存、管理IO的
- 当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程。
- 进程就可以视为程序的一个实例。大部分程序可以同时运行多个实例进程（例如记事本、画图、浏览器等），也有的程序只能启动一个实例进程（例如网易云音
乐、360安全卫士等）

### 线程 ###



