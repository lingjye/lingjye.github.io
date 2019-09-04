---
layout: post
title: "CocoaPods组件化过程中OC与Swift混编问题"
subtitle: ''
author: "lingjye"
header-style: text
tags:
  - CocoaPods
---

#### PCH

pod中饭pch尽量不要使用，每次pod install会重置，事实上其他开源三方库也都不怎么用pch。

可以将需要导入的第三方头文件单独放到一个.h文件中，然后在pod库中的导入这个.h文件。

* 注意，需要将该.h 设置私有，尤其是当pod库中含有OC和Swift混编时，该操作极其重要，私有该.h文件示例：

```
s.private_header_files = 'MixSDK/Classes/MSHelperHeader.h'
```

否则，你可能遇到比如`Include of non-modular header inside framework module `的错误，该错误就是你在头文件中引用了其他库的头文件，这是不推荐的，建议放到.m中去导入。

此处使用PROJECT >Buld Setting 中设置 Allow Non-modular Includes In Framework Modules 为YES是无济于事的。

可以将使用的第三方头文件在使用的.m文件中导入，但是显得极其臃肿，甚至需要为pod库中的每一个.m都导入这些第三方库的.h文件，可以想象以下，这是什么样的感受。

当然，也可以在YourPodName-umbrella.h（跟桥接文件类似）中将单独引用其他第三方库的.h文件删除，这一步也不推荐，不仅会有警告`<module-includes>:1:1: Umbrella header for module 'YourPodName' does not include xxxx`,还会在执行pod install后会重置，慎用。

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

如果在pod库中使用了Swift，那么在主工程中一定要新建一个Swift文件，空白的也可以。否则编译器会报错：

```
ld: warning: Could not find auto-linked library 'swiftCoreGraphics'
ld: warning: Could not find auto-linked library 'swiftFoundation'
ld: warning: Could not find auto-linked library 'swiftMetal'
ld: warning: Could not find auto-linked library 'swiftDarwin'
ld: warning: Could not find auto-linked library 'swiftUIKit'
ld: warning: Could not find auto-linked library 'swiftObjectiveC'
ld: warning: Could not find auto-linked library 'swiftCoreFoundation'
ld: warning: Could not find auto-linked library 'swiftDispatch'
ld: warning: Could not find auto-linked library 'swiftCoreImage'
ld: warning: Could not find auto-linked library 'swiftQuartzCore'
ld: warning: Could not find auto-linked library 'swiftCore'
ld: warning: Could not find auto-linked library 'swiftSwiftOnoneSupport'
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

**指定Swift版本**

当pod库中含有Swift中需要在Podspec文件中指定Swift版本，如下：

```
#s.swift_version = '4.2'
s.swift_version = '5.0'
```

#### 配置忽略文件

使用`pod lib create YourPodName`命令生成的pod库，会自动生成`.gitignore`文件，默认包含如下内容：

```
# OS X
.DS_Store

# Xcode
build/
*.pbxuser
!default.pbxuser
*.mode1v3
!default.mode1v3
*.mode2v3
!default.mode2v3
*.perspectivev3
!default.perspectivev3
xcuserdata/
profile
*.xccheckout
*.moved-aside
DerivedData
*.hmap
*.ipa

# Bundler
.bundle

# Add this line if you want to avoid checking in source code from Carthage dependencies.
# Carthage/Checkouts

Carthage/Build

# We recommend against adding the Pods directory to your .gitignore. However
# you should judge for yourself, the pros and cons are mentioned at:
# https://guides.cocoapods.org/using/using-cocoapods.html#should-i-ignore-the-pods-directory-in-source-control
# 
# Note: if you ignore the Pods directory, make sure to uncomment
# `pod install` in .travis.yml
#
# Pods/
```

注意查看上边忽略的文件是否有和本地pod库中冲突的，然后将其删除或注释，笔者就遇到某一个组件中包含Profile文件夹，其与上边的profile同名（注意，不区分大小写），导致的错误：

```
- ERROR | [YourPodName/Profile/xxxx, and more...] file patterns: The `source_files` pattern did not match any file.
```

这是在提交代码至git仓库时Profile下的所有文件都被忽略了，造成pod repo push时远程验证（验证时会从git仓库去取，如果不存在就会报如上错误）Podspec文件失败造成的。

**[查看Demo](https://github.com/lingjye/iOS-Learning/tree/master/MixSDK){:target="_blank"}**



