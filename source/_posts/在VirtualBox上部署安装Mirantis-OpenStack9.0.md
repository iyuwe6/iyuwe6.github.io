---
title: 在VirtualBox上部署安装Mirantis OpenStack9.0
date: 2018-10-08 20:26:10
tags: 
  - OpenStack
categories: 
  - OpenStack
---

### 1 介绍
#### 1.1 关于Mirantis
Mirantis，一家很牛逼的openstack服务集成商，他是社区贡献排名前5名中唯一一个靠软件和服务吃饭的公司（其他分别是Red Hat, HP, IBM, Rackspace）。相对于其他几个社区发行版，Fuel的版本节奏很快，平均每两个月就能提供一个相对稳定的社区版。
<!--more-->
#### 1.2 关于Fuel
Fuel是一个为openstack端到端”一键部署“设计的工具，是OpenStack自动化、工业级部署方案，其功能涵盖自动的PXE方式的操作系统安装，DHCP服务，Orchestration服务和puppet配置管理相关服务等，此外还有openstack关键业务健康检查和log实时查看等非常好用的服务。详细可查阅[Fuel](https://wiki.openstack.org/wiki/Fuel "Fuel")。
#### 1.3 Fuel的优点
总结一下，Fuel有以下几个优点：
- 节点的自动发现和预校验
- 配置简单、快速
- 支持多种操作系统和发行版，支持HA部署 × 对外提供API对环境进行管理和配置，例如动态添加计算/存储节点 × 自带健康检查工具 × 支持Neutron，例如GRE和namespace都做进来了，子网能配置具体使用哪个物理网卡等。
### 2 安装准备
硬件要求：
- 启用虚拟化技术支持，开启本地机器BIOS设置里的虚拟化技术支持相关选项，这个会很大程度影响你的虚拟机性能。
- 最低硬件配置： CPU：双核2.6GHZ+；内存：8G+；磁盘：80G+
- 在虚拟机设置-系统-硬件加速中开启VT-x和AMD-V，以便支持intel和AMD的CPU开启硬件虚拟化.如果不开启很可能会部署失败
  安装包：
- Oracle VM VirtualBox
- Oracle VM VirtualBox Extension Pack，必须安装VirtualBox扩展包否则无法使用PXE功能
  [点此下载VirtualBox和扩展包](https://www.virtualbox.org/wiki/Downloads "点此下载VirtualBox和扩展包")
- [Mirantis OpenStack](https://www.mirantis.com/software/openstack/download/ "Mirantis OpenStack")
- bootstrao 链接：https://pan.baidu.com/s/1VEuc_3zZxBQV7aS_pxUnnA 提取码：a26r
- mirrors 链接：https://pan.baidu.com/s/1bZqx3z2bRdmjl6zRzRWTxg 提取码：s7ow
- xshell
- xftp
### 3 部署过程
总体流程为：
1. 配置相关网络
2. 安装Fuel服务器（fuel-master)
3. 安装OpenStack
#### 3.1配置相关网络
##### 安装VirtualBox和扩展包
首先在本地电脑上安装好VirtualBox，安装过程很简单，不再赘述。安装好VirtualBox后，安装扩展包，在VirtualBox管理器中点击管理-全局设定-扩展，添加并安装扩展包。如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1jo4r39j30dt09umxf.jpg)
##### 配置相关网络
1.打开VirtualBox管理器的全局工具-主机网络管理器
创建三个Host-Only网络，如图所示：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1lauyr3j30j1096jro.jpg)
所有网络取消DHCP。
##### 创建并配置三个节点
三个节点分别为Fuel部署节点(fuel-master)，控制节点(fuel-controller)，计算节点(fuel-compute)。
###### Fuel部署节点(fuel-master)
操作系统：Ubuntu(64-bit)
内存：4G
硬盘容量：100G
网卡：Adapter 混杂模式：全部允许
Adapter # 2 混杂模式：全部允许
Adapter # 3 混杂模式：全部允许
网卡启动：否
如图所示：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1ls583mj30ju0c23yx.jpg)
###### 控制节点(fuel-controller)
控制节点需要安装较多的服务，需要较大的内存和硬盘，同时需要选择网卡启动。
操作系统：Ubuntu(64-bit)
内存：5G
硬盘容量：120G
网卡：
Adapter 混杂模式：全部允许
Adapter # 2 混杂模式：全部允许
Adapter # 3 混杂模式：全部允许
网卡启动：是
如图所示：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1m8x2nbj30k40bw74z.jpg)
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1mo3kp0j30jy0bk753.jpg)
###### 计算节点(fuel-compute)
计算节点启动服务较少，配置可以稍低一点，但同样需要选择网卡启动。
操作系统：Ubuntu(64-bit)
内存：4G
硬盘容量：100G
网卡：
Adapter 混杂模式：全部允许
Adapter # 2 混杂模式：全部允许
Adapter # 3 混杂模式：全部允许
网卡启动：是
#### 3.2 安装Fuel服务器（fuel-master)
1.启动fuel-master节点虚拟机
2.加载Mirantis OpenStack.iso
3.进入Fuel安装选择界面，选择第一项，然后系统会自动加载和安装Fuel。如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1n7bc63j30dw08mn27.jpg)
一段时间后进入Fuel-Menu界面，在这里可以修改默认密码和一些设置。
在Network Setup中，可以设置Admin(PXE)网络，需要注意的是，不能在fuel-master节点部署完成后修改，因为修改了会导致无法利用PXE部署和管理OpenStack环境。
修改Admin(PXE)网络的步骤：
1.进入Network Setup，选择eth0，将Enable interface改为No，点击Apply。
2.选择另一个网卡，作为Admin(PXE)网络，例如：eth1。
3.将Enable interface改为Yes。
4.配置eth1的网络参数，默认eth0时的网络参数为：
IP address 10.20.0.2
Netmask 255.255.255.0
Gateway 10.20.0.1
5.点击Apply。fuel-master将会用eth1作为Admin(PXE)网络。
6.进入PXE Setup，设置DHCP。默认eth0时的DHCP参数为：
DHCP pool start 10.20.0.3
DHCP pool end 10.20.0.254
DHCP gateway 10.20.0.2
7.最后，点击Check验证配置是否正确。
在Network Setup设置中，你还可以修改登录Fuel web UI的网络。
修改步骤为：
1.进入Network Setup，选择一个未分配的网卡,将Enable interface改为Yes。
2.启用DHCP，将IP address,netmask,gateway置为空。
3.点击Apply。

