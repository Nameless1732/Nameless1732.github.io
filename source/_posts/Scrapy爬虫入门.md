title: Scrapy爬虫入门
author: Jie
categories:
  - 数据采集
date: 2022-11-11 23:27:54
tags:
---
> 实验要求
> 1.要求爬取贝壳新房济南 https://jn.zu.ke.com/zufang 挂牌出租的全部楼盘信息。爬取信息包括楼盘名称、链接、地址、大小、方向、居室、价格、姓名。 
> 2.实现多页爬取（多次请求解析）：start_urls作为请求的入口地址，类型为列表，可以在列表中追加我们想要爬取页面的url地址。还需要在爬虫程序中添加`scrapy.Request(url[,callback,method=“GET”,headers,body,cookies,meta,dont_filter=False])`
> 3.爬取5页中所有要求的数据，起始页从学号尾数开始，存入csv中

<!-- more -->
### 安装Scrapy框架
通过pip安装scrapy：
```
pip install Scrapy
```
创建一个新的scrapy项目：
```
scrapy startproject Beike
```
执行完命令生成的文件树如下图所示
![文件树](/images/pasted-15.png)

### 完善item.py文件
```python
class BeikeItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    title = scrapy.Field()
    link = scrapy.Field()
    address = scrapy.Field()
    big = scrapy.Field()
    where = scrapy.Field()
    how = scrapy.Field()
    price = scrapy.Field()
    name = scrapy.Field()
```
### 在setting.py中设置UA
```python
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36 Edg/107.0.1418.35'
```
### 编写爬虫文件
```python
class BeikeSpider(scrapy.Spider):
    name = 'beike'
    allowed_domains = ['jn.zu.ke.com']
    start_urls = ['https://jn.zu.ke.com/zufang']
    page = 20
    def parse(self, response):
        print(response.url)
        node_list = response.xpath('//div[@class="content__list--item--main"]')
        print(len(node_list))
        item = BeikeItem()
        for node in node_list:
            item["title"] = node.xpath("./p[1]/a/text()").get().strip().strip()
            item["link"] = response.urljoin(node.xpath("./p[1]/a/@href").get().strip())
            item["address"] = node.xpath("./p[2]/a[3]/text()").get()
            item["big"] = node.xpath("./p[2]/text()[last()-3]").get()
            item["where"] = node.xpath("./p[2]/text()[last()-2]").get()
            item["how"] = node.xpath("./p[2]/text()[last()-1]").get().strip()
            item["price"] = node.xpath(
                './span[@class="content__list--item-price"]/em/text()').get().strip() + '元/月'
            if item["address"]:
                item["address"] = item["address"].strip()
            if item["big"]:
                item["big"] = item["big"].strip()
            if item["where"]:
                item["where"] = item["where"].strip()
            yield scrapy.Request(
                url=item["link"],
                callback=self.detail_parse,
                meta={"item": copy.deepcopy(item)},
                dont_filter=True
            )
        if self.page < 24:
            next_url = 'https://jn.zu.ke.com/zufang/pg{}/#contentList'.format(self.page)
            self.page += 1
            yield scrapy.Request(next_url, callback=self.parse)
    def detail_parse(self, response):
        item = response.meta['item']
        item["name"] = response.xpath('//*[@id="aside"]/div[2]/div[2]/div[1]/span/text()').extract_first()
        yield item
```
运行命令进行爬取：
```
scrapy crawl beike
```
![运行截图](/images/pasted-16.png)

### 使用pipline将数据保存到csv
```python
import csv
class SavePipeline(object):
    def open_spider(self, spider):
        self.file = open("贝壳.csv", 'a', newline="",encoding="gb18030")
        self.csv_writer = csv.writer(self.file)
        self.csv_writer.writerow(["标题", "链接", '地址', "大小", "方向", "居室",
                                  "价格", "名字"])
    def process_item(self, item, spider):
        self.csv_writer.writerow(
            [item["title"], item["link"], item["address"],
             item["big"], item["where"], item["how"], item["price"], item["name"]]
        )
        return item
    def close_spider(self, spider):
        self.file.close()
```

![csv文件](/images/pasted-17.png)
![csv文件](/images/pasted-18.png)

## 实验总结
Scrapy是用Python实现的一个为了爬取网站数据、提取结构性数据而编写的应用框架。Scrapy常应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。我们可以很简单的通过Scrapy框架实现一个爬虫，抓取指定网站的内容或图片。