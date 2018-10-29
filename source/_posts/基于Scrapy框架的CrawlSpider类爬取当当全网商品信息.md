---
title: 基于Scrapy框架的CrawlSpider类爬取当当全网商品信息
date: 2018-09-01 09:03:56
categories: "python3网络爬虫"
tags: 
  - 爬虫
---

# 前言

本项目通过使用Scrapy框架的CrawlSpider类，对当当全网商品信息进行爬取并将信息保存至mysql数据库，当当网反爬措施是对IP访问频率的限制，所以本项目使用了中间件`scrapy-rotating-proxies`来管控IP代理池，有关代理ip的爬取请见我的另一篇博文。

CrawlSpider是Spider的派生类，Spider类的设计原则是只爬取start_url列表中的网页，而CrawlSpider类通过定义一些规则(rule)来跟进所爬取网页中的link，从爬取的网页中获取link并继续爬取。

Github地址: https://github.com/RunningGump/crawl_dangdang

# 依赖

1. scrapy 1.5.0
2. python3.6
3. mysql 5.7
4. pymysql 库
5. scrapy-rotating-proxies 库
6. fake-useragent 库

# 创建项目

首先，我们需要创建一个Scrapy项目，在shell中使用`scrapy startproject`命令：

```bash
$ scrapy startproject Dangdang
New Scrapy project 'Dangdang', using template directory '/usr/local/lib/python3.6/dist-packages/scrapy/templates/project', created in:
    /home/geng/Dangdang

You can start your first spider with:
    cd Dangdang
    scrapy genspider example example.com
```

创建好一个名为`Dangdang`的项目后，接下来，你进入新建的项目目录：

```bash
$ cd Dangdang
```

然后,使用`scrapy genspider -t <template> <name> <domain>`创建一个spider：

```bash
$ scrapy genspider -t crawl dd dangdang.com
Created spider 'dd' using template 'crawl' in module:
  Dangdang.spiders.dd
```

此时，你通过`cd ..`返回上级目录，使用`tree`命令查看项目目录下的文件，显示如下：

```bash
$ cd ..
$ tree Dangdang/
Dangdang/
├── Dangdang
│   ├── __init__.py
│   ├── items.py
│   ├── middlewares.py
│   ├── pipelines.py
│   ├── __pycache__
│   │   ├── __init__.cpython-36.pyc
│   │   └── settings.cpython-36.pyc
│   ├── settings.py
│   └── spiders
│       ├── dd.py
│       ├── __init__.py
│       └── __pycache__
│           └── __init__.cpython-36.pyc
└── scrapy.cfg

4 directories, 11 files
```

到此为止，我们的项目就创建成功了。


# rules

在rules中包含一个或多个Rule对象，每个Rule对爬取网站的动作设置了爬取规则。

 **参数介绍：**

`link_extractor`：是一个Link Extractor对象，用于定义需要提取的链接。

`callback`： 回调函数，对link_extractor获得的链接进行处理与解析。

**注意事项：**当编写爬虫规则时，避免使用parse作为回调函数。由于CrawlSpider使用parse方法来实现其逻辑，如果覆盖了 parse方法，crawl spider将会运行失败。

`follow`：是一个布尔(boolean)值，指定了根据规则从response提取的链接是否需要跟进。 如果callback为None，follow 默认设置为True ，否则默认为False

`process_links`：指定该spider中哪个的函数将会被调用，从link_extractor中获取到链接列表时将会调用该函数。该方法主要用来过滤链接。          

`process_request`：指定该spider中哪个的函数将会被调用， 该规则提取到每个request时都会调用该函数。 (用来过滤request)

# LinkExtrator

**参数介绍：**

`allow`：满足括号中“正则表达式”的值会被提取，如果为空，则全部匹配。

`deny`：与这个正则表达式(或正则表达式列表)匹配的URL不提取。

`allow_domains`：会被提取的链接的域名。

`deny_domains`：不会被提取链接的域名。

`restrict_xpaths`：使用Xpath表达式与allow共同作用提取出同时符合对应Xpath表达式和正则表达式的链接；

# 项目代码

## 编写item.py文件

