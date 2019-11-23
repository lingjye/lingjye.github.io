---
layout: post
title: "Mac下安装Jekyll"
subtitle: ''
author: "lingjye"
header-style: 'text'
tags:
  - iOS
---

安装命令行工具

```
xcode-select --install
```

安装 Homebrew

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

使用 brew 安装 ruby

```
brew install ruby
```

配置环境变量

```
export PATH=/usr/local/opt/ruby/bin:$PATH
```

然后可以使用以下命令检测：

```
which ruby
# /usr/local/opt/ruby/bin/ruby

ruby -v
# ruby 2.6.5p114 (2019-10-01 revision 67812) [x86_64-darwin18]
```

安装 jekyll

```
gem install --user-install bundler jekyll
```

配置，替换版本前两位：

```
export PATH=$HOME/.gem/ruby/X.X.0/bin:$PATH
# 例如：
# export PATH=$HOME/.gem/ruby/2.6.0/bin:$PATH
```

安装 bundle 

```
gem install bundle
```

Bundle安装, 依次执行下边命令， 会根据当前目录下的Gemfile，安装所需要的所有软件

```
bundle install

bundle update
```

启动服务：

```
bundle exec jekyll serve
```

参考：https://jekyllrb.com/docs/installation/macos/