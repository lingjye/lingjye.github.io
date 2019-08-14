---
layout: post
title: "iOS知识点总结"
subtitle: ''
author: "lingjye"
header-style: 'text'
tags:
  - iOS
---

MLeakFinder源码
SD,AF,YYCache,AsynDisplay,Texsure
MDM
GCD
Flutter MVVM 埋点，MVP
Router，JLRouts，原理
FMDB sql查询语句
编译原理
RAC，冷热信号，各类信号
Runtime,Runloop
内存管理
KVO
WebSocket，MQTT,TCP/IP,Http

**property 常用的后面修饰符，有啥特殊情况，怎么处理**

1. 线程安全的： atomic, nonatomic, 默认atomic，用来防止在未写完成时被另一线程读取修改，但是比较耗费系统资源。nonatomic禁止多线程，变量保护，提高性能。
2. 访问权限的： readonly， readwrite，readonly修饰的属性只有getter，没有setter，可以在.m的分类接口中将该属性重新声明为readwrite。
3. 内存管理（ARC） assign，strong，weak，copy
4. 内存管理（MRC）assign， retain，copy 
	
	strong和retain区别联系：
		
	相同点：
	strong和retain都是针对对象类型进行内存管理，用来修饰对象类型，不能用来修饰基本数据类型。当修饰对象类型时，setter方法会先将旧的对象release掉，然后再将新的对象赋值给属性，并对该对象进行一次retain操作，两者都会增加对象的引用计数。
	不同点：
	strong一般用于ARC环境，retain用于MRC环境。
		
	strong是由通过runtime维护的一个自动计数表结构管理。
		
	assign和weak的区别和联系：
		
	相同点：
	他们修饰的属性都不会增加引用计数
	不同点：
	assign可修饰基本数据类型，也可修饰OC对象，但是如果修饰对象类型指向的是一个强指针，当它指向的这个指针释放后，它仍然指向这段被释放的内存，会产生野指针，需要手动将其置位nil，否则再次调用已经释放的属性，会访问已经释放的内存，导致EXC_BAD_ACCESS错误。使用weak则只能修饰OC对象，不能修饰基本数据类型（Property with ‘weak’ attribute must be of object type错误），并且在对象释放后自动置为nil，不会产生野指针，相对比较安全。另外weak还可以解决循环引用问题，例如delegate，block。
		
	weak:
		
	Runtime维护了一个hash表（weak表，全局弱引用hash表），用于存储指向某个对象的所有weak指针，该表是由单个自旋锁（spinlock_lock）管理的散列表。在该表中，key是所指对象的指针，value是weak指针的地址数组，而这个地址则是所指向对象的地址。他的作用就是讲对象执行dealloc时将所有指向该对象的weak指针值设为nil，避免悬空指针。它使用不定类型对象的地址的 hash 化后的数值作为 key，用 weak_entry_t 类型的结构体对象作为 value。
		
	weak表weak_table_t结构：
		
	```
	struct weak_table_t {
	    weak_entry_t *weak_entries; 			// 保存了所有指向指定对象的weak指针
	    size_t    num_entries;              // weak对象的存储空间, entries的数目
	    uintptr_t mask;                     // 参与判断引用计数辅助量
	    uintptr_t max_hash_displacement;    // hash key 最大偏移值
	};
	```
		
	copy和strong区别与联系
		
	相同点：
	copy和strong都可修饰不可变类型，但一般用copy修饰不可变类型，以及可变类型，都需要看属性的来源。
		
	不同点：
	使用copy可以保证属性的封闭性，更加安全。copy修饰的属性，在setter方法中会自动判断来源如果不可变，则和strong一样进行浅拷贝，会增加其引用计数，如果是可变的就深拷贝，不增加其引用计数，所以当给不可变对象赋值时，如果来源是可变的，那么使用copy，如果来源是不可变的，使用strong，避免setter内部的if判断过多影响性能。使用strong修饰不可变属性时，如果不可变属性的来源是可变时，不可变属性也会跟着变化，这样就破坏了其封闭性，显得不安全。
		
	如果用copy修饰可变类型，会生成一个不可变的对象，使用赋值操作给可变属性没用问题，但是调用属性的可变方法时会报错（unrecognized selector sent to instance）。
		
	如果用strong修饰可变类型，只能修饰来源可变的属性，如果来源不可变，则编译器报错（unrecognized selector sent to instance）。
		
	例如：NSString
	
