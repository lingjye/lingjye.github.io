---
layout: post
title: "Objective-C消息转发机制及动态解析"
subtitle: ''
author: "lingjye"
header-style: text
tags:
  - iOS
---

## 消息传递

在对象上调用方法，在Objective-C中称作“传递消息”。消息有“名称”（name）或“选择子”（selector，也称作方法），可以接收参数，而且还可能有返回值。

对于在编译期就能决定运行时所应调用的函数的调用方式，是“静态绑定”，而在运行期才能确定的函数调用方式是“动态绑定”。

在Objective-C中，如果向某对象传递消息，那么就会使用动态绑定机制来决定应该调用的方法。在底层调用上，所有方法都是C函数，在对象收到消息后，完全由运行期来决定该调用哪个方法，甚至可以再程序运行时改变，这使得Objective-C成为一门动态语言。

给对象发送消息：

```
/*
 * returnValue 作为返回值
 * object 消息接收者
 * messageName 选择子
 * parameter 参数
 */
id returnValue = [object messageName:parameter];
```

选择子与参数合起来成为“消息”。编译器看到发送的消息后，将其转换为一条标准的C语言函数调用，所调用的函数是消息传递机制中的核心函数：objc_msgSend。原型为：

```
/* 
 * 该函数是参数个数可变的函数
 * 参数self，代表接收者
 * cmd，代表选择子，SEL类型
 * 后续参数为消息中传递的参数，不是必须的

void objc_msgSend(id self, SEL cmd, ...)
```

上边给对象发送消息可转换为：

```
id returnValue = objc_msgSend(object, @selector(messageName:), parameter);
```

objc_msgSend函数会根据消息接收者和选择子的类型来调用适当的方法，其过程为：

1. 在接收者所属的类中的方法列表中寻找与选择子名称相符的方法(选择子的名称是查表时的“键”)；
2. 如果没找到，沿着继承体系继续向父类中寻找；
3. 如果还是没找到，执行“消息转发”操作。
4. 如果匹配到适当的方法，将匹配结果缓存到“快速映射表（fast map）”中，以供下次调用。

当objc_msgSend找到对应方法后，就会跳转到相应方法，在使用Xcode调试过程中会经常看到它出现在栈回溯信息中。

另外还有几个函数：

* objc_msgSend_stret，此函数可返回结构体，如果返回值太大，无法容纳于CPU寄存器中，需要由另一个函数执行派发；
* objc_msgSend_fpret，此函数可返回浮点类型数据；
* * objc_msgSend_fp2ret，同上，在一些处理器上，用来发送返回值类型为浮点类型的消息（x86-64）；
* objc_msgSendSuper，给父类发送消息；
* objc_msgSendSuper_stret，同上。

在objc_msgSend函数中还实现了尾递归优化，来避免过早地发生栈溢出。

## 消息转发机制

当对象接收到无法解读的消息时，会启动消息转发机制，我们也可经由此转发过程来处理未知消息。

消息转发阶段：

1. 动态方法解析；
2. 完整的消息转发机制。

**动态方法解析**

对象收到无法解读的消息后，先询问所属类，是否可以动态添加方法来处理未知选择子，调用方法：

```
// 参数为对应的选择子
+ (BOOL)resolveInstanceMethod:(SEL)selector;
```

与此相对应的，如果调用的方法是类方法，则调用方法：

```
+ (BOOL)resolveClassMethod:(SEL)selector;
```

当然，使用该方式处理未知消息，需要提前将实现代码写好，等待运行时动态插在类中。


## 附 Objective-C类型编码

| 	编码 	| 	含义 	|
| :--- | :--- |
| 	c		|	 char	|
| 	i		| 	int		|
| 	s		| 	short	|
|	 l		| 	long	 <br/> 在64位程序中，l为32位。|
|	 q		| 	long long	|
|	 C		| 	unsigned char	|
|	 I		|	 unsigned int	|
|	 S		| 	unsigned short	|
|	 L		| 	unsigned long	|
|	 Q		|	 unsigned long long	|
|	 f		| 	float	|
| 	d		| 	double	|
| 	B		|	 C++标准的bool或者C99标准的_Bool	|
| 	v		| 	void	|
| 	*		| 	字符串（char *）	|
| 	@		| 	对象（无论是静态指定的还是通过id引用的）	|
| 	#		| 	类（Class）	|
| 	:		| 	方法（SEL)	|
| 	[array type]	| 	数组	|
| 	{name=type...}	| 	结构体	|
| 	(name=type...)	| 	联合体	|
|	 bnum	| 	num个bit的位域	|
|	 ^type	| 	type类型的指针	|
| 	?		| 	未知类型（其它时候，一般用来指函数指针）	|

## 本文[Demo](https://github.com/lingjye/iOS-Learning/tree/master/MessageForwardLearning){:target="_blank"}