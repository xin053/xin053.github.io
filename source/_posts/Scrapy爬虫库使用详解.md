---
title: Scrapy爬虫库使用详解
date: 2016-12-10 12:36:04
categories:
- Python模块学习
tags:
- Python
- Scrapy
- 爬虫
---

## Scrapy简介

![](https://scrapy.org/img/scrapylogo.png)

scrapy发出的请求是异步的，默认过滤掉相同的url。能做html/xml解析，数据能导出多种格式，还有强大的插件系统

scrapy(1.2.2)目前支持python 3，但是官方文档是也有说明，并不支持windows平台上的python3，因为scrapy的核心依赖`Twisted`目前并不支持windows平台上的python 3，所以知乎上有人推荐使用python 2.7，并需要安装[Visual C++ Compiler for Python 2.7](https://www.microsoft.com/en-us/download/details.aspx?id=44266)，并且window10 也支持这个软件，但是按照python开发者手册上的说明，[python2.7只会维护到2020年](https://docs.python.org/devguide/#status-of-python-branches)，并且python的未来也是指向python 3，基本上主流库都支持了python 3，并且很多库已经开始不支持python 2了，所以这里我还是想使用python 3.

关于为什么不支持windows平台，原因是windows上不能编译scrapy的依赖`lxml`和`Twisted`,但是我们可以下载已经编译好的`whl`包，用`pip`安装即可，详情，可以参考这篇博客: [python 3.5 + scrapy1.2 windows下的安装](https://my.oschina.net/wangyuefive/blog/784171)

<!-- more -->

## Scrapy使用

### 创建项目

```powershell
scrapy startproject test_scrapy
```

将会在当前工作目录下创建`test_scrapy`文件夹，文件下下有以下内容:

```
test_scrapy/
    scrapy.cfg            # deploy configuration file

    test_scrapy/             # project's Python module, you'll import your code from here
        __init__.py

        items.py          # project items definition file

        middlewares.py    # Define here the models for your spider middleware

        pipelines.py      # project pipelines file

        settings.py       # project settings file

        spiders/          # a directory where you'll later put your spiders
            __init__.py
```

### 第一个爬虫

我们编写的爬虫类必须继承`scrapy.Spider `并定义好初始请求链接，并且应该将文件放置在`spiders`目录下。

我们在`spiders`目录下创建`quotes_spider.py`:

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log('Saved file %s' % filename)
```

`name`是spider名称，同一项目中不能同名

`start_requests()`必须返回可迭代的`Requests`(一个`Requests`列表或者是生成器对象)，这些请求是爬虫初始的爬取对象.scrapy提供一种简单实现`start_requests()`的方式，就是使用`start_urls`列表，该列表在后台会被自动封装成`Requests`生成器并使用默认的回掉函数`parse()`

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)
```

`parse()`是默认的回调函数。`Request`可以设置得到响应后的回调函数。

### 运行爬虫

在项目的根目录执行:

```powershell
scrapy crawl quotes
```

`quotes`是爬虫名

将会看到以下输出:

```powershell
...
2016-12-11 14:39:27 [scrapy] INFO: Spider opened
2016-12-11 14:39:27 [scrapy] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2016-12-11 14:39:27 [scrapy] DEBUG: Telnet console listening on 127.0.0.1:6023
2016-12-11 14:39:28 [scrapy] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
2016-12-11 14:39:28 [scrapy] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
2016-12-11 14:39:28 [quotes] DEBUG: Saved file quotes-1.html
2016-12-11 14:39:29 [scrapy] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/> (referer: None)
2016-12-11 14:39:29 [quotes] DEBUG: Saved file quotes-2.html
2016-12-11 14:39:29 [scrapy] INFO: Closing spider (finished)
2016-12-11 14:39:29 [scrapy] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 675,
 'downloader/request_count': 3,
 'downloader/request_method_count/GET': 3,
 'downloader/response_bytes': 5976,
 'downloader/response_count': 3,
 'downloader/response_status_count/200': 2,
 'downloader/response_status_count/404': 1,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2016, 12, 11, 6, 39, 29, 492581),
 'log_count/DEBUG': 6,
 'log_count/INFO': 7,
 'response_received_count': 3,
 'scheduler/dequeued': 2,
 'scheduler/dequeued/memory': 2,
 'scheduler/enqueued': 2,
 'scheduler/enqueued/memory': 2,
 'start_time': datetime.datetime(2016, 12, 11, 6, 39, 27, 724826)}
2016-12-11 14:39:29 [scrapy] INFO: Spider closed (finished)
```

并在根目录生成`quotes-1.html`和`quotes-2.html`

### 解析网页

使用类选择器对html/xml进行解析,同时scrapy也支持[XPath表达式](http://www.w3school.com.cn/xpath/)

```python
>>> response.css('title')
[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]

>>> response.css('title::text').extract()
['Quotes to Scrape']

>>> response.css('title').extract()
['<title>Quotes to Scrape</title>']

>>> response.css('li.next a').extract_first()
'<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'

>>> response.css('li.next a::attr(href)').extract_first()
'/page/2/'
```

`response.css()`返回列表，如果想提取第一个，可以这样:

```python
>>> response.css('title::text').extract_first()
'Quotes to Scrape'

>>> response.css('title::text')[0].extract()
'Quotes to Scrape'
```

推荐使用第一种方式，这样，如果`response.css()`返回空列表，前者会返回`None`，后者会触发异常

除了使用 `extract()` 和 `extract_first()`提取数据，也可以使用`re()`进行正则提取

```python
>>> response.css('title::text').re(r'Quotes.*')
['Quotes to Scrape']
>>> response.css('title::text').re(r'Q\w+')
['Quotes']
>>> response.css('title::text').re(r'(\w+) to (\w+)')
['Quotes', 'Scrape']
```

### Following links

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('span small::text').extract_first(),
                # 'author': quote.xpath('span/small/text()').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page) # urljoin()获取完整url地址
            yield scrapy.Request(next_page, callback=self.parse)
```

```python
import scrapy


class AuthorSpider(scrapy.Spider):
    name = 'author'

    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        # follow links to author pages
        for href in response.css('.author+a::attr(href)').extract():
            yield scrapy.Request(response.urljoin(href),
                                 callback=self.parse_author)

        # follow pagination links
        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)

    def parse_author(self, response):
        def extract_with_css(query):
            return response.css(query).extract_first().strip()

        yield {
            'name': extract_with_css('h3.author-title::text'),
            'birthdate': extract_with_css('.author-born-date::text'),
            'bio': extract_with_css('.author-description::text'),
        }
```

### 命令行工具

```powershell
C:\WINDOWS\system32>scrapy
Scrapy 1.2.2 - no active project

Usage:
  scrapy <command> [options] [args]

Available commands:
  bench         Run quick benchmark test
  commands
  fetch         Fetch a URL using the Scrapy downloader
  genspider     Generate new spider using pre-defined templates
  runspider     Run a self-contained spider (without creating a project)
  settings      Get settings values
  shell         Interactive scraping console
  startproject  Create new project
  version       Print Scrapy version
  view          Open URL in browser, as seen by Scrapy

  [ more ]      More commands available when run from project directory

Use "scrapy <command> -h" to see more info about a command
```

更多命令以及命令的详细使用方法请参考[官方文档](https://doc.scrapy.org/en/latest/topics/commands.html#available-tool-commands)

### CrawlSpider

除了继承`scrapy.Spider`，常用的还有`scrapy.spiders.CrawlSpider`,该类可以在前者的基础上添加`Rule`。

```python
import scrapy
from scrapy.spiders import CrawlSpider, Rule
from scrapy.linkextractors import LinkExtractor

class MySpider(CrawlSpider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = ['http://www.example.com']

    rules = (
        # Extract links matching 'category.php' (but not matching 'subsection.php')
        # and follow links from them (since no callback means follow=True by default).
        Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

        # Extract links matching 'item.php' and parse them with the spider's method parse_item
        Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
    )

    def parse_item(self, response):
        self.logger.info('Hi, this is an item page! %s', response.url)
        item = scrapy.Item()
        item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
        item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
        item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
        return item
```

### SitemapSpider 

`scrapy.spiders.SitemapSpider`可以根据sitemaps和robots.txt进行爬去

```python
from scrapy.spiders import SitemapSpider

class MySpider(SitemapSpider):
    sitemap_urls = ['http://www.example.com/robots.txt']
    sitemap_rules = [
        ('/shop/', 'parse_shop'),
    ]
    sitemap_follow = ['/sitemap_shops']

    def parse_shop(self, response):
        pass # ... scrape shop here ...
```

规则中表示含有`/shop/`的url的回调函数为`parse_shop`,`sitemap_follow`表示只跟随包含`/sitemap_shops`的url

### Item

python自带的`dict`没有结构体的概念，所以scrapy提供了`Item`类

```python
import scrapy

class Product(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field()
    stock = scrapy.Field()
    last_updated = scrapy.Field(serializer=str)
```

```python
>>> product = Product(name='Desktop PC', price=1000)
>>> print product
Product(name='Desktop PC', price=1000)
>>> product['name']
Desktop PC
>>> product.get('name')
Desktop PC
>>> product['price']
1000

>>> product.keys()
['price', 'name']
>>> product.items()
[('price', 1000), ('name', 'Desktop PC')]
```

Item Loader能够更好将`response`中的数据注入到`Item`中

```python
from scrapy.loader import ItemLoader
from myproject.items import Product

def parse(self, response):
    l = ItemLoader(item=Product(), response=response)
    l.add_xpath('name', '//div[@class="product_name"]')
    l.add_xpath('name', '//div[@class="product_title"]')
    l.add_xpath('price', '//p[@id="price"]')
    l.add_css('stock', 'p#stock]')
    l.add_value('last_updated', 'today') # you can also use literal values
    return l.load_item()
```

### Item Pipeline

`Item`被爬取后会发送给pipeline进行处理，一般pipeline是只用实现`process_item`的类，也可以实现`open_spider()`(爬虫开始前执行)和`close_spider()`

```python
import pymongo

class MongoPipeline(object):

    collection_name = 'scrapy_items'

    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        self.db[self.collection_name].insert(dict(item))
        return item
```

以上是scrapy基础内容，更多有关scrapy，如log和email等查看[官方文档](https://doc.scrapy.org/en/latest/index.html)

## 参考文档

- [Scrapy官方文档](https://doc.scrapy.org/en/latest/index.html)