在Bootstrap Image中选择Skip building bootstrap image，如果不选择的话，默认Fuel会从国外获取镜像源，速度会很慢，很大概率会导致安装失败。后面我们会部署本地镜像源，稍后说明。如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1nq50euj30dw04tt97.jpg)
然后在Quit Setup中选择Save and Quit。保存修改的设置并退出。
经过大概两个小时的漫长等待，安装完成，进入了如图所示的Fuel登录界面。
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1o4837nj30dw07wdgr.jpg)
在浏览器打开https://10.20.0.2:8443 测试一下是否安装成功。显示如下界面即表示安装成功。
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1oesxvrj30dw0almy5.jpg)
此处建议保存快照，以便后续恢复。

#### Fuel配置本地镜像
bootstrap用于统一部署节点的引导操作系统，为节点安装操作系统。
mirrors保存有节点要安装的操作系统
##### bootstrap
1.将下载下来的bootstrap文件夹解压，用xftp将bootstrap文件夹上传到/var/www/nailgun/下。覆盖原来的bootstrap文件夹
2.用xshell登录到fuel-master节点，使用默认账号密码。
3.执行以下命令创建bootstarp镜像
```shell
fuel-bootstrap activate d01c72e6-83f4-4a19-bb86-6085e40416e6
fuel-bootstrap list
```
##### mirrors
1.将下载下来的mirrors文件夹解压，用xftp将mirrors文件夹上传到/var/www/nailgun/下。
2.执行以下命令创建ubuntu镜像。PS：报错忽略即可。
```shell
fuel-createmirror
```
#### 3.3 安装OpenStack
##### 用fuel-master部署控制节点和计算节点
1.启动fuel-controller虚拟机和fuel-compute虚拟机
设置启动顺序首选从网络启动，以便可以通过PXE网络启动bootstrap引导系统。
如图，选择ubuntu_bootstrap。
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1oxep9lj30ka0d53zf.jpg)
2.登录[fuel dashboard](https://10.20.0.2:8443/#login "fuel dashboard")
登录地址、用户名、密码等可以从fuel-master的控制台获取。一般是默认的，如果你在安装过程中做了修改，就按照你修改的来。
3.开始OpenStack配置，新建OpenStack环境
名称和版本： 名称自定义，OpenStack版本选择Mitaka on Ubuntu 14.04，#Fuel9.0版本去除了CentOS，只保留了Ubuntu的部署方式，但是比之前多了个Ubuntu+UCA(UCA use Ubuntucloud archive as a source of packages for Openstack components)。
计算：由于我们是在VirtualBox虚拟机上运行OpenStack，所以这里选择QEMU-KVM。
网络设置：选用Neutron VLAN模式，因为我们只是测试环境，并需要支持百万个用户网络。
后端存储：选择默认的LVM，其实也可以选择Ceph，但是Ceph需要新建一个Ceph节点，在测试环境下并不需要。
附加服务：在测试环境就不需要添加这些。在生产环境按照需求添加这些附加服务。部署完成后还可以增加。
完成：点击新建，即新建了一个OpenStack环境。
按照提示，我们需要在环境中添加至少一个节点才能进行部署。
4.配置节点
节点：
Controller节点：勾选Controller和Cinder
Compute节点：勾选Compiute和Cinder
点击配置接口，逐一配置节点接口，如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1pcys0tj30ip0gnaac.jpg)
网络：
设置-其他，配置NTP Server，改为10.20.0.2，即fuel-master的地址。
网络验证-连通性检查，验证网络。如有错误信息，按照提示进行修改。
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1psoe99j30dw07jjrs.jpg)
5.部署OpenStack
节点配置完成后，返回控制台，点击Deploy Changes进行部署。
可以在fuel-master节点控制台执行fuel-node命令查看节点信息变化。
第一个步骤是利用Cobbler Server安装Ubuntu系统.
安装过程中可以看到节点的Status变为了provisioning, 含义为正在部署底层系统.并且角色也从pending-roles转移到roles。
部署完ubuntu14.04以后，Fuel会继续部署Openstack, 这里是使用Puppet Master 利用SSH协议的SCP命令将Openstack组件部署到node的.然后，转台已经更新为deploying.
等待一段时间，节点已经成功部署完成。
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1q6n201j30dw06wgly.jpg)
点击Horizon 进行登录, 默认用户名密码都是admin.
登录成功后，进入dashboard界面，如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o1qgbsl0j30dw04lgma.jpg)
大功告成！