```python
import scrapy

class DangdangItem(scrapy.Item):
    category = scrapy.Field() 	 # 商品类别
    title = scrapy.Field()   	 # 商品名称
    link = scrapy.Field()   	 # 商品链接
    price = scrapy.Field()  	 # 商品价格
    comment = scrapy.Field() 	 # 商品评论数
    rate = scrapy.Field()   	 # 商品的好评率
    source = scrapy.Field()      # 商品的来源地
    detail = scrapy.Field()  	 # 商品详情
    img_link = scrapy.Field()	 # 商品图片链接
```

## 编写pipeline.py文件

需提前创建好数据库，本项目创建的数据库名字为`dd`。

```python
import pymysql
from scrapy.conf import settings

## pipeline默认是不开启的，需在settings.py中开启
class DangdangPipeline(object):
    def process_item(self, item, spider):
        ##建立数据库连接
        conn = pymysql.connect(host="localhost",user="root",passwd='yourpasswd',db="dd",use_unicode=True, charset="utf8")
        cur = conn.cursor()             # 用来获得python执行Mysql命令的方法,也就是我们所说的操作游标
        print("mysql connect success")  # 测试语句，这在程序执行时非常有效的理解程序是否执行到这一步
        try:
            category = item["category"]
            title = item["title"]
            if len(title)>40:
                title = title[0:40] + '...'
            link = item["link"]
            img_link = item['img_link']
            price = item["price"]
            comment = item["comment"]
            rate = item["rate"]
            source = item["source"]
            detail = item["detail"]

            sql = "INSERT INTO goods(category,title,price,comment,rate,source,detail,link,img_link) VALUES ('%s','%s','%s','%s','%s','%s','%s','%s','%s')" % (category,title,price,comment,rate,source,detail,link,img_link)
            print(sql)
        except Exception as err:
            pass

        try:
            cur.execute(sql)         # 真正执行MySQL语句，即查询TABLE_PARAMS表的数据
            print("insert success")  # 测试语句
        except Exception as err:
            print(err)
            conn.rollback() #事务回滚,为了保证数据的有效性将数据恢复到本次操作之前的状态.有时候会存在一个事务包含多个操作，而多个操作又都有顺序，顺序执行操作时，有一个执行失败，则之前操作成功的也会回滚，即未操作的状态
        else:
            conn.commit()   #当没有发生异常时，提交事务，避免出现一些不必要的错误
            
        conn.close()  #关闭连接
        return item   #框架要求返回一个item对象

```

## 编写middlewares.py

本项目添加了`RandomUserAgentMiddleWare`中间件，用来随机更换UserAgent。在`middlewares.py`文件的最后面添加如下中间件：

```python
from fake_useragent import UserAgent

class RandomUserAgentMiddleWare(object):
    """
    随机更换User-Agent
    """
    def __init__(self,crawler):
        super(RandomUserAgentMiddleWare, self).__init__()
        self.ua = UserAgent()
        self.ua_type = crawler.settings.get("RANDOM_UA_TYPE", "random")

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler)

    def process_request(self, request, spider):
        def get_ua_type():
            return getattr(self.ua, self.ua_type)   # 取对象 ua 的 ua_type 的这个属性, 相当于 self.ua.self.ua_type

        request.headers.setdefault('User-Agent', get_ua_type())
```

## 修改settings.py 文件

```python
BOT_NAME = 'Dangdang'

SPIDER_MODULES = ['Dangdang.spiders']
NEWSPIDER_MODULE = 'Dangdang.spiders'

# 不遵循robots协议
ROBOTSTXT_OBEY = False

# 下载延迟设置为0，提高爬取速度
DOWNLOAD_DELAY = 0

#禁用Cookie(默认情况下启用)
COOKIES_ENABLED = False  

# 启用所需要的下载中间件
DOWNLOADER_MIDDLEWARES = {
    'rotating_proxies.middlewares.RotatingProxyMiddleware': 610,
    'rotating_proxies.middlewares.BanDetectionMiddleware': 620,
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': None,
    'Dangdang.middlewares.RandomUserAgentMiddleWare': 400,
}
# 代理IP文件路径,此处需改为你自己的路径
ROTATING_PROXY_LIST_PATH = '/home/geng/Projects/Dangdang/proxy.txt'

# 随机更换UserAgent 
RANDOM_UA_TYPE = "random"

# 开启pipeline
ITEM_PIPELINES = {
   'Dangdang.pipelines.DangdangPipeline': 300,
}
```

