# scrapy-file
used to spider data of state  pricespider.py部分
# -*- coding: utf-8 -*-
import scrapy
from anjukeSpider.items import AnjukespiderItem
from scrapy.loader import ItemLoader
from scrapy.http import Request
import re


class LianjiaSpiderSpider(scrapy.Spider):
    name = 'lianjia'
    allowed_domains = [
        'nj.fang.lianjia.com',
        # 'nj.lianjia.com',
    ]
    start_urls = [
        # 'https://nj.fang.lianjia.com/loupan/p_zsjpgkbitoj/?fb_expo_id=236108697135906816#around'
        'https://nj.fang.lianjia.com/loupan/pg1',
        # 'https://nj.lianjia.com/ershoufang/rs%E5%8D%97%E4%BA%AC/',
    ]

    # 爬取items文件内的内容
    def parse_item(self, response):
        """
        @url https://nj.fang.lianjia.com/loupan/p_zsjpgkbitoj/
        @returns items 1
        @scrapes housing_estate average_price age packing_spot block_number type developers pmg greening
        """
        loader = ItemLoader(item=AnjukespiderItem(), response=response)
        loader.add_xpath('housing_estate', '//h1[@class="DATA-PROJECT-NAME"]/text()')      #房屋名称
        loader.add_xpath('average_price', '//p[@class="jiage"]/span[@class="junjia"]/text()')   #房屋均价
        loader.add_xpath('average_price', '//p[@class="jiage"]/span[@class="yuan"]/text()')

        loader.add_xpath('age',          '//ul[@class="table-list clear"]/li[3][@class="odd"]/p[@class="desc-p clear"]/span[@class="label-val"]/text()')  #建筑年代
        loader.add_xpath('parking_spot', '//ul[@class="table-list clear"]/div[@class="clear"]/li[@class="odd"]//span[@class="label-val"]/text()') #停车位
        loader.add_xpath('household_number', '//ul[@class="table-list clear"]/li[7][@class="odd"]/p[@class="desc-p clear"]/span[@class="label-val"]/text()')  # 户数
        loader.add_xpath('type',         '//ul[@class="table-list clear"]/li[11][@class="odd"]/p[@class="desc-p clear"]/span[@class="label-val"]/text()')  #建筑类型

        loader.add_xpath('developers',   '//div[@class="box-loupan"]/p[4][@class="desc-p clear"]/span[@class="label-val"]/text()')  #开发商
        loader.add_xpath('pmg',          '//div[@class="box-loupan"]/p[5][@class="desc-p clear"]/span[@class="label-val"]/text()')  #物业公司

        loader.add_xpath('greening',     '//ul[@class="table-list clear"]/li[6][@class="even"]/p[@class="desc-p clear"]/span[@class="label-val"]/text()') #绿化率
        return loader.load_item()

    def parse(self, response):
        # 页面跳转
        total_items = int(response.xpath('//div[@class="page-box"]/@data-total-count').get())
        total_page = total_items // 10 + 1
        cur_page = re.search(r'pg(\d+)', response.url).group(1)
        cur_page = int(cur_page)
        next_page = cur_page + 1 if cur_page < total_page else total_page
        self.log('cur_page = %d' % cur_page)
        self.log('next_page = %d' % next_page)
        yield Request('https://nj.fang.lianjia.com/loupan/pg' + str(next_page))

        # 获取下一个房屋页面
        item_selector = response.xpath(
            '//li[@class="resblock-list post_ulog_exposure_scroll has-results"]\
            /a[@class="resblock-img-wrapper "]/@href').getall()
        for url in item_selector:
            yield Request('https://nj.fang.lianjia.com' + url, callback=self.parse_item)
