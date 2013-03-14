---
layout: post
title: "UML 建模之时序图(Sequence Diagram)"
date: 2013-03-10 15:52
comments: true
categories: UML设计
tags: UML
---
>原文:[http://www.cnblogs.com/ywqu](http://www.cnblogs.com/ywqu "灵动生活")

__一、时序图简介（Brief introduction）__  
__二、时序图元素（Sequence Diagram Elements）__

>
+ 角色（Actor）  
+ 对象（Object）  
+ 生命线(Lifeline)  
+ 控制焦点（Focus of Control）  
+ 消息（Message）  
+ 自关联消息（Self-Message）  
+ Combined Fragments  

__三、时序图实例分析（Sequece Diagram Example Analysis）__

>
+ 时序图场景  
+ 时序图实例  
+ 时序图实例分析  

__四、总结（Summay）__

###一、时序图简介（Brief introduction）
   时序图是显示__对象之间交互的图__,这些__对象时按时间顺序排列__的。顺序图中显示的是__参与交互对象之间消息交互的顺序__。时序图中包括建模元素主要有:__对象、控制焦点、消息__等等。
###二、时序图元素（Sequence Diagram Elements）
__角色__：系统角色，可以是人、甚至其他的系统或子系统  
__对象__：对象包括三中命名方式：  
   1. 包括对象名和类名  
   2. 只显示类名不显示对象名，即表示它是匿名对象  
   3. 只显示对象名不显示类名    
![对象的命名方式](/images/common/2013-03-10-sequence-diagram/3347379_1.jpg 'Object')

__生命线__：生命线在顺序图中表示从对象图片向下延伸的一条虚线,表示对象存在的时间,如下图：![生命线](/images/common/2013-03-10-sequence-diagram/3347379_2.gif 'Lifeline')

__控制焦点__：控制焦点是顺序图中表示时间的符号，在这个时间段内对象执行相应的操作，用小矩形表示，如下图：![控制焦点](/images/common/2013-03-10-sequence-diagram/3347379_3.jpg 'Focus of Control')  

__消息__：消息一般分为同步消息（Synchronous Message），异步消息（Asynchronous Message）和返回消息（Return Message）如下图：  
![消息](/images/common/2013-03-10-sequence-diagram/3347379_4.gif 'Message')  

+   同步消息 == 调用消息  
消息的发送者把控制传递给消息的接收者,然后停止活动,等待消息的接受者放弃或者返回控制。用来表示__同步__的意义。
+   异步消息
消息的发送者通过消息把信号传递给消息的接受者,然后继续自己的活动,不等待接受者返回消息或控制。异步消息的接受者和发送者是并发工作的。
+   返回消息
返回消息表示从过程调用返回

__自关联消息__ : 表示方法的自身调用以及一个对象内的一个方法调用另一个方法。如下图:  
![自关联消息](/images/common/2013-03-10-sequence-diagram/3347379_5.gif 'Sef-Message')

__Combined Fragments__  
![Combined Fragments](/images/common/2013-03-10-sequence-diagram/3347379_6.gif 'Combined Fragments')  

+   Altemative fragment(denoted "alt")与 if...then...else对应
+   Option fragment(denoted "opt")与Switch对应
+   Parallel fragment(denoted "par")表示同时发生
+   Loop fragment(denoted "loop")与for或foreach对应

###三、时序图实例分析
__时序图场景__  
完成课程创建功能,主要流程有:  
1. 请求添加课程页面,填写课程表单,点击`create`按钮  
2. 添加课程信息到数据库  
3. 向课程对象追加主题信息  
4. 为课程指派教师  
5. 完成课程创建功能  

__时序图实例__
![时序图实例](/images/common/2013-03-10-sequence-diagram/3347379_7.jpg) 

__时序图分析__  
1. 序号1.0-1.3   完成页面的初始化  
2. 序号1.4-1.5   课程管理员填充课程表单  
3. 序号1.6-1.7   课程管理员点击`create`按钮,并相应点击事件  
4. 序号1.8       Service 层创建课程  
5. 序号1.9-1.10  添加课程到数据库,并返回课程编号CourseId  
6. 序号1.11-1.12 添加课程主题到数据库,并返回主题编号topicId  
7. 序号1.13      给课程指派教师   
8. 序号1.14      向界面抛创建课程成功与否的消息  

###四、总结
   时序图是显示对象之间交互顺序的图,这次对象是按时间顺序排列的.时序图中显示的是参与交互的对象及对象之间消息交互的顺序.时序图中包括的建模的元素主要有:对象、控制焦点、消息等等。
