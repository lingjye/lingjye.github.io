---
layout: post
title: "iOS重签名流程"
subtitle: ''
author: "lingjye"
header-style: text
tags:
  - iOS
---

### 一、准备

1. 需要签名的ipa
2. 重签名使用到的企业签(.p12)证书，以及(.mobileprovision)描述文件

### 二、签名

1.	前往ipa所在目录：
	
	```
	ipaDirectory
		--xxx.ipa
	```
	
	```
	cd  ipaDirectory
	```

2. 修改描述文件名(.mobileprovision)为 embedded.mobileprovision

3. 通过 embedded.mobileprovision 生成 embedded.plist
	
	```
	security cms -D -i embedded.mobileprovision > embedded.plist
	```
4. 通过 embedded.plist 生成 entitlements.plist 文件

	```
	/usr/libexec/PlistBuddy -x -c 'Print:Entitlements'  embedded.plist > entitlements.plist
	```
5. 解压 .ipa 文件，可手动解压，也可以命令行解压

	```
	unzip xxx.ipa
	```
	
	解压后在当前目录下可看到生成一个 Payload 的文件夹
	
	```
	Payload
		--xxx.app
	```
	
	其中的 xxx.app 可以右键显示包内容
	
6. 找出并删除 xxx.app 已生成的所有签名文件，即 _CodeSignature 文件夹。这里的 _CodeSignature 不单指 xxx.app/_CodeSignature，还包括动态库以及工程创建的扩展组件（PlugIns）下的签名文件，即：
	
	```
	Payload/xxx.app/_CodeSignature
	
	Payload/xxx.app/PlugIns/xxxExtension-1.appex/_CodeSignature
	···
	Payload/xxx.app/PlugIns/xxxExtension-n.appex/_CodeSignature
	
	Payload/xxx.app/Frameworks/xxxSDK-1.framework/_CodeSignature
	···
	Payload/xxx.app/Frameworks/xxxSDK-n.framework/_CodeSignature
	
	```
	
	可手动前往 xxx.app 下找到后删除，也可命令行删除：
	
	```
	rm -rf Payload/xxx.app/_CodeSignature
	···
	rm -rf Payload/xxx.app/Frameworks/xxxSDK-n.framework/_CodeSignature
	
	```

7. 删除成功以后，需要我们重新签名，接下来使用 codesign 命令依次对动态库进行签名，该过程需要用到证书，由于证书名称比较长，签名前可以先复制企业证书的证书名称（如：iPhone Distribution：XXX Network Technology Co., Ltd.）：
	 
	```
	codesign -f -s "iPhone Distribution：XXX Network Technology Co., Ltd." Payload/xxx.app/Frameworks/xxxSDK.framework
	```

8. 接下来对扩展组件（PlugIns）签名，首先我们替换 `Payload/xxx.app/PlugIns/xxxExtension-1.appex/embedded.mobileprovision` 描述文件为第2步重命名的 `embedded.mobileprovision` 
	
	```
	cp embedded.mobileprovision Payload/xxx.app/PlugIns/xxxExtension-1.appex/embedded.mobileprovision
	```
	
	然后使用 codesign 命令进行签名：
	
	```
	codesign -f -s "iPhone Distribution：XXX Network Technology Co., Ltd." Payload/xxx.app/PlugIns/xxxExtension.appex
	```
	
	如果有多个扩展组件，重复该过程为所以组件签名。
	
9. 最后对 xxx.app 进行重签名，这里同上一步一样， 需要先替换 `Payload/xxx.app/embedded.mobileprovision` 描述文件为第2步重命名的 `embedded.mobileprovision` 

	```
	cp embedded.mobileprovision Payload/xxx.app/embedded.mobileprovision
	```
	
	重新签名：
	
	```
	codesign -f -s "iPhone Distribution：XXX Network Technology Co., Ltd." --no-strict --entitlements=entitlements.plist  Payload/xxx.app
	```

	等待执行完成即可。
	
10. 查看重签名信息：

	```
	codesign -vv -d Payload/xxx.app
	```
	
	信息如下：
	
	```
	Identifier=com.xxx.bundleid
	Format=app bundle with Mach-O thin (arm64)
	CodeDirectory v=20400 size=175793 flags=0x0(none) hashes=5485+5 location=embedded
	Signature size=4831
	Authority=iPhone Distribution: XXX Network Technology Co., Ltd.
	Authority=Apple Worldwide Developer Relations Certification Authority
	Authority=Apple Root CA
	Signed Time=Jan 15, 2020 at 4:15:04 PM
	Info.plist entries=39
	TeamIdentifier=证书TeamID
	Sealed Resources version=2 rules=10 files=246
	Internal requirements count=1 size=196
	```

11. 将签名后的 .app 所在的 Payload 目录压缩为 .ipa

	```
	zip -r xxx.ipa Payload
	```

签名工具:[ResignForiOS](https://github.com/HanProjectCoder/ResignForiOS)

### 有关签名的原理可前往[《iOS App 签名的原理》](http://blog.cnbang.net/tech/3386/){target="_blank"}