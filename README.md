��ش����Ѿ��޸ĵ���----2017-3-21
ʵ�֣�ǧͼ���ϸ���ͼƬ����ȡ
��������20Сʱ����ȡ��Լ162000��ͼƬ��һ��49G,����ٶ���
spider.py
# -*- coding: utf-8 -*-
import scrapy
from scrapy.http import Request
from second.items import SecondItem
jishu=1
class QiantuSpider(scrapy.Spider):
    name = "qiantu"
    allowed_domains = ["58pic.com"]
    start_urls = (
        'http://www.58pic.com/new/',
    )

    #������ȡ�����url
    def parse(self, response):
        urldata=response.xpath('//div[@class="classify clearfix"]/a/@href').extract()  #һ����62�����
        print urldata
        for i in range(0,len(urldata)):
            thisurldata=urldata[i]
            yield Request(url=thisurldata,callback=self.next1)
    #��ȡ�����ÿһҳ��url
    def next1(self,response):
        thisurl=response.url
        pagelist=response.xpath("//div[@id='showpage']/a/text()").extract()
        if(len(pagelist)>=3):
            page=pagelist[-3]
            for j in range(1,int(page) + 1):
                pageurl=thisurl.replace('day-1','day-'+str(j))
                yield Request(url=pageurl,callback=self.next2)
        else:
            pass
    #��ȡÿһ��ͼƬ����ϸ��url
    def next2(self,response):
        url_erery=response.xpath("//a[@class='thumb-box']/@href").extract()   #��һ��
        if url_erery==[]:       #����ͼ���ӵ���ҳ�ṹ������
            url_erery=response.xpath('//div[@class="list fl"]/a/@href').extract()   #�ڶ���
        #print url_erery
        for k in url_erery:
            yield Request(url=k,callback=self.getimg)
    #�õ�����ͼ��url
    def getimg(self,response):
        global jishu
        print("��ʱ����ȡ��"+str(jishu)+"��ͼƬ---"+response.url+"----")
        item=SecondItem()
        #��ϸҳ�����ֽṹ
        item['title']= response.xpath('//div[@class="show-area-pic"]/img/@title').extract() #��һ��
        item['url']= response.xpath('//div[@class="show-area-pic"]/img/@src').extract()
        if item['title']==[]:
            item['title']= response.xpath('//div[@class="show-area-pic"]/div/img/@title').extract()#�ڶ���
        if item['url']==[]:
            item['url']= response.xpath('//div[@class="show-area-pic"]/div/img/@src').extract()
        if item['title']==[]:
            item['title']= response.xpath('//div[@class="pic-show hidden"]/div/img/@title').extract() #������
        if item['url']==[]:
            item['url']= response.xpath('//div[@class="pic-show hidden"]/div/img/@src').extract()
        #print item['title'][0],item['url'][0]
        jishu+=1
        yield item
item.py
# -*- coding: utf-8 -*-
import scrapy
class SecondItem(scrapy.Item):
    url=scrapy.Field()
    title=scrapy.Field()
setting.py
# -*- coding: utf-8 -*-

# Scrapy settings for second project
#
# For simplicity, this file contains only settings considered important or
# commonly used. You can find more settings consulting the documentation:
#
#     http://doc.scrapy.org/en/latest/topics/settings.html
#     http://scrapy.readthedocs.org/en/latest/topics/downloader-middleware.html
#     http://scrapy.readthedocs.org/en/latest/topics/spider-middleware.html

BOT_NAME = 'second'

SPIDER_MODULES = ['second.spiders']
NEWSPIDER_MODULE = 'second.spiders'


# Crawl responsibly by identifying yourself (and your website) on the user-agent
USER_AGENT = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.22 Safari/537.36 SE 2.X MetaSr 1.0'

# Obey robots.txt rules
ROBOTSTXT_OBEY = False

# Configure maximum concurrent requests performed by Scrapy (default: 16)
CONCURRENT_REQUESTS = 20