5. 指定方法名称： setter= getter=
	
	就是不用系统getter 和 setter，替换成自定义的函数。


**MLeakFinder的源码实现原理**

直接拖入项目，无需代码，即可使用，依赖于 FBRetainCycleDetector 寻找泄露者。AOP方式注入，项目无侵蚀。

原理： 当对象执行 dissmiss 以及 pop 操作2s后检测对象是否还存在，来判断内存泄露。可以通过NSObject+MemoryLeak分类文件查看。

NSObject+MemoryLeak 分类中为基类 NSObject 添加一个 willDealloc 方法，它先用一个弱指针指向 self，并在一小段时间 (2秒) 后，通过这个弱指针调用 -assertNotDealloc，而 assertNotDealloc 主要作用是打印堆栈信息。

在 UIApplication+MemoryLeak 分类中 hook sendAction:to:from:forEvent:方法，将事件响应者sender与application管理，方便后边获取。

在 "UINavigationController+MemoryLeak 分类中 hook pushViewController:animated:，popViewControllerAnimated:， popToViewController:animated:， popToRootViewControllerAnimated:方法，在 UIViewController+MemoryLeak 分类中 hook viewDidDisappear， viewWillAppear， dismissViewControllerAnimated:completion 方法，来调用自身及 childrens 的 willDealloc 方法，在其内部实现了对 subviews 的 willDealloc 方法调用。

在对象释放前 willDealloc 中有一个GCD延时调用 assertNotDealloc，2秒后获取堆栈信息，如果被释放成功，实际上不会调用 assertNotDealloc 的。如果获取到堆栈信息，则弹出Alert，点击Retain Cycle后，使用 FBRetainCycleDetector 检测当前未释放对象上未释放的对象。

对于某些不需要检测的对象，可以调用 addClassNamesToWhitelist 方法，添加进白名单列表。

MLeaksFinder.h

```
//#define MEMORY_LEAKS_FINDER_ENABLED 0
//_INTERNAL_MLF_ENABLED 宏用来控制 MLLeaksFinder库 
//什么时候开启检测，可以自定义这个时机，默认则是在DEBUG模式下会启动，RELEASE模式下不启动
//它是通过预编译来实现的
#ifdef MEMORY_LEAKS_FINDER_ENABLED
#define _INTERNAL_MLF_ENABLED MEMORY_LEAKS_FINDER_ENABLED
#else
#define _INTERNAL_MLF_ENABLED DEBUG
#endif

//#define MEMORY_LEAKS_FINDER_RETAIN_CYCLE_ENABLED 1
//COCOAPODS 因为MLLeaksFinder引用了FBRetainCycleDetector用来检查循环引用，所以必须是当前项目中使用了COCOAPODS，才能使用这个功能。
#ifdef MEMORY_LEAKS_FINDER_RETAIN_CYCLE_ENABLED
#define _INTERNAL_MLF_RC_ENABLED MEMORY_LEAKS_FINDER_RETAIN_CYCLE_ENABLED
#elif COCOAPODS
#define _INTERNAL_MLF_RC_ENABLED COCOAPODS
#endif

```

**原生与JS交互，具体的实现方式**

1. 通过webView的shouldStartLoadWithRequest方法进行拦截，根据scheme进行处理，这种方法适合没有返回值的交互（JS调用OC方法），可以根据NSSelectorFromString来动态生成选择子，使用performSelector方法来进行调用以及传参。

	还可以在 webViewDidFinishLoad 方法中调用 stringByEvaluatingJavaScriptFromString 方法来调用JS方法。
	
	WKWebView中通过 userContentController 把需要观察的 JS 执行函数注册起来。然后通过一个协议方法，将所有注册过的 JS 函数执行的参数传递到此协议方法中。
	
	```
	[webView.configuration.userContentController addScriptMessageHandler:self name:@"jsFunc"];
	// WKScriptMessageHandler 协议
	- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{
		NSLog(@"%@", message);
	}
	```

