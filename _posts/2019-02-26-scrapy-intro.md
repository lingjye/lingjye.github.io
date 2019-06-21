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

### 简介

Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架。 可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。

其最初是为了 页面抓取 (更确切来说, 网络抓取 )所设计的， 也可以应用在获取API所返回的数据或者通用的网络爬虫。

### 安装

环境: Mac, Python 3.7 

使用 pip 安装命令:

```
pip install scrapy
```

### 创建 Scrapy 项目

1. 命令行 cd 到代码存储路径下, 例如桌面上新建的文件夹 Scrapy 目录

	```
	cd desktop/Scrapy
	```

2. 使用 scrapy 命令创建项目:

	```
	scrapy startproject tutorial
	```
	
	等待执行结束, 一般会有提示进行下一步创建爬虫操作.
	
	改命令还会创建如下 tutorial 目录:
	
	```
	tutorial/
	    scrapy.cfg
	    tutorial/
	        __init__.py
	        __pycache__
	        items.py
	        pipelines.py
	        settings.py
	        spiders/
	            __init__.py
	            __pycache__
	```
	
3. 创建爬虫文件

	安装第二步提示, 分别执行以下命令:
	
	```
	cd tutorial
	scrapy genspider Baidu baidu.com
	```
	
	可以在 spiders 目录下看到新生成一个 JanShu.py 文件
	
	使用 PyCharm 或者 VSCode 将项目打开, 可以看到 Baidu.py 文件中有一个 BaiduSpider 类, 且继承自 scrapy.Spider, 其中的属性如下:
	
	* name: 用于区别Spider。 该名字必须是唯一的，您不可以为不同的Spider设定相同的名字。
	* start_urls: 包含了Spider在启动时进行爬取的url列表。 因此，第一个被获取到的页面将是其中之一。 后续的URL则从初始的URL获取到的数据中提取。
	* parse(self, response) 是spider的一个方法。 被调用时，每个初始URL完成下载后生成的 Response 对象将会作为唯一的参数传递给该函数。 该方法负责解析返回的数据(response data)，提取数据(生成item)以及生成需要进一步处理的URL的 Request 对象。