## spider文件(dd.py)编写

```python
# -*- coding: utf-8 -*-
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from Dangdang.items import DangdangItem
import re
import urllib.request
import json

class DdSpider(CrawlSpider):
    name = 'dd'
    allowed_domains = ['dangdang.com']
    start_urls = ['http://category.dangdang.com/']
    
    # 分析网页链接，编写rules规则,提取商品详情页的链接
    rules = (
        Rule(LinkExtractor(allow=r'/cp\d{2}.\d{2}.\d{2}.\d{2}.\d{2}.\d{2}.html$|/pg\d+-cp\d{2}.\d{2}.\d{2}.\d{2}.\d{2}.\d{2}.html$', deny=r'/cp98.\d{2}.\d{2}.\d{2}.\d{2}.\d{2}.\d{2}.html'),
             follow=True),
        Rule(LinkExtractor(allow=r'/cid\d+.html$|/pg\d+-cid\d+.html$', deny=r'/cp98.\d{2}.\d{2}.\d{2}.\d{2}.\d{2}.\d{2}.html'),
             follow=True),
        Rule(LinkExtractor(allow=r'product.dangdang.com/\d+.html$', restrict_xpaths=("//p[@class='name']/a")),
             callback='parse_item',
             follow=False),      # allow与restrict_xpath配合使用,效果很好,可以更精准筛选链接.
    )

    # 解析商品详情页面
    def parse_item(self, response):
        item = DangdangItem()  # 实例化item
        item["category"] = response.xpath('//*[@id="breadcrumb"]/a[1]/b/text()').extract_first()+'>'+response.xpath('//*[@id="breadcrumb"]/a[2]/text()').extract_first()+'>'+response.xpath('//*[@id="breadcrumb"]/a[3]/text()').extract_first()
        item["title"] = response.xpath("//*[@id='product_info']/div[1]/h1/@title").extract_first()
        item["detail"] = json.dumps(response.xpath("//*[@id='detail_describe']/ul//li/text()").extract(),ensure_ascii=False)
        item["link"] = response.url
        item["img_link"] =json.dumps(response.xpath("//div[@class='img_list']/ul//li/a/@data-imghref").extract())
        try:
            item["price"] = response.xpath("//*[@id='dd-price']/text()").extract()[1].strip()
        except IndexError as e:
            item["price"] = response.xpath("//*[@id='dd-price']/text()").extract()[0].strip()
            item["comment"] = response.xpath("//*[@id='comm_num_down']/text()").extract()[0]

        try:
            item["source"] = response.xpath("//*[@id='shop-geo-name']/text()").extract()[0].replace('\xa0至','')
        except IndexError as e:
            item["source"] = '当当自营'

        # 通过抓包分析,提取商品的好评率
        goodsid = re.compile('\/(\d+).html').findall(response.url)[0]  # 提取url中的商品id
        # 提取详情页源码中的categoryPath
        script = response.xpath("/html/body/script[1]/text()").extract()[0]
        categoryPath = re.compile(r'.*categoryPath":"(.*?)","describeMap').findall(script)[0]
        # 构造包含好评率包的链接
        rate_url = "http://product.dangdang.com/index.php?r=comment%2Flist&productId="+str(goodsid)+"&categoryPath="+str(categoryPath)+"&mainProductId="+str(goodsid)
        rate_date = urllib.request.urlopen(rate_url).read().decode("utf-8")  # 读取包含好评率的包的内容
        item["rate"] = re.compile(r'"goodRate":"(.*?)"').findall(rate_date)[0]+'%'   # 用正则表达式提取好评率

        yield item
```

# 使用方法

在命令行中执行以下命令开始爬取：

```bash
$ scrapy crawl dd
```

# 结果展示

![结果展示](基于Scrapy框架的CrawlSpider类爬取当当全网商品信息/结果.png)