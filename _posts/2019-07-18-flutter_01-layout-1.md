---
layout: post
title: "Flutter基础——布局一"
subtitle: '拥有多个子元素的布局widget'
author: "lingjye"
header-style: text
tags:
  - Flutter
---

iOS中常常借助frame，VFL，AutoLayout，UIStackView或者Masonry，SnapKit等来实现子控件怎么布局。Flutter中的布局需要使用Widget，不同的Widget可以实现不同的效果。本文主要介绍几个基本的布局组件，及其用法。

### 

#### Container

Container一个拥有绘制、定位、调整大小的 widget。

**布局行为**

如果Container没有子窗口，没有height，没有width，没有约束，并且父窗口提供无限制约束，则Container会尝试尽可能小。

如果Container没有子widget，没有alignment，而是一个height，width或constraints提供，则Container试图给出这些限制和父widget的约束相结合，以尽可能小。

如果Container没有子widget，没有height，没有width，没有constraints，没有alignment，但是父widget提供了有界约束，那么Container会扩展以适应父widget提供的约束。

如果Container具有alignment，并且widget提供无限制约束，则Container会尝试围绕子widget调整自身大小。

如果Container具有alignment，并且父widget提供有界约束，则Container会尝试展开以适合父widget，然后根据对齐方式将子widget置于其自身内部。

如果Container有一个子widget，但没有height，没有width，没有constraints，没有alignment，Container将约束从父级传递给子级，并调整自身大小以匹配子级。

另外margin和padding属性也会影响布局，decoration也会隐式的增加padding。

总体来说，Container会尝试对齐(alignment)，根据子widget的大小调整自身大小，采用width，height以及约束(constraints)，改变自身以适应父widget，并且要尽可能的小。

构造函数：

```
Container({Key key, AlignmentGeometry alignment, EdgeInsetsGeometry padding, Color color, Decoration decoration, Decoration foregroundDecoration, double width, double height, BoxConstraints constraints, EdgeInsetsGeometry margin, Matrix4 transform, Widget child })
```

参数：

* alignment → AlignmentGeometry，子widget的对齐方式；
* child → Widget，应该添加的子widget；
* constraints → BoxConstraints，为子widget添加的约束；
* decoration → Decoration，在子widget背后添加的装饰，例如边框等，不能与color同时使用；
* foregroundDecoration → Decoration，在子widget前添加装饰；
* margin → EdgeInsetsGeometry，子widget和decoration周围空出空间；
* padding → EdgeInsetsGeometry，decoration里面的空出的空间，如果有的话将子widget放到padding内；
* transform → Matrix4，在绘制容器前应用的转换矩阵；
* hashCode → int，此对象的哈希码，只读；
* key → Key，控制一个widget如何替换树中的另一个widget；
* runtimeType → Type，表示对象的运行时类型。

方法：

* build（BuildContext context） → Widget，描述此widget表示的用户界面部分；
* debugFillProperties(DiagnosticPropertiesBuilder properties) → void，添加与节点关联的其他属性；
* createElement() → StatelessElement，创建StatelessElement以管理该widget在树中的位置；
* debugDescribeChildren() → List<DiagnosticsNode>，返回描述此节点的子节点的DiagnosticsNode对象列表；
* noSuchMethod(Invocation invocation) → dynamic，访问不存在的方法或属性时调用；
* toDiagnosticsNode({String name, DiagnosticsTreeStyle style }) → DiagnosticsNode，返回调试工具和DiagnosticsNode.toStringDeep使用的对象的调试表示形式；
* toString({DiagnosticLevel minLevel: DiagnosticLevel.debug }) → String，返回此对象的字符串表示形式；
* toStringDeep({String prefixLineOne: '', String prefixOtherLines, DiagnosticLevel minLevel: DiagnosticLevel.debug }) → String，返回此节点及其后代的字符串表示形式；
* toStringShallow({String joiner: ', ', DiagnosticLevel minLevel: DiagnosticLevel.debug }) → String，返回对象的单行详细描述；
* toStringShort() → String，该widget的简短文字描述

简单示例：

```
@override
Widget build(BuildContext context) {
	return Scaffold(
	  appBar: AppBar(
	    title: Text('Container'),
	  ),
	  body: Center(
	    child: Container(
	      margin: EdgeInsets.all(20.0), // 与父widget的边距为20
	      color: Colors.amber[600],
	      width: 500,// 当宽度超过屏幕宽度，看到的效果实际上是与屏幕保持20的边距
	      height: 500,
	    ),
	  ),
	);
}
```

