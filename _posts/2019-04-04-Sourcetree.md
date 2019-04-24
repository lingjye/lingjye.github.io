---
layout: post
title: "Mac环境下跳过SourceTree登录"
subtitle: ''
author: "lingjye"
header-style: text
tags:
  - git
---

之前一直使用的SourceTree版本是2.0.5.2版本，只需要注册license, 升级后打开一直卡在登录页，国内的话一般是无法直接登录的，翻墙除外，可以使用Google账号登录，且该步骤按正常流程无法跳过。

![atlassian](https://raw.githubusercontent.com/lingjye/lingjye.github.io/master/img/sourcetree/atlassian.png)

接下来新建一个Finder窗口，找到应用程序->SourceTree，右键显示包内容:

![显示包内容](https://raw.githubusercontent.com/lingjye/lingjye.github.io/master/img/sourcetree/show_content.png)

之后再Contents页面点击右上角搜索，在记得在Contents目录下搜索：

![搜索](https://raw.githubusercontent.com/lingjye/lingjye.github.io/master/img/sourcetree/search_0.png)

在搜索框输入atlassian，会看到包含atlassian的所有文件列表，之后全选搜索结果，并且移到废纸楼:

![搜索结果](https://raw.githubusercontent.com/lingjye/lingjye.github.io/master/img/sourcetree/search.png)

最后重启SourceTree，可以看到直接打开仓库页面，也不需要再登录。

![仓库](https://raw.githubusercontent.com/lingjye/lingjye.github.io/master/img/sourcetree/menu.png)


之前使用的版本比较老，但是也不影响使用，该版本以及license文件已上传至百度云盘，如有需要可自行下载。

百度云链接: https://pan.baidu.com/s/1x4TkaaCO8PEWwhX43olSbw 

提取码: ix2i