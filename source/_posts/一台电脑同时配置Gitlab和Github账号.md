---
title: 一台电脑同时配置Gitlab和Github账号
date: 2018-09-26 21:27:08
tags: 
  - git
categories: 
  - git
---

在公司的电脑上配置好了公司的Gitlab账号，现在想在公司的电脑上同时配置个人的Github账号，方便对自己的Github进行管理。那么如何进行相关的设置呢？如何避免公司账号与个人账号独立管理，避免XX公司程序员将公司源码传到Github上导致代码泄露的悲剧呢？这篇文章给你答案。


<!--more-->

### 问题描述：
首先，假定你已经在电脑上配置好了公司的Gitlab账号，进行了用户名和邮箱的全局配置，并生成了SSH key，添加到了Gitlab中。
目前，你的电脑上默认情况下的各种本地仓库都是用Gitlab的配置。也就是说默认情况下，你的本地仓库是在你的Gitlab账号名下，只能与你远程的Gitlab账号下管理的仓库进行正常的关联。当然你的这个仓库也能关联其他人的远程库，但是你是没法进行push的，因为你的SSH key不在对方的账户列表里，它拒绝了你的操作。
我们要达成的目的就是：

  1. 分别生成Gitlab/Github的私钥和公钥，并能让git正确区分它们
  2. 本地仓库能正确到对应的远程仓库

### 解决方案：

  1. 生成私钥/公钥时，密钥文件命名避免重复
  2. 添加config文件，设置Host对应其HostName但密钥不同
  3. 取消git全局用户名/邮箱设置，为每个仓库独立设置用户名/邮箱

### 操作步骤：
1.由于Gitlab账号已经配置好SSH key，这里就不再赘述。现在我们需要配置好Github账号的SSH key。右键打开Git Bash Here，执行

    ssh-keygen -t rsa -C "注册的github邮箱"
注意对生成的密钥文件进行重命名id_rsa_github，同样可以不设置密码。这样就生成了id_rsa_github.pub和id_rsa_github连个文件，将它们放到c:/Users/用户名/.ssh/目录下。然后将生成的SSH key添加到Github账户中。
2.在.ssh/目录下新建config文件，进行如下配置：

    #gitlab
    Host gitlab.com        #自定义的host名称，建议与HostName相同
    HostName gitlab.com    #主机名
    User git               #登录用户名
    IdentityFile ~/.ssh/id_rsa    #证书文件路径
    #github
    Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github

3.测试SSH连接

    $ ssh -T git@gitlab.com
    Welcome to GitLab, 谢东!
    
    $ ssh -T git@github.com
    The authenticity of host 'github.com (192.30.253.112)' can't be established.
    RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'github.com,192.30.253.112' (RSA) to the list of known hosts.
    Hi iyuwe6! You've successfully authenticated, but GitHub does not provide shell access.

出现上述情形，即表明confgi文件配置正确。

4.取消全局用户名/邮箱设置，并进入相应的本地仓库进行单独设置

    # 取消全局 用户名/邮箱 配置
    git config –global –unset user.name
    git config –global –unset user.email
    # 单独设置每个repo 用户名/邮箱
    git config user.email “xxxx@xx.com”
    git config user.name “xxxx”

当然你也可以不取消全局设置，那么本地仓库默认就是与你默认配置的账号进行关联。若要新建由其他账号管理的本地仓库，则需要在初始化本地仓库时单独设置用户名和邮箱。