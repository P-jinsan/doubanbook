## 环境
Pycharm、Python3.8、谷歌浏览器

## 要求

 1. 网址：[豆瓣读书](https://www.douban.com/doulist/1264675/)
 2. 要求：获取每本书的标题、作者、评分
 3. 将获取的信息输出到一个excle文件中

 ## 注意
 
 需要获取的内容可能不只是一页
 
## 过程(只列出修改过的文件)

 1. dbbook.py

```python
import scrapy
import re
from doubanbook.items import DoubanbookItem
class DbbookSpider(scrapy.Spider):
    name = 'dbbook'
    #allowed_domains = ['https://www.douban.com/doulist/1264675/']
    start_urls = ['https://www.douban.com/doulist/1264675/']

    def parse(self, response):
        item = DoubanbookItem() # 引入 item
        selector = scrapy.Selector(response) # 使用Selector选择器 xpath
        books = selector.xpath('//div[@class="bd doulist-subject"]') # 通过控制台查看路径，选择所有class="bd doulist-subject" 的div
        
        for each in books: # 遍历查看，筛选出需要的信息
            title = each.xpath('div[@class="title"]/a/text()').extract()[0] 
            rate = each.xpath('div[@class="rating"]/span[@class="rating_nums"]/text()').extract()[0]
            author = re.search('<div class="abstract">(.*?)<br', each.extract(), re.S).group(1) #作者这里采用的是正则，原因为xpath会获取到多余的信息 例如出版社等
            title = title.replace(' ', '').replace('\n', '') # 去除空格和换行
            author = author.replace(' ', '').replace('\n', '')
            item['title'] = title # 存入item
            item['rate'] = rate
            item['author'] = author
            #print('标题:' + title)
            #print('评分:' + rate)
            #print('作者:' + author)
            #print('')
            yield item # 回溯
            
            # 获取下一页地址（若存在下一页）原理同上，只是获取信息为链接
            nextPage = selector.xpath('//span[@class="next"]/link/@href').extract()
            if nextPage:
                next = nextPage[0]
                # print(next)
                yield scrapy.http.Request(next,callback=self.parse) # 重新获取该页面



```
 2. items.py
 

```python
import scrapy
class DoubanbookItem(scrapy.Item):
    title = scrapy.Field()
    rate = scrapy.Field()
    author = scrapy.Field()
```

3. settings.py

```python
BOT_NAME = 'doubanbook'

SPIDER_MODULES = ['doubanbook.spiders']
NEWSPIDER_MODULE = 'doubanbook.spiders'
USER_AGENT = 'Mozilla/5.0 (Windows NT 6.3; WOW64; rv:45.0) Gecko/20100101 Firefox/45.0'
FEED_URI = u'file:///d://doubanbooks.csv' #即存入d盘 doubanbooks.csv文件中
FEED_FORMAT = 'CSV'
ROBOTSTXT_OBEY = True
```

4. main.py
```python
from scrapy import cmdline
cmdline.execute("scrapy crawl dbbook".split())#运行爬虫
```
## 结束
1. 运行main.py文件爬虫即可运行
2. 运行结束后文件可能出现乱码，需要将其编码修改为UTF-8-Bom编码（个人采用notepad++）
3. 网页显示存在648本，实际爬出来只有631本
 
