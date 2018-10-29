---
title: hexo博客SEO优化
date: 2018-10-29 09:36:31
categories: Study
tags: seo
---

最近自己建了一个自己的博客，但是无奈的是在百度搜索我的博客网站却怎么也搜索不到，有啥办法呢，自己学着对网站做下SEO吧。下面是我在网上搜集的SEO方法，记录下学习过程分享给大家。

### 添加站点地图
*网站地图，又称站点地图，它就是一个页面，上面放置了网站上需要搜索引擎抓取的所有页面的链接（注：不是所有页面）。*
step1: 安装sitemap和百度版本的sitemap
```
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
```
step2: 打开配置文件`_config.yml`，添加如下内容
```
sitemap:
path: sitemap.xml
baidusitemap:
path: baidusitemap.xml
```
### 添加robots配置
*该配置可以管理我们想要搜索引擎爬取的内容，Allow表示允许被访问的，Disallow是不允许的意思。注意后面两个Sitemap就是网站地图了。而网站地图前面说了是给爬虫用的。这里配置在robots中。可参考如下配置*
```
User-agent: *
Allow: /
Allow: /home/
Allow: /archives/
Allow: /about/
Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/
Sitemap: http://yourwebsite/sitemap.xml
Sitemap: http://yourwebsite/baidusitemap.xml
```

### 进入搜索引擎管理平台更新robots

Google Search Console: https://www.google.com/webmasters/tools/home?hl=zh-CN

百度站长平台: http://zhanzhang.baidu.com/

### Url持久化
Hexo默认生成的文章地址路径是【网站名称／年／月／日／文章名称】这种路径，这种路径对搜索引擎是不友好的，所以我们要把他们进行优化。优化方法如下：
方法1、打开_config.yml配置文件，找到permalink,修改为如下
`permalink:   /articles/:title.html`
方法2、安装三方插件`npm install hexo-abbrlink --save`
在配置_config.yml中添加
```
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32
  rep: hex    # 进制：dec(default) and hex
```

### 对文章中的超链接添加nofollow
*nofollow 是HTML页面中a标签的属性值,它能告诉搜索引擎"不要追踪此网页上的链接或不要追踪此特定链接"*
这个需要在主题模板中去添加的，比如pug模板
```
a(href=theme.links[i].url title=theme.links[i].title target='_blank' rel='external nofollow') #{theme.links[i].title}
```

原文链接：http://www.apkbus.com/blog-889706-76401.html