# Configure a delay for requests for the same website (default: 0)
# See http://scrapy.readthedocs.org/en/latest/topics/settings.html#download-delay
# See also autothrottle settings and docs
#DOWNLOAD_DELAY = 1
# The download delay setting will honor only one of:
#CONCURRENT_REQUESTS_PER_DOMAIN = 16
#CONCURRENT_REQUESTS_PER_IP = 16

# Disable cookies (enabled by default)
COOKIES_ENABLED = False  #����¼cookie

# Disable Telnet Console (enabled by default)
#TELNETCONSOLE_ENABLED = False

# Override the default request headers:
#DEFAULT_REQUEST_HEADERS = {
 #  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
  # 'Accept-Language': 'en',
   # 'User-Agent':'',
#}

# Enable or disable spider middlewares
# See http://scrapy.readthedocs.org/en/latest/topics/spider-middleware.html
#SPIDER_MIDDLEWARES = {
#    'second.middlewares.MyCustomSpiderMiddleware': 543,
#}

# Enable or disable downloader middlewares
# See http://scrapy.readthedocs.org/en/latest/topics/downloader-middleware.html
#DOWNLOADER_MIDDLEWARES = {
#    'second.middlewares.MyCustomDownloaderMiddleware': 543,
#}

# Enable or disable extensions
# See http://scrapy.readthedocs.org/en/latest/topics/extensions.html
#EXTENSIONS = {
#    'scrapy.extensions.telnet.TelnetConsole': None,
#}

# Configure item pipelines
# See http://scrapy.readthedocs.org/en/latest/topics/item-pipeline.html
ITEM_PIPELINES = {
    'second.pipelines.SecondPipeline': 300,
}

# Enable and configure the AutoThrottle extension (disabled by default)
# See http://doc.scrapy.org/en/latest/topics/autothrottle.html
#AUTOTHROTTLE_ENABLED = True
# The initial download delay
#AUTOTHROTTLE_START_DELAY = 5
# The maximum download delay to be set in case of high latencies
#AUTOTHROTTLE_MAX_DELAY = 60
# The average number of requests Scrapy should be sending in parallel to
# each remote server
#AUTOTHROTTLE_TARGET_CONCURRENCY = 1.0
# Enable showing throttling stats for every response received:
#AUTOTHROTTLE_DEBUG = False

# Enable and configure HTTP caching (disabled by default)
# See http://scrapy.readthedocs.org/en/latest/topics/downloader-middleware.html#httpcache-middleware-settings
#HTTPCACHE_ENABLED = True
#HTTPCACHE_EXPIRATION_SECS = 0
#HTTPCACHE_DIR = 'httpcache'
#HTTPCACHE_IGNORE_HTTP_CODES = []
#HTTPCACHE_STORAGE = 'scrapy.extensions.httpcache.FilesystemCacheStorage'
pipelines.py
# -*- coding: utf-8 -*-
import urllib
import random
class SecondPipeline(object):
    def process_item(self, item, spider):
        try:
            title=item['title'][0].encode('gbk')
            file="E:/tupian/"+str(title)+str(int(random.random()*10000))+".jpg"
            urllib.urlretrieve(item['url'][0],filename=file)
        except Exception,e:
            print e
            pass
        return item


�ʼǣ�
һ��scrapyͼƬ���湹��˼·
     1.������վ
     2.ѡ����ȡ��ʽ�����
     3.����������Ŀ ������items.py
     4.��д�����ļ�
     5.��дpipelines��setting
     6.����

����ǧͼ���ѵ㣨http://www.58pic.com/��
     1.Ҫ��ȡȫվ��ͼƬ
     2.Ҫ��ȡ�����ͼƬ------�ҳ������ַ����
     3Ҫ����Ӧ�ķ��������------��ģ�������������¼cookie�ȣ�ֻҪ��Ӧע��ȥ������COOKIES_ENABLED = False

����ɢ��֪ʶ
   1.from scrapy.http import Request  #�ǻص�������Request(url=...,callback=...)
   2.xpath��//��ʾ��ȡ���з��ϵĽڵ�



