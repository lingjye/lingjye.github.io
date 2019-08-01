---
layout: post
title: "Flutter基础——布局二"
subtitle: '拥有多个子元素的布局widget'
author: "lingjye"
header-style: text
tags:
  - Flutter
---

#### Row

在水平方向上排列子widget的列表。

使用Row的来包括的Widget不会滚动，当子widget多于可用空间中能容纳的数目时报错。要使子项扩展以填充可用的水平空间，请将子项包装在Expanded小部件中。

如果只有一个孩子优先考虑Align或Center。

**构造函数**

```
Row({Key key, MainAxisAlignment mainAxisAlignment: MainAxisAlignment.start, MainAxisSize mainAxisSize: MainAxisSize.max, CrossAxisAlignment crossAxisAlignment: CrossAxisAlignment.center, TextDirection textDirection, VerticalDirection verticalDirection: VerticalDirection.down, TextBaseline textBaseline, List<Widget> children: const [] })
```

**参数**

* children → List<Widget>，树中此widget下方的子widget。
* crossAxisAlignment → CrossAxisAlignment，如何将孩子放在此widget上的对齐方式。
* direction → Axis，用作主轴的方向。
* mainAxisAlignment → MainAxisAlignment，如何将孩子放到主轴上。
* mainAxisSize → MainAxisSize，主轴占用多少空间。
* textBaseline → TextBaseline，如果根据基线对齐项目，则使用哪个基线。
* textDirection → TextDirection，确定水平放置孩子以及如何解释start和end水平方向的顺序
* verticalDirection → VerticalDirection，确定垂直放置孩子以及如何解释start和end垂直方向的顺序

**示例**

```
Row(
  children: <Widget>[
    const FlutterLogo(),
    const Text('Hello World'),
    const Icon(Icons.sentiment_very_satisfied),
  ],
  // crossAxisAlignment: CrossAxisAlignment.center,
  // crossAxisAlignment: CrossAxisAlignment.baseline,
  // textBaseline: TextBaseline.ideographic,
  // textDirection: TextDirection.rtl,
)
```

**Expanded**

用于展开Row，Column或Flex 的子项，以便子项填充可用空间。

它使得Row，Column，或Flex的子widget沿着主轴线的可用空间（例如，水平地为一个行或垂直于一列）填充。如果扩展了多个子节点，则根据弹性因子将可用空间划分为多个子节点。

一个Expanded widget必须是一个Row，Column，或Flex的后代，并从所述路径扩展插件到其封闭的Row，Column，或Flex，必须只包含StatelessWidgets或StatefulWidgets（而不是其他类型的widget，像RenderObjectWidgets）。

Expanded会先让未使用Expanded的子widget先布局，然后让使用了Expanded的子widget填充剩下的空白空间，如果所有子widget全部都使用了Expanded，则会平均父widget主布局方向的空间。

示例：

```
Row(
  children: <Widget>[
    const FlutterLogo(),
    const Expanded(
      child: Text('Flutter\'s hot reload helps you quickly and easily experiment, build UIs, add features, and fix bug faster. Experience sub-second reload times, without losing state, on emulators, simulators, and hardware for iOS and Android.'),
    ),
    const Icon(Icons.sentiment_very_satisfied),
  ],
)
```

#### Column

在垂直方向上排列子widget的列表。

同Row一样，可使用Expanded来让子widget填充垂直方向上的空间，子widget不能超过父widget的可用空间。如果只有一个孩子优先考虑Align或Center。

列的布局分为六个步骤：

