---
title: 抓取网页内容（爬虫）一些方法介绍
tags:
- spider
- crawler
- selenium
- puppeteer
- JS
- JavaScript
---

## 抓取网页内容的一般流程
1. 发HTTP请求
2. 解析请求返回的结果
3. 得到的结果持久化

## 常用的抓取网页内容的方法和工具
### 命令行 
* cURL

### PHP:
* 原生Client URL 库（cURL)
* 类库，比如：[Requests](http://requests.ryanmccue.info/), [Guzzle](http://guzzlephp.org/)
* 爬虫开发框架：[phpspider](https://doc.phpspider.org/)

### Python
* 类库，比如：[Requests](http://docs.python-requests.org/)
* 爬虫开发框架：[Scrapy](https://scrapy.org/)

### NodeJS
* [curl-request](https://www.npmjs.com/package/curl-request)
* 爬虫开发框架：[node-crawler](http://nodecrawler.org/)

## 可能遇到的问题和困难
* 登录：保存Cookie, 认证
* 被屏蔽
* 验证码
* 执行JS
* 其他

## 另外一种思路
利用界面自动化测试工具进行抓取，简单点讲就是：程序唤起浏览器后模拟用户行为进行操作
<!--more-->

适用场景：
* 登录验证比较麻烦、或者验证码无法破解
* 目标地址有强大的反爬机制
* 一次性操作

### 目前的一些方案
### [Selenium](https://selenium-python.readthedocs.io/) 
[demo](https://github.com/ayonliu/exercise/tree/master/selenium/amazon_login.py)

### [Puppeteer](https://developers.google.com/web/tools/puppeteer/)
[demo](https://github.com/ayonliu/exercise/tree/master/puppeteer/amazon.js)
![screenshot](https://raw.githubusercontent.com/ayonliu/exercise/master/images/amazon-affilate.png)

### [PhantomJS](http://phantomjs.org/) （项目已暂停）

## 更临时的一种方法

适用场景：临时性的一次性抓取

### [Fetch API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch) 
[demo](https://github.com/ayonliu/exercise/blob/master/js/e-commerceaffiliates.js)

