---
title: typecho博客配置
date: 2018-09-10 21:24:08
tags: 
  - typecho
categories: 
  - typecho
---

本篇文章用于记录typecho博客的配置过程。


<!--more-->


### Typecho自定义后台路径 ##
在安装好Typecho后，默认的后台路径是yourdomain.com/admin/,为了提高安全性，防止其他人可以获取到你的后台登录URL。
操作步骤：
1.重命名admin文件夹；2.打开根目录下的 config.inc.php 文件，找到
    /** 后台路径(相对路径) */
    define('__TYPECHO_ADMIN_DIR__', '/admin/');
将admin修改成你重命名后的名称。
### Typecho关闭meta中的版本信息 ##
meta标签默认显示Typecho的版本信息和模板的名称，可以关闭掉或者修改成其他信息。
操作步骤：
1.打开模板下的header.php文件，找到

    <?php $this->header(); ?>
在header()里加入参数（去掉这两项参数）：

    <?php $this->header("generator=&template="); ?> 
参数：

 - keywords:关键词
 - keywords：关键词
 - description：描述、摘要
 - rss1：feed rss1.0
 - rss2：feed rss2.0
 - atom：feed atom
 - generator：程序版本等
 - template：模板名称
 - pingback：文章引用
 - xmlrpc：离线写作
 - wlw：m$的离线写作工具
 - commentReply：评论回复

说明：
等号=为空则不输出该项目，各个参数之间使用&连接。
### Typecho关闭侧边栏登录入口 ##
登录后台首页,选择**控制台-外观-设置外观**，把**显示其他杂项**去掉勾选，**保存设置**。
### Typecho开启伪静态 ##
Typecho默认的链接会在域名后加上index.php，对于强迫症很不习惯，只需两步，就可以拯救强迫症了。提示：本博客服务器后台安装有宝塔面板，可以直接在面板中操作。
操作步骤：
1.进入宝塔面板，点击**网站-站点设置-站点修改-伪静态**，选择Typecho,点击，自动生成一段代码，保存，就开启了伪静态。如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o2x6dcyjj30sz0dpt9r.jpg)

2.进入Typecho后台，进入**设置-永久链接**，启用地址重写功能，自定义文章路径，保存设置。如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o2xk6533j30pk0ayjrx.jpg)

大功告成，你的博客URL就去掉了index.php。

