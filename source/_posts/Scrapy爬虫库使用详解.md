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

scrapy发出的请求是异步的。能做html/xml解析，数据能导出多种格式，还有强大的插件系统

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

`start_requests()`必须返回可迭代的`Requests`(一个`Requests`列表或者是生成器对象)，这些请求是爬虫初始的爬取对象

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



## 参考文档

- [Scrapy官方文档](https://doc.scrapy.org/en/latest/index.html)