1. 使用无界垂直约束和传入的水平约束将每个孩子布局为无或零伸缩因子（例如，未展开的因子 ）。如果crossAxisAlignment是 CrossAxisAlignment.stretch，则使用与传入的最大宽度匹配的紧密水平约束。
2. 剩余垂直空间根据伸缩因子，除以具有非零伸缩因子（例如，扩展的那些）的子节点。例如，flex因子为2.0的孩子将获得两倍于垂直空间的孩子，其伸缩系数为1.0。
3. 使用与步骤1中相同的水平约束来布局每个剩余子项，但不使用无界垂直约束，而是使用基于步骤2中分配的空间量的垂直约束。子项的FlexFit.tight属性是Flexible.fit，它们是给定严格约束（即强制填充分配的空间），具有FlexFit.loose的Flexible.fit属性的子项被赋予松散约束（即不强制填充分配的空间）。
4. Column的宽度是孩子们（这将始终满足传入水平约束）的最大宽度。
5. Column的高度由mainAxisSize属性决定。如果mainAxisSize属性是MainAxisSize.max，则Column的高度是传入约束的最大高度。如果mainAxisSize属性是MainAxisSize.min，则Column的高度是子项的高度之和（受传入约束限制）。
6. 根据mainAxisAlignment和crossAxisAlignment确定每个子项的位置。例如，如果 mainAxisAlignment是MainAxisAlignment.spaceBetween，则任何尚未分配给子项的垂直空间将均匀分配并放置在子项之间。

**构造函数**

```
Column({Key key, MainAxisAlignment mainAxisAlignment: MainAxisAlignment.start, MainAxisSize mainAxisSize: MainAxisSize.max, CrossAxisAlignment crossAxisAlignment: CrossAxisAlignment.center, TextDirection textDirection, VerticalDirection verticalDirection: VerticalDirection.down, TextBaseline textBaseline, List<Widget> children: const [] })
```

**示例**

```
Column(
  children: <Widget>[
    Text('We move under cover and we move as one'),
    Text('Through the night, we have one shot to live another day'),
    Text('We cannot let a stray gunshot give us away'),
    Text('We will fight up close, seize the moment and stay in it'),
    Text('It’s either that or meet the business end of a bayonetIt’s either that or meet the business end of a bayonet'),
    Text('The code word is ‘Rochambeau,’ dig me?'),
  ],
  // 左对齐
  crossAxisAlignment: CrossAxisAlignment.start,
  mainAxisSize: MainAxisSize.max,
);
```

#### Stack

可以允许其子widget简单的堆叠在一起。即z轴方向上排列子widget的列表，它们会进行覆盖。

Stack的每个子节点都已定位或未定位。定位子项是包含在具有至少一个非null属性的 Positioned widget中的子项。Stack大小自身包含所有未定位的子项，这些子项根据对齐方式定位（默认为左上角,从左到右和右上角从右到左）。然后根据它们的顶部，右侧，底部和左侧属性相对于Stack放置已定位的子项。

当第一个孩子在底部时，Stack按顺序绘制其子项。如果要更改子项绘制的顺序，可以使用新顺序中的子项重建Stack。如果重新排序，需要考虑为子项指定非空键。这些键将使框架将子项的基础对象移动到新位置，而不是在新位置重新创建它们。

如果要以特定模式放置多个子项，或者如果要创建自定义布局管理器，则可能需要使用 CustomMultiChildLayout。特别是，使用Stack时无法根据大小或Stack自身大小来定位子项。

**构造函数**

```
Stack({Key key, AlignmentGeometry alignment: AlignmentDirectional.topStart, TextDirection textDirection, StackFit fit: StackFit.loose, Overflow overflow: Overflow.clip, List<Widget> children: const [] })
```

**参数：**

* alignment → AlignmentGeometry，子项对齐方式。
* fit → StackFit，如何调整Stack中未定位的子项大小。
* overflow → Overflow，是否应该裁剪掉溢出的子widget。
* textDirection → TextDirection，用于解决文本对齐方向。

**示例**

```
Stack(
  children: <Widget>[
    Container(
      width: 100,
      height: 100,
      color: Colors.blue,
    ),
    Container(
      width: 90,
      height: 90,
      color: Colors.yellow,
    ),
    Container(
      width: 80,
      height: 80,
      color: Colors.red,
    ),
  ],
  alignment: Alignment.topRight,
);
```

#### IndexedStack

从一个子widget列表中显示单个孩子的Stack。

#### Flow

一个实现流式布局算法的widget。

通过委托的FlowDelegate.getSize函数，使得流容器的大小独立于子容器。然后根据FlowDelegate.getConstraintsForChild函数的约束，单独调整子项的大小 。

