---
title: 使用Travis CL自动部署Hexo
date: 2019-04-02 14:03:47
tags: 
  - GithubPage
  - Hexo
  - TravisCI
categories: 
  - Hexo
---

前面我们已经搭建好了一个Hexo博客，但是有一个问题是，我们必须在本地搭建好的环境下写博客，然后再生成静态文件发布到Github或者coding上，这样实在是很不方便，我们希望能随时随地修改我们的博客或者写文章，并且能方便地发布出去。这里，我们就要使用到Travis CI的持续集成。

<!--more-->
### 1. 持续集成简介
持续集成(Continuous Integration)是一种软件开发实践。在持续集成中，团队成员频繁集成他们的工作成果，一般每人每天至少集成一次，也可以多次。每次集成会经过自动构建（包括自动测试）的检验，以尽快发现集成错误。
### 2. Travis CI简介
[Travis CI](https://api.travis-ci.org)是目前新兴的开源持续集成构建工具，它与jenkins，GO的明显区别在于采用yaml格式，同时它是在线服务，不像jenkins需要搭建本地环境。目前大多数的Github开源项目都已经移入到Travis CL的构建队列中，据说Travis CL每天运行超过4000次完整构建。

### 3. 自动部署流程
Travis CI与Github集成的程度非常高，对于Github上的开源项目，可以免费在Travis上构建。下图是利用Travis构建Hexo的业务时序图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1qkoxq5cuj30qo0kp41l.jpg)
步骤如下：

- 首先在Github的博客仓库新建一个blog-source分支，然后把博客的整个源代码托管到这个分支
- 每当我们在本地修改博客或者写好博文，将修改push到blog-source分支
- Travis上可以对这个项目的blog-source分支设置钩子，每当检测到push操作的时候，就去blog-source分支clone代码到Travis
- Travis执行构建脚本
- Travis把构建结果push到博客仓库的master分支和Coding的仓库

根据上述的自动部署流程，我们唯一需要做的是push我们的博文到blog-source分支，其他的事情交给Travis。
简要说一下如何将博客源代码提交到blog-source分支下。
1. 首先在本地博客根目录下，执行git init将博客文件夹初始化为一个本地git仓库。
2. 然后配置用户名和邮箱
- git config user.name "github用户名"
- git config user.email "github邮箱"
3. 本地新建一个blog-source分支，并切换到该分支下
- git checkout -b blog-source
4. 提交本地文件
- git add .
- git commit -m "blog-source init"
5. 关联远程仓库
- git remote add origin https://github.com/iyuwe6/iyuwe6.github.io.git
6. 将本地仓库blog-source分支提交到远程仓库的blog-source，注意这里会在远程仓库新建一个blog-source分支，并与本地的blog-source分支关联起来，以后本地的blog-source分支都会提交到对应的远程仓库blog-source分支
- git push origin blog-source:blog-source
这样，我们就将博客源代码提交到了github上。
### 4. 配置Travis
#### 1. 注册登录Travis
[Travis CI](https://api.travis-ci.org)不需要单独注册，直接使用Github账号登录即可。点击左边的ync accoun，会同步你的Github账号，然后右边选择你要Activate(激活)的仓库。
#### 2. 配置Travis CI
点击进入勾选的仓库，进入设置页面，如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1qe8osgolj310r0hct9m.jpg)
这里设置构建的行为以及配置环境变量等，比如后面我们需要在这里添加Github的Token和Coding的Ttoken，是我们有权限对仓库进行一些操作。

#### 3. 生成Token
我们需要给予Travis CI对我们仓库进行相关操作的权限，所以我们需要生成相应的Token添加到Travis CI中。
首先，生成Github的Token，进入Github的设置中，如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1qeih0zrij30l30d0q4l.jpg)
先给Token取一个名字，然后设置一些权限，其中红框内的权限是必须的，其他可以按需添加，如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1qel7ctcaj30kv0x00u6.jpg)
生成Token，将生成的Token值复制添加到Travis CI仓库设置里的环境变量，并将Display value in build log设置为OFF，关闭变量显示，避免Token泄露。
然后，生成Coding的Token(令牌)，如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1qerj27ogj30yi0eh0su.jpg)
先给令牌取一个名字，然后设置一些权限，如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1qeshbh4nj30tb0drwew.jpg)
生成令牌，复制令牌值复制到Travis CI仓库设置里的环境变量中。
#### 4. 新建.travis.yml文件
```yaml
language: node_js  #构建环境
node_js:
- stable  #选择当前稳定版本
branches:
  only:
  - blog-source   #这里替换你要监听的分支
cache:
  directories:
  - node_modules  #把node_modules文件夹放入缓存，节约每次构建的时间
before_install:
- npm install hexo-cli -g #安装Hexo脚手架工具
install:
- npm install  #安装package.json中的依赖
script:
- hexo clean
- hexo generate
after_success: #script阶段成功后执行，构建失败不会执行
  - cd ./public
  - git init
  - git config user.name "github用户名"
  - git config user.email "github邮箱"
  - git add .
  - git commit -m "TravisCI 自动部署"
  # Github Pages
  - git push --force --quiet "https://${CI_TOKEN}@${GH_REF}" master:master 
  # Coding Pages
  - git push --force --quiet "https://coding用户名:${CO_TOKEN}@${CO_REF}" master:master

env:
 global:
   # Github Pages
   - GH_REF: github.com/lanpangzhi/lanpangzhi.github.io  这里替换你的github仓库地址
   # Coding Pages
   - CO_REF: git.coding.net/bule/bule.coding.me.git  这里替换你的coding仓库地址
```
将这个文件提交到blog-source根目录下，以后travlis-ci就会自动构建了。以后只要将博客提交到blog-source，travis就会帮你自动构建并部署。看到下面这张图就表示成功了。
![](https://ws1.sinaimg.cn/large/006aBttAly1g1qfb3yx0hj30yy08xq37.jpg)

### 参考文档
1. [基于 Hexo 的全自动博客构建部署系统](https://kchen.cc/2016/11/12/hexo-instructions/#Travis-CI-%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90)
2. [使用travis-ci自动部署Hexo到github和coding](https://juejin.im/post/5afe61f5f265da0b8d422a3e)
3. [使用Travis CI自动部署Hexo到GitHub](https://dmego.me/2017/10/13/deylpoy-hexo-with-TravisCI.html)



