---
layout: post
title: "Objective-C消息传递与消息转发机制"
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
* objc_msgSend_fp2ret，同上，在一些处理器上，用来发送返回值类型为浮点类型的消息（x86-64）；
* objc_msgSendSuper，给父类发送消息；
* objc_msgSendSuper_stret，同上。

在objc_msgSend函数中还实现了尾递归优化，来避免过早地发生栈溢出。

## 消息转发机制

当对象接收到无法解读的消息时，会启动消息转发机制，我们也可经由此转发过程来处理未知消息。

如果未经处理，则会由NSObject的`doesNotRecognizeSelector:`方法抛出异常。

消息转发阶段：

1. 动态方法解析；
2. 完整的消息转发机制。

编译器无法确定某类型对象到底能解读多少种选择子，因为运行期还可以向其中动态新增。

### 动态方法解析

对象收到无法解读的消息后，先询问所属类，是否可以动态添加方法来处理未知选择子，调用方法：

```
// 参数为对应的选择子
+ (BOOL)resolveInstanceMethod:(SEL)selector;
```

在继续往下执行转发机制之前，本类有机会新增一个处理此选择子的方法。

与此相对应的，如果调用的方法是类方法，则调用方法：

```
+ (BOOL)resolveClassMethod:(SEL)selector;
```

当然，使用该方式处理未知消息，需要提前将实现代码写好，等待运行时动态插在类中。

下面使用resolveInstanceMethod:来实现@dynamic属性，示例：

```
// Setter 方法
void autoDictionarySetter(id self, SEL _cmd, id value) {
    MFLAutoDictionary *typeSelf = (MFLAutoDictionary *) self;
    NSMutableDictionary *backingStore = typeSelf.backingStore;

    NSString *selectorString = NSStringFromSelector(_cmd);
    NSMutableString *key = [selectorString mutableCopy];

    //remove the ':' at the end
    [key deleteCharactersInRange:NSMakeRange(key.length - 1, 1)];

    //remove the 'set' prefix
    [key deleteCharactersInRange:NSMakeRange(0, 3)];

    //lowercase the first character
    NSString *lowercaseString = [[key substringToIndex:1] lowercaseString];

    [key replaceCharactersInRange:NSMakeRange(0, 1) withString:lowercaseString];

    if (value) {
        [backingStore setObject:value forKey:key];
    } else {
        [backingStore removeObjectForKey:key];
    }
}

// Getter方法
id autoDictionaryGetter(id self, SEL _cmd) {
    MFLAutoDictionary *typeSelf = (MFLAutoDictionary *) self;
    NSMutableDictionary *backStore = typeSelf.backingStore;

    NSString *key = NSStringFromSelector(_cmd);
    return [backStore objectForKey:key];
}

// 动态解析处理
+ (BOOL)resolveInstanceMethod:(SEL)sel {
	 // 将选择子转换为字符串
    NSString *selectorString = NSStringFromSelector(sel);
    // 如果包含set前缀，则表示set方法
    if ([selectorString hasPrefix:@"set"]) {
    	 // 添加处理该选择子的方法到类中，所添加的方法是用C函数实现的
        class_addMethod(self,
                        sel,
                        (IMP) autoDictionarySetter,
                        "v@:@");
    } else {
    	 // 如果包含get前缀，则表示get方法
        class_addMethod(self,
                        sel,
                        (IMP) autoDictionaryGetter,
                        "@@:");
    }
    return YES;
}

```

### 备用接收者

当resolveInstanceMethod:返回NO时，运行期系统还是会询问当前接收者是否可以把这条消息转给其他接收者来处理。此时，调用方法：

```
// 参数依然是未知的选择子
- (id)forwardingTargetForSelector:(SEL)selector
```

如果当前接收者能找到备用对象，则将其返回，否则返回nil。通过这种方案，可以使用“组合（composition）”来模拟出“多重继承（multiple inheritance）”的某些特性。在一个对象内部，可能还有一系列其他对象，该对象可经由此方法将能够处理某选择子的相关内部对象返回。这样在外界看起来就好像是由该类处理了这些消息。

但是，这一步只能返回接收消息的备用者，无法对消息进行处理。如果需要在发送给备用接收者之前修改消息内容，则需要通过完整的消息转发机制。

### 完整的消息转发

如果forwardingTargetForSelector返回的备用接收者为nil，则转发算法启用完整的消息转发机制。

在该过程中，首先会创建一个NSInvocation对象，然后把与尚未处理的消息有关的全部细节都封装起来，包含目标（target）、选择子以及参数等。在触发NSInvocation对象时，将启用消息派发系统（message-dispatch system）来把消息指派给目标对象。

此过程调用方法：

```
- (void)forwardInvocation:(NSInvocation *)anInvocation
```

该过程只需要改变调用目标，然后使消息在新目标上得以调用即可。但是使用这种方法与“备用接收者”方案所实现的方法是等效的，所以一般采用的实现方式是：在触发消息前，先用其他方法来改变消息内容，其调用函数为：

```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)selector
```

在这个函数中可以追加一些其他参数，甚至是改变选择子等。

在实现该方法时，如果发现其调用操作不应由本类处理，则需要调用父类的同名方法。即：

```
return [super methodSignatureForSelector:aSelector];
```

这样，继承体系中的每个类都有机会来处理此调用请求，直至NSObject类。如果最后调用了NSObject类的方法，那么该方法还会继而调用`doesNotRecognizeSelector:`方法抛出异常，如果抛出异常，则表明选择子最终没有得到处理。

### 消息转发全流程

 消息转发机制处理消息的各个步骤：
 
 ![消息转发](https://raw.githubusercontent.com/lingjye/lingjye.github.io/master/img/iOS/messageforwarding.jpg)

在消息转发过程中，越往后处理消息的代价越大。如果在第一步就处理完，则运行期系统会将此方法缓存起来，等到这个类接收到同名选择子时，无须启动消息转发流程，这也是为什么在测试中`resolveInstanceMethod:`只调用一次的原因。如果第三步中只是把消息转给备用接收者，建议把转发操作提前到第二步，否则还需要创建并处理完整的NSInvocation。

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