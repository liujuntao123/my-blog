---
title: 最简单易懂的node爬虫教程
date: 2018-05-17 19:22:16
tags:
- nodeJS 
- 爬虫
categories: nodeJS
---


最近了解了一下node实现网络爬虫的知识。于是我借鉴吸收之后，决定用request工具和cheerio，结合比较新的async异步语法，写了个浅显易懂的[node-demo](https://github.com/liujuntao123/crawler-demo)，供大家一起学习交流。
建议阅读本篇文章的同时，把这个demo克隆下来，更易于理解本文。
其实用node实现爬虫的基本原理很简单，就是通过request异步请求网页文件，然后再用cheerio解析这个文件的内容，然后根据自己所需爬取内容，最后借助node将爬取内容的结果封装成自己需要的形式，如json格式。

### 介绍一下request和cheerio这两个工具和async语法。
request是当前最简单的http请求工具，只需要简单的语句和配置就可以发送一个http请求，它的功能很强大，基本可以满足你所有的http功能的需求。这里使用的是bluebird开发的http-promise工具，它和request的区别是，使用request请求的响应结果是在它的回调函数里面，而http-promise的响应结果是以一个promise对象返回，这有利于async语句的书写，其他的，两者一样。
cheerio被称为是node端的jQuery，它可以把html文档解析成dom树的形式，然后使用类似于jQuery的元素选择器语法来选取元素。具体语法可以参见我的代码。
async是最新的也被认为是终极的js异步编程方式，如果对它还不熟悉，可以移步我的另一篇文章[JS异步编程学习笔记](http://www.jianshu.com/p/fb1a86555b6c)熟悉一下async。
### 用node写一个网络爬虫
下面是爬取知乎头条的步骤。首先，为了能在node里用上最新的es语法，使用严格模式：
```js
'use strict'
```
然后引入所需依赖，设定目标URI:
```js
let request = require('request-promise')
let $ = require('cheerio')
let express = require('express')
let app = express()
const URI = 'https://daily.zhihu.com/'
```
接下来开始写第一个接口，形式为直接访问'/':
```js
app.get('/', async (req, res) => {
  let result = await request(URI)//异步请求网页
  let data = []
  let elements = $('a', '.main-content .wrap .box', result)//解析出网页里的a元素
  elements.map((i, ele) => {
    let json = {}
    let $ele = $(ele)
    json.url = $ele.attr('href')//获取a元素的地址链接
    json.title = $ele.children().text()//获取标题
    data.push(json)
  })
  console.log(data)
  res.send(data)//把data数据返回给前台
})
```
再写一个获取文章详情的接口，访问路径为'/detail?path=关键字':
```js
app.get('/detail', async (req, res) => {
  let result = await request(URI + req.query.path)
  let data = {}
  let element = $('.content-inner .answer', result)//解析网页的指定元素
  data.avatar_url = $('.meta img.avatar', element).attr('src')//获取头像url
  data.author = $('.meta .author', element).text()//获取作者
  data.bio = $('.meta .bio', element).text()//获取作者签名
  data.content = $('.content', element).text()//获取文章内容
  res.send(data)//返回data给前台
})
```
最后启动服务：
```js
app.listen(8001, () => {//启动一个8001端口的server服务
  console.log('Listening on port 8001')
})
```
项目启动后，访问http://localhost:8001即可看到接口返回的数据

![image.png](http://upload-images.jianshu.io/upload_images/4050018-a9fc7a4998f626e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)