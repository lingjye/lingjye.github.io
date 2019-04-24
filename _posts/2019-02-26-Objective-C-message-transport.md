---
layout: post
title: "Objective-C消息转发机制"
subtitle: ''
author: "lingjye"
header-style: text
tags:
  - iOS
---

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
