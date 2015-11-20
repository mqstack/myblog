title: 使用Hexo搭建博客
date: 2015-11-19 19:44:13
categories:
- Hexo
tags:
- Hexo
- Blog
---
##引言

Hexo 是一个快速、简洁且高效的博客框架，使用Markdown解析，生成静态网页。本文介绍Hexo的安装过程，及基本使用。

##安装

###准备

安装前需准备以下应用：

Git

Node.js

###安装Hexo

准备完毕后，执行如下命令安装Hexo

	$ npm install hexo-cli -g
	$ npm install hexo --save

###初始化

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

	$ hexo init <folder>
	$ cd <folder>
	$ npm install

新建完成后，文件夹目录如下：

	.
	├── _config.yml
	├── package.json
	├── scaffolds
	├── scripts
	├── source
	|   ├── _drafts
	|   └── _posts
	└── themes

其中：

_config.yml 全局配置文件

package.json 应用程序的信息

scaffolds 模版文件夹

scripts 脚本文件夹

source 资源文件夹是存放用户资源的地方

themes 存放皮肤的文件夹,默认皮肤为landscape

###安装插件

可按需选择插件安装

	$ npm install hexo-generator-index --save
	$ npm install hexo-generator-archive --save
	$ npm install hexo-generator-category --save
	$ npm install hexo-generator-tag --save
	$ npm install hexo-server --save
	$ npm install hexo-deployer-git --save
	$ npm install hexo-deployer-heroku --save
	$ npm install hexo-deployer-rsync --save
	$ npm install hexo-deployer-openshift --save
	$ npm install hexo-renderer-marked --save
	$ npm install hexo-renderer-stylus --save
	$ npm install hexo-generator-feed --save
	$ npm install hexo-generator-sitemap --save

更多插件：https://hexo.io/plugins/

至此，安装完毕，执行如下命令后可至[127.0.0.1:4000](http://127.0.0.1:4000)预览Hello World。

	hexo server

##使用

###配置

可以在 _config.yml 中修改大部份的配置，这里引用官方文档里描述。
https://hexo.io/zh-cn/docs/configuration.html

###写文章

可以直接在source/_posts文件夹下新建.md文件，或者使用如下命令新建文章。

$ hexo new post <title>

使用命令生成的文章，会套用scaffolds下面的模板，模板中可以自定义参数。

	参数			描述				默认值
	layout		布局	
	title		标题	
	date		建立日期			文件建立日期
	updated		更新日期			文件更新日期
	comments	开启文章的评论功能	true
	tags		标签（不适用于分页）	
	categories	分类（不适用于分页）	
	permalink	覆盖文章网址	

文章使用Markdown书写，写完后

	hexo server

就可以本地预览了。

###部署到Github

Github为个人和项目提供了页面展示的功能，以个人为例，创建一个以

>username.github.io

为名称的公开仓库。当部署完毕后，便可以访问个人主页。

>http://username.github.io


Hexo提供了方便的部署功能，可以将生成的静态页面推到git仓库。在＿config.yml文件中配置如下信息。

	deploy:
	  type: git
	  repo: https://github.com/username/username.github.io.git
	  branch: master

同时记得安装[hexo-deployer-git](#安装插件)插件,就可以将静态页面推到github中了。

	hexo g #生成页面
	hexo d #部署
  


##参考
https://hexo.io/zh-cn/docs/

http://wsgzao.github.io/post/hexo-guide/
