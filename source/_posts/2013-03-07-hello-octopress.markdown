---
layout: post
title: "Hello Octopress,在windwow平台下自己动手搭建一个GitHub Blog!"
date: 2013-03-07 11:08
comments: true
categories: 
---
> `github`本身不仅作为代码共享仓库，并且支持`github-pages`功能，而`octopress`就是在这样的环境下诞生的博客系统. 
     
### 需要安装的软件
	1. git
	2. ruby(或者说rubyinstaller)
	3. DevKit(windows平台下编译和使用本地C/C++扩展包的工具,即用来模拟Linux平台下的make,gcc,sh命令来进行编译)
	4. python(支持代码高亮)    

### 搭建本地ruby环境和配置工具
依次安装`git`,`rubyinstaller`和`python`,建议最好安装在C盘(系统盘)下,并把相应路径加入系统环境变量中,以避免不必要的错误。之后进入cmd窗口或git窗口来安装DevKit,解压DevKit,比如C:/DevKit,然后执行如下命令:
	c:                          #进入c盘
	cd c:/DevKit                
	ruby dk.rb init             #执行该命令时,可能会由于编码错误导致失败,则在后面加入 --encoding=utf-8
	ruby dk.rb install
如果至此你都未出现任何错误的话,搭建成功!