4. 开启爬虫任务
	
	这里需要进入项目的根目录, 也就是第二步最开始的tutorial目录下.
	
	开启命令:
	
	```
	scrapy crawl Baidu
	```
	
	之后可以看到命令行控制台输出响应, 可以看到返回状态码200.
	
	由于scrapy 默认遵从爬虫爬取规则, 所以也会看到:
	
	> DEBUG: Forbidden by robots.txt: <GET http://baidu.com/>
	
	可点击查看[http://baidu.com/robots.txt]('http://baidu.com/robots.txt'){:target="_blank"}
	
	下面会继续通过实例介绍 Scrapy 如何使用.
	
	也推荐阅读 Scrapy [文档](https://scrapy-chs.readthedocs.io/zh_CN/0.24/index.html){:target="_blank"}

### 实例
	
我们以简书为例, 来爬取首页中的文章标题, 作者, 缩略图等.

1. 新建爬虫 JianShuSpider
	
	```
	scrapy genspider JianShu www.jianshu.com
	```
2. 声明Item

	Item使用简单的class定义语法以及 Field 对象来声明。
	
	```
	import scrapy


	class TutorialItem(scrapy.Item):
	    # define the fields for your item here like:
	    # name = scrapy.Field()
	    # 文章标题
	    title = scrapy.Field() 	
	    # 作者
	    author = scrapy.Field()
	    # 图片	
	    img = scrapy.Field()	
	```
3. 设置UA(可选)

	在 settings.py 中修改 
	
	```
	# settings.py
	USER_AGENT = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36'
	```
	
	如果要设置随机值, 可以如下, 附上一些 UA:
	
	```
	# settings.py
	USER_AGENTS_LIST = [
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36',
   'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_3) AppleWebKit/536.5 (KHTML, like Gecko) Chrome/19.0.1084.54 Safari/536.5',
   "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; AcooBrowser; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; Acoo Browser; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; .NET CLR 3.0.04506)",
    "Mozilla/4.0 (compatible; MSIE 7.0; AOL 9.5; AOLBuild 4337.35; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
    "Mozilla/5.0 (Windows; U; MSIE 9.0; Windows NT 9.0; en-US)",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
    "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
    "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)",
    "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0",
    "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5",
    "Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.8) Gecko Fedora/1.9.0.8-1.fc10 Kazehakase/0.5.6",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3) AppleWebKit/535.20 (KHTML, like Gecko) Chrome/19.0.1036.7 Safari/535.20",
    "Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; fr) Presto/2.9.168 Version/11.52",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.11 (KHTML, like Gecko) Chrome/20.0.1132.11 TaoBrowser/2.0 Safari/536.11",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 (KHTML, like Gecko) Chrome/21.0.1180.71 Safari/537.1 LBBROWSER",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E; LBBROWSER)",
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; QQDownload 732; .NET4.0C; .NET4.0E; LBBROWSER)",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.84 Safari/535.11 LBBROWSER",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E)",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E; QQBrowser/7.0.3698.400)",
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; QQDownload 732; .NET4.0C; .NET4.0E)",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Trident/4.0; SV1; QQDownload 732; .NET4.0C; .NET4.0E; 360SE)",
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; QQDownload 732; .NET4.0C; .NET4.0E)",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E)",
    "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.1 (KHTML, like Gecko) Chrome/21.0.1180.89 Safari/537.1",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 (KHTML, like Gecko) Chrome/21.0.1180.89 Safari/537.1",
    "Mozilla/5.0 (iPad; U; CPU OS 4_2_1 like Mac OS X; zh-cn) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8C148 Safari/6533.18.5",
    "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.0b13pre) Gecko/20110307 Firefox/4.0b13pre",
    "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:16.0) Gecko/20100101 Firefox/16.0",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.64 Safari/537.11",
    "Mozilla/5.0 (X11; U; Linux x86_64; zh-CN; rv:1.9.2.10) Gecko/20100922 Ubuntu/10.10 (maverick) Firefox/3.6.10"
]
	```
	
	移动端UA标识:
	
	```
	MOBILE_USER_AGENTS_LIST = [
	    "Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B137 Safari/601.1",
	    "Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1 (KHTML, like Gecko) CriOS/74.0.3729.131 Mobile/13B143 Safari/601.1.46",
	    "Mozilla/5.0 (iPad; CPU OS 9_1 like Mac OS X) AppleWebKit/601.1 (KHTML, like Gecko) CriOS/74.0.3729.131 Mobile/13B143 Safari/601.1.46",
	    "Mozilla/5.0 (iPhone; CPU iPhone OS 8_3 like Mac OS X) AppleWebKit/600.1.4 (KHTML, like Gecko) FxiOS/1.0 Mobile/12F69 Safari/600.1.4",
	    "Mozilla/5.0 (iPad; CPU iPhone OS 8_3 like Mac OS X) AppleWebKit/600.1.4 (KHTML, like Gecko) FxiOS/1.0 Mobile/12F69 Safari/600.1.4",
	    "Mozilla/5.0 (iPad; CPU OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B137 Safari/601.1",
	    "Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B137 Safari/601.1",
	    "UCWEB/2.0 (iPad; U; CPU OS 7_1 like Mac OS X; en; iPad3,6) U2/1.0.0 UCBrowser/9.3.1.344",
	    "Mozilla/5.0 (iPhone; CPU iPhone OS 11_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/11.0 Mobile/15E148 Safari/604.1",
	]

	```
	
	之后在 middlewares.py 中修改 TutorialSpiderMiddleware 类的init方法:
	
	```
	# middlewares.py
	class TutorialSpiderMiddleware(object):

	    def __init__(self, user_agent=""):
	        self.user_agent = user_agent
	```
	
	以及 TutorialDownloaderMiddleware 类中的方法:
	
	```
	# middlewares.py
	class TutorialDownloaderMiddleware(object):
	
		def __init__(self, user_agent):
   			self.user_agent = user_agent
		        
		@classmethod
		def from_crawler(cls, crawler):
		# This method is used by Scrapy to create your spiders.
			s = cls(user_agent=crawler.settings.get("USER_AGENTS_LIST"))
			crawler.signals.connect(s.spider_opened, signal=signals.spider_opened)
			return s
        
       def process_request(self, request, spider):
       	 agent = random.choice(self.user_agent)
       	 request.headers["User-Agent"] = agent
	```
	
	这里有个问题, 就是在 process_request() 方法中设置 UA 时, 使用 `request.headers.setdefaults()` 可能会无效. 这里可以查看[这篇文章](https://blog.csdn.net/stupid56862/article/details/86654546){:target="_blank"},
        
   	>这是因为 request.headers.setdefault() 方法中 . setdefault 方法意味着 :
    如果键不存在于字典中，将会添加键并将值设为默认值。如果存在,则不添加 。
    可参考 [文档](https://www.runoob.com/python/att-dictionary-setdefault.html){:target="_blank"},
    因为系统的 UserAgentMiddleware 已经添加了 User-Agent ,所以我们这里添加失败.

4. 防爬处理(可选)

	首先设置不遵循反爬虫规则, 一般网站都有设置, 可参考百度 (http://baidu.com/robots.txt)
	
	这里我们修改 settings.py
	
	```
	ROBOTSTXT_OBEY = False
	```
	
	设置延时, 延时能解决大部分反爬问题.
	
	```
	# 可设置随机值
	DOWNLOAD_DELAY = 1
	```
	
	也可以在 middlewares.py中 TutorialDownloaderMiddleware 类的 process_request() 方法设置
	
	```
	# middlewares.py
	class TutorialDownloaderMiddleware(object):
	
		def process_request(self, request, spider):
			deley = random.choice([0.5, 0.6, 0.7, 0.8, 1, 1.1, 1.2])
			time.sleep(deley)
	```
	
	还有就是设置代理, 一般需要收费, 也有免费的, 但是不太好使.
	
	言归正传. 在 middlewares.py 增加:
	
	```
	# middlewares.py
	class HttpbinProxyMiddleware(object):
 
    def process_request(self, request, spider):
        pro_addr = requests.get('http://127.0.0.1:5000/get').text
        request.meta['proxy'] = 'http://' + pro_addr
	```
	
	然后回到 settings.py, 修改:
	
	```
	# settings.py
	DOWNLOADER_MIDDLEWARES = {
   		'httpbin.middlewares.HttpbinProxyMiddleware': 543,
	}
	```
	
5. 自定义过滤规则(可选)
	
	新建 httpdupefilters.py 文件, 如下:
	
	```
	# httpdupefilters.py
	from scrapy.dupefilters import BaseDupeFilter
	from scrapy.utils.request import request_fingerprint
	
	class HttpDupefilters(BaseDupeFilter):
		def __init__(self):
			#初始化visited_fd为一个集合[也可以放到redis中]
			self.visited_fd = set()

		@classmethod
		def from_settings(cls, settings):
			return cls()
	
		def request_seen(self, request):
			'''
			:param request: 请求url[进行类似md5加密的操作]
			http://www.baidu.com?su=123&456
			http://www.baidu.com?su=456&123 以上两个的伪MD5是一样的
			伪MD5值得方法是request_fingerprint
			:return:
			'''
			print('=======过滤=======')
			fd = request_fingerprint(request=request)
			# 如果路径在visited_fd中返回True
			if fd in self.visited_fd:
				print('已过滤:', request)
				return True
			#添加到集合中
			self.visited_fd.add(fd)
	
		def open(self):	#can return deferred
			print('=====开始=====')
	
		def close(self, reason):	#can return a deferred
			print('=====结束=====')
	
		def log(self, request, spider):	#log that a request has been filterd
			print('=====日志=====')
			spider.crawler.stats.inc_value('dupefilter/filtered', spider=spider)
	```
	
	然后回到 settings.py 中, 编辑:
	
	```
	# settings.py
	DUPEFILTER_CLASS = 'tutorial.httpdupefilters.HttpDupeFilter'
	```
	
	然后配合爬虫文件中 parse() 方法使用:
	
	```
	# JianShu.py
	import scrapy
	from scrapy.http import Request
	from tutorial.items import TutorialItem
	
	class JianshuSpider(scrapy.Spider):
	    name = 'JianShu'
	    allowed_domains = ['jianshu.com']
	    start_urls = ['http://www.jianshu.com/']
	    
	    def parse(self, response):
	        meta = response.meta['item']
	        print(meta, response.body)
	        items = []
	        item = TutorialItem()
	        item.title = ''
	        item.author = ''
	        item.img = ''
	        item.url = ''
	        for item in items:
	            yield Request(item.url,
	                          callback=self.parse_content,
	                          meta={"item": 'item'},
	                          dont_filter=False,
	                          )
	
	        return items
		        
		def parse_content(self, response):
			meta = response.meta['item']
			print(meta, response.body)
			pass
	```
	
6. 在请求前处理 start_urls 参数, 或者传参

	重写父类 start_requests() 方法:
	
	```
	# JianShu.py
	class JianshuSpider(scrapy.Spider):
	    name = 'JianShu'
	    allowed_domains = ['jianshu.com']
	    start_urls = ['http://www.jianshu.com/']
	
	    # 重写父类的方法, 处理 start_urls 参数等
	    def start_requests(self):
	        yield Request('http://www.jianshu.com/',
	                      callback=self.parse,
	                      meta={"item": 'item'},
	                      dont_filter=False,
	                      )
	```

7. 保存数据

	此处以 MongoDB 为例, Monogo安装步骤, 参考文档: [https://docs.mongodb.com/](https://docs.mongodb.com/){:target="_blank"}
	
	Python 中使用的库为 pymongo, 安装步骤, 参考文档: [http://api.mongodb.com/python/current/index.html](http://api.mongodb.com/python/current/index.html){:target="_blank"}
	
	修改 settings.py 文件:
	
	```
	# settings.py
	MONGO_URI = 'mongodb://localhost:27017'
	MONGO_DB = "tutorial"
	```
	
	找到 pipelines.py, 编辑内容如下:
	
	```
	# pipelines.py
	import pymongo
	
	class TutorialPipeline(object):
	
	    def __init__(self, mongo_uri, mongo_db):
	        # self.file = codecs.open('item.json', 'wb', encoding='utf-8')
	        self.mongo_uri = mongo_uri
	        self.mongo_db = mongo_db
	
	    @classmethod
	    def from_crawler(cls, crawler):
	        '''
	        scrapy为我们访问settings提供了这样的一个方法，这里，
	        我们需要从settings.py文件中，取得数据库的URI和数据库名称
	        '''
	        return cls(
	            mongo_uri = crawler.settings.get('MONGO_URI'),
	            mongo_db = crawler.settings.get('MONGO_DB'),
	        )
	
	    def open_spider(self, spider):
	        '''
	        爬虫一旦开启, 就会实现这个方法, 链接到数据库
	        '''
	        self.client = pymongo.MongoClient(self.mongo_uri)
	        self.db = self.client[self.mongo_db]
	
	    def close_spider(self, spider):
	        '''
	        爬虫一旦关闭，就会实现这个方法，关闭数据库连接
	        '''
	        self.client.close()
	
	    def process_item(self, item, spider):
	        '''
	        每个实现保存的类里面必须要有这个方法, 且名字固定, 用来具体实现怎么保存
	        '''
	        # if not item['title']:
	        #     # return item
	        #     print('item不存在title:',item)
	        '''
	        SQLServer 更新数据库
	        '''
	        if not item['description']:
	            return item
	
	        '''
	        MongoDB 更新数据库 
	        '''
	        # 本地测试 MongoDB
	        data = {
	            'title': item['title'],
	            'author': item['author'],
	            'img': item['img'],
	            'url': item['url'],
	        }
	        #
	        table = self.db[spider.name]
	        # # table.insert_one(data)
	        table.update({'href': item['href']}, data, True);
	        return item
	```
	
	使用文件格式保存, 可使用如下方式:
	
	```
	line = json.dumps(dict(item), ensure_ascii=False) + '\n'
   self.file.write(line)
	```

[Demo](https://github.com/lingjye/Python-Notes/tree/master/Note_spider/tutorial){:target="_blank"}

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