---
title: Hexo + Github Pages搭建个人博客
categories: Hexo
tags:
- Hexo
toc: true
---

# 关于Hexo与Github Pages
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

而Github Pages 是面向用户、组织和项目开放的公共静态页面搭建托管服 务，站点可以被免费托管在 Github 上，你可以选择使用 Github Pages 默 认提供的域名 github.io或者自定义域名来发布站点。

利用Github Pages和Hexo，我们可以搭建一个个人的博客网站，可以参考示例: [lightmen.github.io](https://lightmen.github.io/)。接下来我将分步骤介绍如何用GitHub Pages和Hexo搭建个人网站。

# 安装Hexo
安装Hexo很简单，但是在安装前，必须确保你的电脑中已安装下列两个应用程序:

- [Node.js](https://nodejs.org/en/)
- [Git](https://git-scm.com/)

(备注：在终端中输入 npm 命令可以验证电脑中是否安装了Node.js，输入git命令可以验证是否安装了Git，如果没有安装相应的程序，属于npm或者git命令，终端会提示找不到该命令)。

如果您的电脑中已经安装了上述两个程序，那么恭喜你！接下来你只需要在终端中输入如下命令就可以完成 Hexo 的安装：
 ``` bash
 $ npm install -g hexo-cli
 ```

 Hexo安装完成后，接下来在指定文件夹中新建需要的文件，我们可以假设有个hexo目录，然后在该目录里面创建一个blog文件：
 ``` bash
 $ cd hexo
 $ mkdir blog
 $ hexo init blog
 $ cd blog
 $ npm install
 ```

 新建完成后，在blog目录下面会生成如下文件：
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

这些文件的具体释义，参考 hexo.io官网中[配置](https://hexo.io/zh-cn/docs/configuration.html)一节, 这里不做过多解释。

继续操作，在终端中输入如下命令：
``` bash
$ hexo g
$ hexo s
```

提示: "Hexo is running at http://localhost:4000/. "  在浏览器中打开http://localhost:4000/, 你将会看到:
![landscape](http://note.youdao.com/yws/api/personal/file/WEB109a2e85adcbf21b729cc70c8df2ef85?method=download&shareKey=e3cf0fdd4ce77bfbc0dcf0b559f3a529)

至此，Hexo的本地配置基本完成。

# 配置Github Pages

配置Github Pages，首先你得有个github账号，如果没有，请进入 [GitHub网站](https://github.com/)注册，登陆进去。

登陆网站后，在网站右上角的"+"图标那里，点击 "[New repository](https://github.com/new)", 创建一个仓库。仓库名必须为你的用户名＋ github.io，比如我的github用户名为：lightmen， 那么我创建的仓库名就必须为：lightmen.github.io。 创建好后，可以试着在浏览器里面输入 https://yourname.github.io/, 确认是否可以正常访问(当然，刚刚创建的仓库，里面什么内容都没有，浏览器会显示空白)。

另外，如果你是第一次使用github，你必须在本地配置好你的git环境，确保你可以往github上面的仓库传文件。这里，不做过多介绍，网上一堆可参考的教程。

# 部署到Github Pages

这一节的内容可以参考Hexo官方文档中[部署部分](https://hexo.io/zh-cn/docs/deployment.html)，我这里分三个步骤讲解。

1. 运行如下命令，安装[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)。
``` bash
$ npm install hexo-deployer-git --save
```
2. 然后打开Hexo主文件夹(备注：hexo init命令设置的那个文件夹）中的_config.yml, 设置其中的deploy参数，以我的为例，设置的内容如下：
```
deploy:
  type: git
  repo: https://github.com/lightmen/lightmen.github.io.git
  branch: master
```

3. 然后在当前目录下，打开终端，输入下面命令，将网站部署到服务器上：
``` bash
$ hexo d
```
随后，按照提示输入github的账号和密码，上传文件。网站部署到服务区后，就可以通过 https://yourname.github.io/ 来访问自己刚刚上传的网站。

# 添加新文章

在网站中添加文章，通过运行以下命令创建POST：
``` bash
$ hexo n “your post name”
$ hexo g
$ hexo s
```
然后可以在浏览器查看修改的内容。

打开Hexo主文件夹下的source 目录，所有的文章都会以md形式保存在_post文件夹中，只要在 _post文件夹中新建md类型的文档，就能在执行hexo g的时候被渲染。 新建的文章头需要添加一些yml信息，如下所示：
```
title: hello-world   //在此处添加你的标题。
date: 2017-03-18 22:15:29   //在此处输入你编辑这篇文章的时间。
categories: Exercise   //在此处输入这篇文章的分类。
toc: true  //在此处设定是否开启目录，需要主题支持。
```

# 进阶
如果成功完成了上述的全部步骤，恭喜你，你已经搭建了一个最为简单且基础的博客。但是这个博客还非常简单， 没有个人的定制，操作也比较复杂。我们还可以进行一些设置主题、更换域名等操作，调整自己的博客，这部分内容待续。

参考文档：

- [使用Hexo搭建个人博客](http://www.jianshu.com/p/f4dce0e76886)

- [史上最详细的Hexo博客搭建图文教程](https://xuanwo.org/2015/03/26/hexo-intor/)

- [手把手教你使用Hexo + Github Pages搭建个人独立博客](https://linghucong.js.org/2016/04/15/2016-04-15-hexo-github-pages-blog/)

- [Hexo使用指南](http://www.jianshu.com/p/84a8384be1ae)


