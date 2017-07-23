---
title: Hexo主题添加评论模块
categories: 环境搭建
tags: Hexo
---

在Hexo主题里面添加评论模块，可以让浏览博客的读者评论我们的文章。接下来，将介绍通过第三方评论系统多说，在Hexo主题里面添加评论模块。
## 创建站点
首先我们需要在[多说](http://duoshuo.com/)里面创建一个站点。具体步骤如下：
1. 登陆多说（备注：不用注册，可以用微信扫描登陆），在首页选择“我要安装”
2. 在“创建站点”，根据自己的情况，配置相关信息，其中，“站点地址”项要求填个人博客网站地址，比如我这里填的是 https://lightmen.github.io/， “多说域名”项比较重要，自己选个名字，例如我这里填的是 lightmen。

## 配置Hexo主题_config.yml
这一步需要配置Hexo主题里面的_config.yml文件，添加 duoshuo_shortname 字段（先搜索，如果有就不用），设置如下：
```
duoshuo_shortname: your-duoshuo-shortname
```
备注：your-duoshuo-shortname，是你在创建多说站点是，在“多说域名”项填的名字。
比如我使用的是jacman，我需要打开 themes/jacman／_config.yml，加入如下一行：
```
duoshuo_shortname:    lightmen
```

至此，已经在Hexo主题里面添加完成评论模块了，接下来，可以在登陆自己的博客，检查在每篇文章后面，是否可以进行评论。如果可以，说明已经成功添加评论模块。

最后，如需取消某个页面的评论，在md文件的front-matter中增加：
```
comments: false
```




