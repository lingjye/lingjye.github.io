---
layout: post
title: "使用Theos编写Tweak插件总结"
subtitle: ''
author: "lingjye"
header-style: 'text'
tags:
  - iOS
---

##### 设备

越狱手机一部

越狱流程：[点击这里](https://www.lingjye.com/2019/06/12/Reveal/){:target="_blank"}。

需要开启SSH通道，可以使用爱思助手打开，需要连接USB，默认IP：127.0.0.1, 端口1025，登录账号：root，密码:alpine。

![img1](https://raw.githubusercontent.com/lingjye/lingjye.github.io/master/img/reverse/01.png)

##### 安装Theos

1. 环境变量配置，编辑bash_profile:

	```
	vim .bash_profile
	```
	
	添加如下内容：
	
	```
	export THEOS=/opt/theos
	export PATH=/opt/theos/bin:$PATH
	```
	
	其作用就是添加环境变量，方便以后使用theos时不需要每次都加上它的路径。

2. 使用root权限克隆Theos，会直接克隆到磁盘根目录的 `/opt/theos`目录下：

	```
	sudo git clone https://github.com/theos/theos.git
	```
3. 使用 homebrew 安装 dkpg, ldid
	
	dpkg（Debian Packager）是Theos依赖的工具之一，主要用来制作deb, 保证能够分发的Cydia的存储目录中，Theos生成的安装包最终也是以.deb格式。
	
	ldid是用作给iOS可执行文件签名的工具，使越狱程序或者插件可以安装到越狱手机上。
	
	```
	brew install dpkg ldid
	```

4. 下载[libsubstrate.dylib](http://www.mediafire.com/?2upm53uzzj0488u){:target="_blank"}，并将它拷贝到 `/opt/theos/lib` 目录下：
	
	libsubstrate.dylib 的作用就是将动态库注入到第三方项目中去。
	
	百度云链接: [https://pan.baidu.com/s/1fmAosfGSS9qE8ZHWTiopSQ](https://pan.baidu.com/s/1fmAosfGSS9qE8ZHWTiopSQ){:target="_blank"} 提取码: r3an

##### 创建项目
	
在命令行输入以下命令，启用模板：
	
```
nic.pl
```
	
然后会有12个模板可供选择，我们选择 `iphone/tweak', 然后依次输入:

1. 项目名称：Project Name (required): MyTeak 
2. 包名，即iOS中的bundleID：Package Name [com.yourcompany.myteak]: com.lingjye.myteak 
3. 作者：Author/Maintainer Name [txooo]: lingjye
4. 需要Hook的应用的bundleID，这里如果是要hook其他APP，请更改为其他APP的bundleID：[iphone/tweak] MobileSubstrate Bundle filter [com.apple.springboard]: com.apple.springboard
5. 依赖的应用，这里可以不用输入直接回车即可，默认是SpringBoard，表示tweak安装后，重启SpringBoard：[iphone/tweak] List of applications to terminate upon installation (space-separated, '-' for none) [SpringBoard]:

	![img2](https://raw.githubusercontent.com/lingjye/lingjye.github.io/master/img/reverse/02.png)
	
然后会在用户目录下(`/Users/YourMacName/myteak`)生成刚才创建的项目，例如：myteak

另外可以点击[这里下载](https://github.com/DHowett/theos-nic-templates/archive/master.zip){:target="_blank"}另外5个模板。然后把5个模板都拷贝到 `/opt/theos/templates/ios` 目录下。

也可以点击[查看如何自定义模板](http://iphonedevwiki.net/index.php/NIC#How_to_set_default_values){:target="_blank"}。

**文件介绍**

默认生成的项目下，会有四个文件：

1. control文件，项目信息描述文件，包含包名，项目名称，版本信息，作者信息等。
2. Makefile文件，用于指定工程用到的文件，例如接下来需要编写的 Tweak.xm 文件，这里我们需要修改内容：

	```
	export SDKVERSION = 10.2.1
	export ARCHS = armv7 arm64
	export TARGET = iphone:clang:latest:9.0
	export THEOS_DEVICE_IP = 127.0.0.1 -p 1025
	
	include $(THEOS)/makefiles/common.mk
	
	TWEAK_NAME = TweakDemo
	TweakDemo_FILES = Tweak.xm
	TweakDemo_FRAMEWORKS = UIKit CoreGraphics
	
	include $(THEOS_MAKE_PATH)/tweak.mk
	
	after-install::
		install.exec "killall -9 SpringBoard"
	```
	
	`export SDKVERSION = 10.2.1` 用于指定Xcode版本号；<br/>
	`export ARCHS = armv7 arm64` 用于指定CPU类型；<br/>
	`export TARGET = iphone:clang:latest:9.0` 用于指定支持的iOS系统版本<br/>
	`export THEOS_DEVICE_IP = 127.0.0.1 -p 1025` 用于指定需要安装的真机IP地址以及端口<br/>
	`TweakDemo_FRAMEWORKS` 用于指定依赖的系统库
	
3. Tweak.xm 这是需要我们编写的 Tweak 文件，用来执行 Hook 操作。
4. xxxx.plist 这个文件用于设置 Tweak 插件的作用范围，用来指定作用于那个 APP 的 BundleID。默认：com.apple.springboard, 可以替换为你公司的 APP BundleID 试一下。

##### theos逆向指令

Tweak.xm 中涉及到 [Logos 语法](http://iphonedevwiki.net/index.php/Logos){:target="_blank"}，其中的几个命令如下：

1. 块级别

* 	1.1. %group： 使用名称Groupname开始一个钩子组（用于条件初始化或代码组织）。所有未分组的钩子都在隐式的“_ungrouped”组中。不能包含另一个 %group，以 %end 结尾。<br/>
* 	1.2. %hook：指定需要hook的class，必须以%end结尾,一个group可以包含 %hook。<br/>
* 		1.2.1. %new: 在%hook内部使用，给一个现有class添加新函数，功能与class_addMethod相同。<br/>
* 	1.3. %subclass：该类在运行时创建并使用方法填充。不支持ivars（使用关联对象）。％new需要一个在父类中不存在的方法。要实例化新类的对象，可以使用％c运算符。<br/>
* 	1.4. %end<br/>
	
2. 	顶级
	
* 	2.1. %config：设置配置标志<br/>
* 	2.2. %hookf：为名为symbolName的函数生成函数钩子。如果名称作为字符串传递，则将动态查找该函数。<br/>
* 	2.3. %ctor：tweak的构造函数,完成初始化工作；如果不显示定义，Theos会自动生成一个%ctor，并在其中调用%init(_ungrouped)。%ctor一般可以用来初始化%group，以及进行MSHookFunction等操作。<br/>
* 	2.4. %dtor：与 %ctor相反，析构函数。<br/>
	
3. 	功能级别
	
* 	3.1. %init：用于初始化某个 _%group ,必须在 %hook _或_ %ctor 内调用;如果带参数,则初始化指定的 group,如果不带参数,则初始化 __ungrouped。<br/>
* 	3.2. %c：等同于 objc_getClass 或 NSClassFromString,即动态获取一个类的定义,在 %hook 或 %ctor 内使用。<br/>
* 	3.3. %orig：在%hook内部使用，执行被 hook 的方法的原始代码。<br/>
* 	3.4. %log：在 %hook 内部使用，将函数的类名、参数等信息写入 syslog，可以用 ％log(@”“, @”“,...)的格式追加其他打印信息。<br/>

	参考： [Logos 语法](http://iphonedevwiki.net/index.php/Logos){:target="_blank"}

##### 编译、打包、安装、卸载

首先需要 cd 到项目路径下：

```
cd /Users/txooo/myteak
```

**编译**

```
make 
```

如果遇到错误，请点击<a href="#error">这里</a>

**打包**

```
make package
```

之后会在项目下看到生成了一个 `packages` 文件夹，它下面存放的就是 .deb 格式的Tweak安装包。

**安装**

```
make install
```

**卸载**

需要使用SSH访问设备的文件系统，然后执行以下命令：

```
// 此处是要卸载的bundleID
dpkg -r com.domain.xxxx
```

可以前往 Cydia->已安装->最近查看安装是否成功，或者使用 ssh 命令或工具前往设备的 `Library/MobileSubstrate/DynamicLibraries/` 目录下查看是否有刚才安装的插件，后缀为 .dylib和.plist。

注意，请在Makefile文件修改自己设备的真实IP

##### 本文[Demo](https://github.com/lingjye/TweakDemo){:target="_blank"}

#### <a name="error">错误</a>

1. 错误： git submodule update --init --recursice

	执行：
	
	```
	cd /opt/theos
	```
	
	然后执行
	
	```
	git submodule update --init --recursice
```

2. 错误：include path for stdlibc++ headers not found; pass '-stdlib

	修改Makefile, 在首行加入：
	
	```
	export SDKVERSION = 10.2.1
	export ARCHS = armv7 arm64
	export TARGET = iphone:clang:latest:9.0
	export THEOS_DEVICE_IP = 127.0.0.1 -p 1025
```

3. 错误：error: obsolete compression type 'lzma'; use xz instead

	修改`opt/theos/makefiles/package/deb.mk`中的第六行：
	
	```
	THEOSPLATFORM_DPKG_DEB_COMPRESSION ?= lzma
	```
	
	改为：
	
	```
	THEOSPLATFORM_DPKG_DEB_COMPRESSION ?= xz
	```