---
layout: post
title: "Scrapy爬虫项目实战"
subtitle: '入门到上手'
author: "lingjye"
header-img: 'img/python_imgs/spider.jpg'
header-mask:	0.3
tags:
  - python
  - scrapy
  - scrapy-splash
---

* 部分由js动态渲染的网页抓取为空时, 可以采用scapy-splash框架, 下面介绍下如何使用scrapy-splash框架抓取动态渲染的页面

## Scrapy-Splash

**什么是Scrapy-Splash?**

它是一个JavaScript渲染服务, 由Python实现的一个HTTP API的轻量级浏览器, 同时使用Twisted和QT, Twisted（QT）用来让服务具有异步处理能力，以发挥webkit的并发能力, 所以性能上会比Selenium会好许多.

**特点:**

* 并行处理多个网页
* 得到HTML结果以及（或者）渲染成图片
* 关掉加载图片或使用 Adblock Plus规则使得渲染速度更快
* 使用JavaScript处理网页内容
* 使用Lua脚本
* 能在Splash-Jupyter Notebooks中开发Splash Lua scripts
* 能够获得具体的HAR格式的渲染信息

**安装:**

scrapy-splash使用的是Splash HTTP API, 所以需要一个splash instance, 一般采用docker运行splash, 首先需要先安装Docker

### I. Mac环境下安装

[1] 安装并启动Docker

	点击下载 [Docker for Mac](https://docs.docker.com/docker-for-mac/install/){:target="_blank"} 

[2] 拉取 splash 镜像资源

```
$ sudo docker pull scrapinghub/splash
```

如果Docker获取镜像报错 docker: Error response from daemon, 可使用Docker国内镜像. 

修改方法: 

> 右键点击桌面顶栏的 docker 图标，选择 Preferences ，在 Daemon 标签（Docker 17.03 之前版本为 Advanced 标签）下的 Registry mirrors 列表中加入下面的镜像地址:
> 
> http://141e5461.m.daocloud.io
>
> 点击 Apply 应用, 然后点击 Restart 使设置生效
> 

[3] 开启容器

```
$ sudo docker run -p 8050:8050 scrapinghub/splash
```

运行成功, 可在浏览器中输入`localhost:8050`验证.

### II. Linux CentOS环境安装

[1] 安装并启动Docker

安装

```
$ sodu yum install docker
```

启动

```
$ sudo service docker start
```

该过程可能需要几分钟时间, 

[2] 拉取 splash 镜像资源

```
$ sudo docker pull scrapinghub/splash
```

[3] 开启容器

```
$ sudo docker run -p 8050:8050 scrapinghub/splash
```

**使用**

[1] 安装Scrapy-Splash模块

```
$ pip install scrapy-splash
```

[2] 修改配置信息:

找到Scrapy爬虫项目的settings.py文件, 修改配置信息:

```
# 添加splash服务地址
SPLASH_URL = 'http://localhost:8050'

# 启用SplashDeduplicateArgsMiddleware
SPIDER_MIDDLEWARES = {
    'scrapy_splash.SplashDeduplicateArgsMiddleware': 100,
}

# 将splash middleware添加到DOWNLOADER_MIDDLEWARE中：
DOWNLOADER_MIDDLEWARES = {
    'scrapy_splash.SplashCookiesMiddleware': 723,
    'scrapy_splash.SplashMiddleware': 725,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 810
}

# 使用splash默认过滤规则, 也可使用自定义规则
DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'
# 使用splash缓存
HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'
```

[3] 修改爬虫文件MySpider.py

```
# -*- coding: utf-8 -*-
from scrapy import Spider, Request
from scrapy_splash import SplashRequest

class MySpider(scrapy.Spider):
    name = 'MySpider'
    # allowed_domains = ['']
    url = 'http://m.2958.cn/'
    
    # start request
    def start_requests(self):
    	 # splash lua script
    	 splash_args = {"lua_source": """
                                        --splash.response_body_enabled = true
                                        splash.private_mode_enabled = false
                                        splash:set_user_agent("Mozilla/5.0 (iPhone; CPU iPhone OS 11_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/11.0 Mobile/15E148 Safari/604.1")
                                        assert(splash:go("%s"))
                                        splash:wait(1)
                                        return {html = splash:html()}
                                        """ % (self. url)}
        # Scrapy 方法
        # yield Request(self.url, callback=self.parse)
        # Scrapy-Splash 方法
        yield SplashRequest(self.url, callback=self.parse, endpoint='run', args= splash_args)
   
    # parse the html content 
    def parse(self, response):
    	# 此处可对比下使用Scrapy的Request和Scrapy-Splash的SplashRequest获取到response.body的不同
        print(response.body)
```