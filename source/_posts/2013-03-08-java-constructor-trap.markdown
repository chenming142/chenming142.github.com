---
layout: post
title: "Java构造时成员初始化的陷阱"
date: 2013-03-08 22:21
comments: true
categories: J2SE基础
tags: J2SE
---
>原文:[http://blog.csdn.net/haoel/article/details/4319793](http://blog.csdn.net/haoel/article/details/4319793 "陈皓专栏")   

让我们先来看两个类: __Base__ 和 __Derived__ 类.注意其中的 _whenAmISet_ 的成员变量,和方法 _preProcess()_.
	public class Base{
	    Base(){
			preProcess();
	    }
		void preProcess();
	}
	public class Derived extends Base{
		public String whenAmISet = "set when declared";
		@Override void preProcess(){
			whenAmISet = "set in preProcess()";
		}
	}
如果我们构造一个子类实例,那么 __whenAmISet__ 的值会是什么呢?
	public class Main{
		public static void main(String[] args){
			Derived d = new Derived();
			System.out.println( d.whenAmISet );
		}
	}
再续继往下阅读之前，请先给自己一些时间想一下上面的这段程序的输出是什么？是的，这看起来的确相当简单，甚至不需要编译和运行上面的代码，我们也应该知道其答案，那么，你觉得你知道答案吗？你确定你的答案正确吗？

很多人都会觉得那段程序的输出应该是`set in preProcess()`，这是因为当子类Derived 的构造函数被调用时，其会隐晦地调用其基类Base的构造函数（通过super()函数），于是基类Base的构造函数会调用preProcess() 函数，因为这个类的实例是Derived的，而且在子类Derived中对这个函数使用了override关键字，所以，实际上调用到的是：`Derived.preProcess()`，而这个方法设置了whenAmISet 成员变量的值为：`set in preProcess()`。

当然，上面的结论是错误的。如果你编译并运行这个程序，你会发现，程序实际输出的是`set when declared `。怎么为这样呢？难道是基类Base 的preProcess() 方法被调用啦？也不是！你可以在基类的preProcess中输出点什么看看，你会发现程序运行时，__Base.preProcess()__ 并没有被调用到（不然这对于Java所有的应用程序将会是一个极具灾难性的Bug）

虽然上面的结论是错误的，但推导过程是合理的，只是不完整，下面是整个运行的流程：
	1. 进入 Derived 构造函数
	2. Derived 成员变量的内存被分配
	3. Base 构造函数被隐含的调用
	4. Base 构造函数调用 preProcess()
	5. Derived 的 preProcess() 设置 whenAmISet 值为"set in preProcess()"
	6. Derived 的成员变量初始化被调用
	7. 执行 Dervied 构造函数体
等一等，这怎么可能？在第6步，__Derived__ 成员的初始化居然在 __preProcess()__ 调用之后？  
是的，正是这样，我们不能让成员变量的声明和初始化变成一个原子操作，虽然在Java中我们可以把其写在一起，让其看上去像是声明和初始化一体。__但这只是假象，我们的错误就在于我们把Java中的声明和初始化看成了一体__ 。在C++的世界中，C++并不支持成员变量在声明的时候进行初始化，其需要你在构造函数中显式的初始化其成员变量的值，看起来很土，但其实C++用心良苦。

在面向对象的世界中，因为程序以对象的形式出现，导致了我们对程序执行的顺序雾里看花。__所以，在面向对象的世界中，程序执行的顺序相当的重要__ 。

下面是对上面各个步骤的逐条解释:
	1. 进入构造函数。
	2. 为成员变量分配内存。
	3. 除非你显式地调用super()，否则Java 会在子类的构造函数最前面偷偷地插入super() 。
	4. 调用父类构造函数。
	5. 调用preProcess，因为被子类override，所以调用的是子类的。
	6. 于是，初始化发生在了preProcess()之后。  
	   这是因为，Java需要保证父类的初始化早于子类的成员初始化，否则，在子类中使用父类的成员变量就会出现问题。
	7. 正式执行子类的构造函数（当然这是一个空函数，居然我们没有声明）。
你可以查看 [《Java语言的规格说明书》](http://java.sun.com/docs/books/jls/third_edition/html/execution.html#12.5,'相关章节') 来了解更多的Java创建对象时的细节。  
C++的程序员应该都知道，在C++的世界中在`构造函数中调用虚函数`是不行的，Effective C++ 条款9：Never call virtual functions during construction or destruction，Scott Meyers已经解释得很详细了。

在语言设计的时候，__“在构造函数中调用虚函数”__是个两难的问题:
	1. 如果调用的是父类的函数的话，这个有点违反虚函数的定义。  
	2. 如果调用的是子类的函数的话，这可能产生问题的：
	     因为在构造子类对象的时候，首先调用父类的构造函数,而这时候如果去调用子类的函数  
	     由于子类还没有构造完成，子类的成员尚未初始化，这么做显然是不安全的。
C++选择了第一种，而Java选择了第二种。  
	C++类的设计相对比较简陋，通过虚函数表来实现，缺少类的元信息。
	而Java类的则显得比较完整，有super指针来导航到父类。

最后，需要向大家推荐一本书，Joshua Bloch 和 Neal Gafter 写的 `Java Puzzlers: Traps, Pitfalls, and Corner Cases` ，中文版`《JAVA解惑 》`。
	
