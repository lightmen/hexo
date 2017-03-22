---
title: Markdown图片居中的方法
categories: Hexo
tags: 
- Hexo
- Markdown
toc: false
---

有两种方法可以使Markdown中插入的图片居中：
法一：
```
<div style="text-align:center" markdown="1">
![image description](image url)
</div>
```


示例：
<div style="text-align:center" markdown="1">
![SD Memory Card Architecture](http://on61oh42c.bkt.clouddn.com/SD-Memory-Card-Architecture.png)
</div>


法二：
```
<img src="", alt= "" class="img-center">
```
这里，在我的jacman主题下测试有效，img-center定义在我的Hexo主目录下的themes/jacman/source/css/_partial／article.styl文件里面。img-center 定义如下：
```
.img-center
  display block 
  margin auto
```

或者
```
<img src="" style="display:block;margin:auto"/>
```

示例：

<img src="http://on61oh42c.bkt.clouddn.com/SD-Memory-Card-Architecture.png", alt="SD Memory Card Architecture" style="display:block;margin:auto">