Flow不是在布局期间定位子项，而是使用FlowDelegate.paintChildren函数中的矩阵在绘制阶段使用变换矩阵定位子项。通过简单地重新绘制Flow就可以有效地重新定位孩子，这可以发生在没有孩子再次布局的情况下（与Stack相反，在布局期间进行尺寸调整和定位）。

触发重绘Flow的最有效方法是向FlowDelegate的构造函数提供动画。Flow将监听此动画并在动画启动时重新绘制，从而避免管道的构建和布局。

**构造函数**

```
Flow({Key key, @required FlowDelegate delegate, List<Widget> children: const [] })
```

**参数**

* delegate → FlowDelegate，控制子项转换矩阵的委托。
* children → List<Widget>，子widgets。

**示例**

```
class _FlowSampleAppPageState extends State<FlowSampleAppPage> with SingleTickerProviderStateMixin {

  AnimationController menuAnimation;
  IconData lastTapped = Icons.notifications;
  final List<IconData> menuItems = <IconData>[
    Icons.home,
    Icons.new_releases,
    Icons.notifications,
    Icons.settings,
    Icons.menu,
  ];

  @override
  void initState() {
    // TODO: implement initState
    super.initState();
    menuAnimation = AnimationController(
      duration: const Duration(microseconds: 300),
      vsync: this
    );
  }

  Widget flowMenuItem(IconData icon) {
    final double buttonDiameter = MediaQuery.of(context).size.width / menuItems.length;
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8.0),
      child: RawMaterialButton(
        fillColor: lastTapped == icon ? Colors.amber[700] : Colors.blue,
        splashColor: Colors.amber[100],
        shape: CircleBorder(),
        constraints: BoxConstraints.tight(Size(buttonDiameter, buttonDiameter)),
        onPressed: (){
          updateMenu(icon);
          menuAnimation.status == AnimationStatus.completed ? menuAnimation.reverse() : menuAnimation.forward();
        },
        child: Icon(
          icon,
          color: Colors.white,
          size: 45.0,
        ),
      ),
    );
  }

  void updateMenu(IconData icon) {
    if (icon != Icons.menu) {
      setState(() {
        lastTapped = icon;
      });
    }
  }

  getFlow() {
    return Container(
      child: Flow(
        delegate: FlowMenuDelegate(menuAnimation: menuAnimation),
        children: menuItems.map<Widget>((IconData icon) => flowMenuItem(icon)).toList(),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Flow'),
      ),
      body: getFlow(),
    );
  }
}

class FlowMenuDelegate extends FlowDelegate {
  FlowMenuDelegate({this.menuAnimation}) : super(repaint:menuAnimation);
  final Animation<double> menuAnimation;

  @override
  bool shouldRepaint(FlowMenuDelegate oldDelegate) {
    // TODO: implement shouldRepaint
    return menuAnimation != oldDelegate.menuAnimation;
  }

  @override
  void paintChildren(FlowPaintingContext context) {
    // TODO: implement paintChildren
    double dx = 0.0;
    for (var i = 0; i < context.childCount; ++i) {
      dx = context.getChildSize(i).width * i;
      context.paintChild(
        i,
        transform: Matrix4.translationValues(
          dx * menuAnimation.value,
          20,
          0
        ),
      );
    }
  }
}
```

#### Table

为其子widget使用表格布局算法的widget。

行数根据其内容垂直排列，列宽根据columnWidths属性控制，适用于多行多列情况，每一行的列数必须相等。

**构造函数**

```
Table({Key key, List<TableRow> children: const [], Map<int, TableColumnWidth> columnWidths, TableColumnWidth defaultColumnWidth: const FlexColumnWidth(1.0), TextDirection textDirection, TableBorder border, TableCellVerticalAlignment defaultVerticalAlignment: TableCellVerticalAlignment.top, TextBaseline textBaseline })
```

**参数**

* border → TableBorder，绘制表格的边界和内部分区时使用的样式。
* children → List<TableRow>，表的行widgets
* columnWidths → Map<int, TableColumnWidth>，如何确定此表的列的水平范围。
* defaultColumnWidth → TableColumnWidth，如何确定没有显式大小调整算法的列的宽度。
* defaultVerticalAlignment → TableCellVerticalAlignment，未明确指定垂直对齐的单元格如何垂直对齐。
* textBaseline → TextBaseline，使用TableCellVerticalAlignment.baseline对齐行时使用的文本基线。
* textDirection → TextDirection，列的排序方向。

