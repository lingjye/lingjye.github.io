---
layout: post
title: "使用CocoaPods拆分项目时的pch问题"
subtitle: ''
author: "lingjye"
header-style: text
tags:
  - CocoaPods
---

pod中饭pch尽量不要使用，每次pod install会重置，事实上其他开源三方库也都不怎么用pch。

可以将需要导入的第三方头文件单独放到一个.h文件中，然后在pod库中的导入这个.h文件。

* 注意，需要将该.h 设置私有，尤其是当pod库中含有OC和Swift混编时，该操作极其重要，私有该.h文件示例：

```
s.private_header_files = 'MixSDK/Classes/MSHelperHeader.h'
```

否则，你可能遇到比如`Include of non-modular header inside framework module `的错误，该错误就是你在头文件中引用了其他库的头文件，这是不推荐的，建议放到.m中去导入。

此处使用PROJECT >Buld Setting 中设置 Allow Non-modular Includes In Framework Modules 为YES是无济于事的。

可以将使用的第三方头文件在使用的.m文件中导入，但是显得极其臃肿，甚至需要为pod库中的每一个.m都导入这些第三方库的.h文件，可以想象以下，这是什么样的感受。

当然，也可以在YourPodName-umbrella.h中将单独引用其他第三方库的.h文件删除，这一步也不推荐，不仅会有警告`<module-includes>:1:1: Umbrella header for module 'YourPodName' does not include xxxx`,还会在执行pod install后会重置，慎用。

设置私有后执行下`pod install`可以看到xxx--umbrella.h中已经没有专门导入第三方库的.h文件了。

![umbrella.h](https://raw.githubusercontent.com/lingjye/lingjye.github.io/master/img/pods/umbrella-h.png)

[Podspec文档](https://guides.cocoapods.org/syntax/podspec.html)

#### OC与Swift混编

在pod库中，OC引用swift文件时，需要导入：

```
#import <MixSDK/MixSDK-Swift.h>
```

主工程中的OC文件引用swift文件时使用`#import "YourTargetName-Swift.h"`来引用，在XCode 10.2.1，Swift 5.0中使用`@import YourPodName;`，例如：

```
#import "MixSDK_Example-Swift.h"
```

Xcode 10.2.1，swift 5.0使用：

```
@import MixSDK;
```

而swift引用OC文件时，不需要使用桥接文件，pod会自动生成YourPodName-umbrella.h直接使用即可，例如：

```
import UIKit

open class MSModel: NSObject {
    @objc open class func show() {
        MSAnimal.show();
    }
}
```



