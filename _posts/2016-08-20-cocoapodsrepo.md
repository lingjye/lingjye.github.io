---
layout: post
title: "使用 Cocoapod 创建私有库步骤"
author: "lingjye"
header-style: text
tags:
  - cocoapod
  - iOS
---

1. 在git远端创建私有库仓库

2. cd 到本地私有库创建的目录下

3. 添加到pod repo
	
	```pod repo add SpecName SpecURL```

4. 生成项目 

	```
	pod lib create SpecName
	```
	
5. 替换 Classes 目录下的文件

	刚创建时 Classes 目录下有一个 ReplaceMe.m, 删掉它, 并将需要的文件放到该目录下, 如果包含其他目录, 可在 podspec 中 s.source_files 指定路径
	
6. 修改 podspec 文件 

	```
	Pod::Spec.new do |s|
	s.name             = 'SpecName'
	s.version          = '0.0.1'
	s.summary          = 'A short description of SpecName.'
	
	# This description is used to generate tags and improve search results.
	#   * Think: What does it do? Why did you write it? What is the focus?
	#   * Try to keep it short, snappy and to the point.
	#   * Write the description between the DESC delimiters below.
	#   * Finally, don't worry about the indent, CocoaPods strips it!
	
	s.description      = <<-DESC
	TODO: Add long description of the pod here.
	                   DESC
	
	s.homepage         = 'https://github.com/lingjye/SpecName'
	# s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
	s.license          = { :type => 'MIT', :file => 'LICENSE' }
	s.author           = { 'lingjye' => 'github@lingjye.com' }
	s.source           = { :git => 'https://github.com/lingjye/SpecName.git', :tag => s.version.to_s }
	# s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'
	
	s.ios.deployment_target = '8.0'
	
	s.source_files = 'SpecName/Classes/**/*'
	  
	# s.resource_bundles = {
	#   'SpecName' => ['SpecName/Assets/*.png']
	# }
	
	# s.public_header_files = 'Pod/Classes/**/*.h'
	# s.frameworks = 'UIKit', 'MapKit'
	# s.dependency 'AFNetworking', '~> 2.3'
	end
	```
	
	s.name ：私有库名称<br/>
	s.version ：版本号，每一个版本对应一个tag<br/>
	s.summary : 简介<br/>
	s.description : 介绍<br/>
	s.homepage : 项目主页地址<br/>
	s.screenshots : 截图地址<br/>
	s.license : 许可协议<br/>
	s.author : 作者<br/>
	s.source : 项目地址<br/>
	s.social_media_url : 社交网址<br/>
	s.ios.deployment_target = '8.0' : 支持的pod最低版本<br/>
	s.source_files : 需要包含的源文件<br/>
	s.resources: 资源文件<br/>
	> “\*” 表示匹配所有文件<br/>
	> “**” 表示匹配所有子目录
	
	s.resource_bundles : 同上<br/>
	s.public_header_files : 公开头文件, 可不指定, 默认所有<br/>
	s.frameworks : 依赖的系统动态库<br/>
	s.dependency ：依赖库, 如果依赖多个分行写, 例如:<br/>
	
	```
	s.dependency 'AFNetworking', '~> 2.3'
	s.dependency 'ReactiveCocoa', '~> 2.5'
	``` 
	
	s.requires_arc : 是否支持ARC
	

7. 本地验证

	```
	pod lib lint SpecName.podspec --use-libraries --allow-warnings
	```
	
	如果使用了第三方库就添加 `--use-libraries`
	
	忽略警告 `--allow-warnings`

8. 上传本地代码到远程仓库

	```
	git add .
	git commit -am "desc" 
	git remote add origin https://github.com/lingjye/SpecName
	git pull origin master
	git push origin master
	```
	如果提示:
	
	```
	! [rejected]        master -> master (non-fast-forward)
	error: failed to push some refs to ''
	hint: Updates were rejected because the tip of your current branch is behind
	hint: its remote counterpart. Integrate the remote changes (e.g.
	hint: 'git pull ...') before pushing again.
	hint: See the 'Note about fast-forwards' in 'git push --help' for details.
	```
	
	出现这个问题是因为远程分支上存在本地分支中不存在的提交
	
	可以先fetch, 再merge, 也就是pull, 把运程分支上的提交合并到本地分支之后再push
	
	也可以使用 `git push origin master -f` 命令, 强行让本地分支覆盖远程分支

9. 设置标签

	```
	git tag -m "description" "version"
	git push --tags 
	```	
	version 与 podspec 文件中的保持一致


10. 提交pod

	```
	pod repo push SpecName SpecName.podspec  --use-libraries --allow-warnings
	```
	

	
	