示例

```
class TableSampleAppPage extends StatelessWidget {
  const TableSampleAppPage({Key key}) : super(key: key);

  getTable() {
    return Table(
      children: <TableRow>[
        TableRow(
          children: <Widget>[
            TableCell(
              child: Center(
                child: Text('Index 1'),
              ),
            ),
            TableCell(
              child: Text('Index 2'),
            ),
             TableCell(
              child: Text('Index 3'),
            ),
          ]
        ),
        TableRow(
          children: <Widget>[
            TableCell(
              child: Text('index 1', textAlign: TextAlign.center,),
            ),
            Text('index 2'),
             TableCell(
              child: Text('Index 3'),
            ),
          ]
        )
        
      ],
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Table'),
      ),
      body: getTable()
    );
  }
}
```

#### Wrap

可以在水平或垂直方向多行显示其子widget。

Wrap勾画出每个孩子并尝试给出相邻于主轴线的前一个孩子及方向，空出之间的间隔。如果没有足够的空间来容纳孩子，Wrap会在横轴上的现有子项旁生成新的run（行或列）。

所有子项将根据主轴中的对齐方式并根据横轴中的crossAxisAlignment进行定位。

然后根据runSpacing和runAlignment将run本身定位在十字轴上 。

**构造函数**

```
Wrap({Key key, Axis direction: Axis.horizontal, WrapAlignment alignment: WrapAlignment.start, double spacing: 0.0, WrapAlignment runAlignment: WrapAlignment.start, double runSpacing: 0.0, WrapCrossAlignment crossAxisAlignment: WrapCrossAlignment.start, TextDirection textDirection, VerticalDirection verticalDirection: VerticalDirection.down, List<Widget> children: const [] })
```

**属性**

* alignment → WrapAlignment，主轴方向上的对齐方式。
* crossAxisAlignment → WrapCrossAlignment，run中孩子在横轴上相对于彼此的对齐方式。
* direction → Axis，主轴方向。
* runAlignment → WrapAlignment，run的对齐方式。run可以理解为新的行或者列，如果是水平方向布局的话，run可以理解为新的一行。
* runSpacing → double，横轴上run的间距
* spacing → double，主轴方向上的间距。
* textDirection → TextDirection，文本方向。
* verticalDirection → VerticalDirection，children摆放顺序。
* children → List<Widget>，子widgets。

示例

```
class _WrapSampleAppPageState extends State<WrapSampleAppPage> {

  getWrap() {
    return Wrap(
      spacing: 8.0,
      runSpacing: 10.0,
      children: <Widget>[
        Chip(
          avatar: CircleAvatar(
            backgroundColor: Colors.blue.shade900,
            child: Text('A'),
          ),
          label: Text('AAA'),
        ),
        Chip(
          avatar: CircleAvatar(
            backgroundColor: Colors.blue.shade900,
            child: Text('B'),
          ),
          label: Text('BBB'),
        ),
        Chip(
          avatar: CircleAvatar(
            backgroundColor: Colors.blue.shade900,
            child: Text('C'),
          ),
          label: Text('CCC'),
        ),
        Chip(
          avatar: CircleAvatar(
            backgroundColor: Colors.blue.shade900,
            child: Text('D'),
          ),
          label: Text('DDD'),
        ),
      ],
    );
  }
}
```

#### ListBody

一个widget，它沿着一个给定的轴，顺序排列它的子元素。

#### ListView

可滚动的列表控件。ListView是最常用的滚动widget，它在滚动方向上一个接一个地显示它的孩子。在纵轴上，孩子们被要求填充ListView。

#### CustomMultiChildLayout

使用一个委托来对多个孩子进行设置大小和定位的小部件。

#### LayoutBuilder

构建一个可以依赖父窗口大小的widget树。

#### 本文[Demo](https://github.com/lingjye/Flutter-Learning/tree/master/helloworld){:target="_blank"}