设置约束并添加变换：

```
@override
Widget build(BuildContext context) {
	return Scaffold(
	  appBar: AppBar(
	    title: Text('Container'),
	  ),
	  body: Center(
	    child: Container(
	      constraints: BoxConstraints.expand(
	        height: Theme.of(context).textTheme.display1.fontSize * 1.1 + 200,
	      ),
	      padding: const EdgeInsets.all(8.0),
	      color: Colors.blue,
	      alignment: Alignment.center,
	      child: Text('Hello world', style: Theme.of(context).textTheme.display1.copyWith(color: Colors.white),),
	      transform: Matrix4.rotationZ(0.1),
	    ),
	  ),
	);
}
```

#### Padding

一个给其子widget添加指定的填充widget。

将布局约束传递给其子级时，Padding会按给定的填充缩小约束，从而导致子级以较小的大小进行布局。Padding然后根据其子widget的大小调整自身大小，通过填充物膨胀，有效地在子widget周围创造出空的空间。

Padding和Container.padding没有区别。Container不直接实现其属性。相反，Container 将许多更简单的小部件组合到一个方便的包中。Flutter中使用组合（而非继承）是构建widgets的主要机制。

构造函数：

```
Padding({Key key, @required EdgeInsetsGeometry padding, Widget child })
```

#### Center

将其子widget居中显示在自身内部的widget。

如果它的尺寸受到约束且[widthFactor](https://api.flutter.dev/flutter/widgets/Align/widthFactor.html){:target="_blank"}和[heightFactor](https://api.flutter.dev/flutter/widgets/Align/heightFactor.html){:target="_blank"}为null，则此widget将尽可能大。如果维度不受约束且相应的大小因子为null，则此widget将匹配其在该维度中的子项大小。如果大小因子不为null，则此widget的相应维度将是子维度和大小因子的乘积。例如，如果widthFactor为2.0，则此widget的宽度将始终是子widget宽度的两倍。

构造函数：

```
Center({Key key, double widthFactor, double heightFactor, Widget child })
```

参数：

* heightFactor → double，如果为非null，则将其高度设置为子widget高度乘以此系数；
* widthFactor → double，如果为非null，则将其宽度设置为子widget宽度乘以此系数；

#### Align

一个可以将其子widget对齐，并可以根据子widget的大小自动调整大小的widget。

如果它的尺寸受到约束且widthFactor和heightFactor为null，则此widget将尽可能大 。如果维度不受约束且相应的大小因子为null，则widget将匹配其在该维度中的子项大小。如果大小因子不为null，则此widget的相应维度将是子维度和大小因子的乘积。例如，如果widthFactor为2.0，则此widget的宽度将始终是子widget宽度的两倍。

构造函数：

```
Align({Key key, AlignmentGeometry alignment: Alignment.center, double widthFactor, double heightFactor, Widget child })
```

示例：

```
Center(
  child: Container(
    height: 120.0,
    width: 120.0,
    color: Colors.blue[50],
    child: Align(
      alignment: Alignment.topRight,
      child: FlutterLogo(
        size: 60,
      ),
    ),
  ),
)
```

#### FittedBox

按自己的大小调整其子widget的大小和位置的widget。

#### AspectRatio

一个试图将子widget的大小指定为某个特定的长宽比的widget。

#### ConstrainedBox

对其子项施加附加约束的widget。

#### Baseline

根据子项的基线对它们的位置进行定位的widget。

#### FractionallySizedBox

一个widget，它把它的子项放在可用空间的一小部分。

#### IntrinsicHeight

一个将它的子widget的高度调整其本身实际的高度的widget。

#### IntrinsicWidth

将它的子widget的宽度调整其本身实际的宽度的widget。

#### LimitedBox

一个当其自身不受约束时才限制其大小的盒子。

#### Offstage

一个布局widget，可以控制其子widget的显示和隐藏。

#### OverflowBox

对其子项施加不同约束的widget，它可能允许子项溢出父级。

#### SizedBox

一个特定大小的盒子。这个widget强制它的孩子有一个特定的宽度和高度。如果宽度或高度为NULL，则此widget将调整自身大小以匹配该维度中的孩子的大小。

#### SizedOverflowBox

一个特定大小的widget，但是会将它的原始约束传递给它的孩子，它可能会溢出。

#### Transform

在绘制子widget之前应用转换的widget。

#### CustomSingleChildLayout

一个自定义的拥有单个子widget的布局widget

#### 本文[Demo](https://github.com/lingjye/Flutter-Learning/tree/master/helloworld){:target="_blank"}