2. 使用JavaScriptCore，他是WebKit中用来解释JavaScript的核心引擎。该框架主要由JSVirtualMachine、JSContext、JSValue类组成。每个JSVirtualMachine对象只能执行一个线程，多个JSVirtualMachine之间的对象无法传递。它为JavaScript代码的运行提供虚拟环境，拥有自己的GC机制。JSContext是执行上下文，负责js和原生数据传递。JSValue是JavaScript的值对象，用于对原生value进行转换。同一个JSVirtualMachine包含多个JSContext，同一个JSContext中也可包含多个JSValue。

	示例：
	 
	```
	_jsContext = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
	_jsContext.exceptionHandler = ^(JSContext *context, JSValue *exception) {
	NSLog(@"%@", exception);
	};
	```	
		
	[Demo](https://github.com/lingjye/iOS-Learning/tree/master/JSBrigde){:target="_blank"}

3. WebViewJavascriptBridge 

	它也需要提前约定来进行交互，过程如下：
	
	* JS 端加入 src 为 https://__bridge_loaded__ 的 iframe，
	* Native 端检测到 Request，检测如果是 __bridge_loaded__ 则通过当前的 WebView 组件注入 WebViewJavascriptBridge_JS 代码
	* 注入代码成功之后会加入一个 messagingIframe，其 src 为 https://__wvjb_queue_message__
	* 之后不论是 Native 端还是 JS 端都可以通过 registerHandler 方法注册一个两端约定好的 HandlerName 的处理，也都可以通过 callHandler 方法通过约定好的 HandlerName 调用另一端的处理（两端处理消息的实现逻辑对称） [参考原文](https://juejin.im/post/5a40492f6fb9a0451969ce95){:target="_blank"}


**AOP可以用到哪些地方，有啥优缺点？**


AOP(Aspect Oriented Programming) 面向切面编程，是OOP（Object Oriented Programming）的延续。

OOP的特点在于它可以很好的将系统纵向分为多个模块，每个子模块也可以横向的衍生出更多的模块，用于更好的区分业务逻辑。而AOP其实相当于是横向的切入系统模块，将个个模块里的公共部分提取出来（即那些与业务逻辑不相关的部分，如日志，事件统计等等），与业务逻辑相分离开来，从而降低代码的耦合度。AOP主要是被使用在日志记录，性能统计，安全控制，事务处理，异常处理等几个方面。
 
在iOS中使用AOP思想进行开发的技巧，莫过于Method Swizzling。

**AOP的优势：**
 
* 减少切面业务的开发量，“一次开发终生使用”，比如日志。
* 减少代码耦合，方便复用。切面业务的代码可以独立出来，方便其他应用使用。
* 提高代码review的质量，比如我可以规定某些类的某些方法才用特定的命名规范，这样review的时候就可以发现一些问题。

AOP的弊端：
 
* 它破坏了代码的干净整洁。
* 需要在+load方法中进行交换，在其他时候进行交换无法保证其他线程同时调用被交换的方法，导致程序无法按预期执行。
* 被交换的方法必须是当前类的方法们不能是父类的方法。
* 如果交换的方法依赖了cmd，交换后cmd发生变化时，出现问题难以排查。尤其是交换系统方法，无法保证系统方法内部是否依赖cmd。
*也无法保证出现命名冲突。

代表Aspects。


**load vs initialize**

Objective-C运行时为每个类自动调用两种方法。+load在最初加载类时调用，而+initialize在应用程序使用该类或该类的实例时调用其第一个方法之前调用。两者都是可选的，只有在实现方法时才会执行。

在Method swizzling会影响全局状态，因此最小化竞争条件非常重要。+load保证在类初始化期间加载，这为更改系统范围的行为提供了一定的一致性。相比之下，+initialize它没有提供何时执行的保证，如果该类永远不被应用程序使用，那么它就永远不会被调用。另外在+load中使用Method swizzling，一定要使用dispatch_once来保证代码只执行一次。
