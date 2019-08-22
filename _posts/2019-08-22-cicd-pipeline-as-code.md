---
layout: post
title: "CI/CD Pipeline as Code"
date: 2019-08-22 15:06:15
image: '/assets/img/'
description:  '自动构建 CI/CD 环境 Pipeline as Code'
main-class:  'tools'
color: 
tags: 
 - tools
 - script
categories:
 - tools
twitter_text:  "auto deploy CI/CD env"
introduction: "auto deploy CI/CD env"
---


# 前言


根据需求，这里通过组合 **[VirtualBox][virtualbox]，[Vagrant][vagrant]，[gogs][gogs]** 和 **[Minikube][minikube]** 构建出一个一键部署的 CI/CD 本地测试环境


> **Tip:** 当前使用的版本为 **minikube v1.2.0** 、 **kubectl v1.15.0** 、**virtualbox 5.2.24** 和 **vagrant 2.2.2**

---

## 设计思路

* 开发，测试，生产三个环境
* 极简的布署过程(尽量做到开袋即食)
* 充分使用容器的灵活与快捷
* 尽量让常用操作自动化完成

## 当前版本解决了上一个版本存在的以下几点不足

* 开发测试生产环境的流水线打通
* 实现了不同环境在流水线构建过程中将文件整体做为实参传递的机制
* 通过 DockerHub 的国内镜像网站加速镜像的下载过程
* 添加了代码的版本管理功能，并且提供了其 web 管理后台
* 添加了提交代码后的主动触发构建机制
* 实现了 Pipeline as Code
* 加入了构建过程中的人为确认环节


# 部署方法


## OS环境

~~~
nothing@pc:~$ hostnamectl 
   Static hostname: pc
         Icon name: computer-laptop
           Chassis: laptop
        Machine ID: da43bae367934cabac43b08864bb91f9
           Boot ID: 72636b14dc414045989163692df5e6e8
  Operating System: Ubuntu 18.10
            Kernel: Linux 4.18.0-25-generic
      Architecture: x86-64
nothing@pc:~$ 
~~~

**Ubuntu 18.10** 下测试没问题

>**Note:** 其它Linux 版本没有直接测试, 但是理论上能正常运行 **virtualbox 5.2.24** 和 **vagrant 2.2.2** 的 Linux 环境也没问题

## 基础软件依赖

需要系统中已经安装如下两个软件

* **virtualbox 5.2.24**
* **vagrant 2.2.2**

~~~
nothing@pc:~$ virtualbox --help | head 
Oracle VM VirtualBox Manager 5.2.24
(C) 2005-2019 Oracle Corporation
All rights reserved.

Usage:
  --startvm <vmname|UUID>    start a VM by specifying its UUID or name
  --separate                 start a separate VM process
  --normal                   keep normal (windowed) mode during startup
  --fullscreen               switch to fullscreen mode during startup
  --seamless                 switch to seamless mode during startup
nothing@pc:~$ vagrant version
Installed Version: 2.2.2
Latest Version: 2.2.5
 
To upgrade to the latest version, visit the downloads page and
download and install the latest version of Vagrant from the URL
below:

  https://www.vagrantup.com/downloads.html

If you're curious what changed in the latest release, view the
CHANGELOG below:

  https://github.com/hashicorp/vagrant/blob/v2.2.5/CHANGELOG.md
nothing@pc:~$ 
~~~


## 安装方法

~~~
git clone https://github.com/wilmosfang/cicd.git
cd cicd
vagrant up
~~~


## 目录结构


~~~
nothing@pc:~/vagrant/cicd$ tree -L 3 . 
.
├── init.bash
├── package
│   ├── containerd.io-1.2.6-3.3.el7.x86_64.rpm
│   ├── docker-ce-19.03.1-3.el7.x86_64.rpm
│   ├── docker-ce-cli-19.03.1-3.el7.x86_64.rpm
│   ├── docker-ce.repo
│   ├── gogs.zip
│   ├── jenkins-2.176.2-1.1.noarch.rpm
│   ├── jenkins.repo
│   ├── jenkins.zip
│   ├── kubectl
│   └── minikube
├── test_file
│   ├── assets
│   │   ├── assets
│   │   ├── Dockerfile
│   │   ├── Jenkinsfile
│   │   └── README.md
│   ├── dynamic_web
│   │   ├── Dockerfile
│   │   ├── Jenkinsfile
│   │   ├── README.md
│   │   └── server.js
│   ├── dy.zip
│   └── static.zip
└── Vagrantfile

5 directories, 21 files
nothing@pc:~/vagrant/cicd$
~~~

主要由三部分构成

### Vagrantfile

用于对虚拟机的配置进行定义

### init.bash 

用作虚拟机的初始化

* 基础环境初始化
* 安装基础包 (java jenkins docker kubectl minikube git)
* 配置 jenkins
* 配置 Docker 镜像加速
* 生成单机 k8s 集群
* 拉取基础镜像
* 创建与初始化开发测试生产三个运行环境(并且提供三个环境的访问地址)
* 构建与初始化代码版本管理运行环境

**init.bash** 被 **Vagrantfile** 调用

### package

(由于构建过程中网速实在太慢)

为了加速构建过程，里面下载了离线的安装包

~~~
./package/
├── containerd.io-1.2.6-3.3.el7.x86_64.rpm
├── docker-ce-19.03.1-3.el7.x86_64.rpm
├── docker-ce-cli-19.03.1-3.el7.x86_64.rpm
├── docker-ce.repo
├── gogs.zip
├── jenkins-2.176.2-1.1.noarch.rpm
├── jenkins.repo
├── jenkins.zip
├── kubectl
└── minikube

0 directories, 10 files
~~~

* docker 安装包 (也附上了 docker 的库文件)
* jenkins 安装包 (也附带上了 jenkins 的库文件)
* jenkins 插件
* gogs 配置
* kubectl  命令 
* minikube 命令

**package** 被 **init.bash** 调用

## 其它

**test_file** 在此只是测试 demo，对项目的构建不起作用

# 使用


## 访问 jenkins 

安装完成后，可以直接访问 jenkins 

链接 **`http://10.1.0.165:8080/view/CI_CD/`**

![cicd](/assets/img/cicd/cicd13.png)

## 静态资源的发布

![cicd](/assets/img/cicd/cicd14.png)

![cicd](/assets/img/cicd/cicd03.png)

直接上传 zip 包，和对应的发布环境，点构建

## 静态资源的 pipeline 构建

第一步提示上传文件

![cicd](/assets/img/cicd/cicd15.png)

进入后选择要构建的文件

![cicd](/assets/img/cicd/cicd16.png)

在测试环境构建完成和STAGE环境构建完成后都会提示确认

都需要人工确认

![cicd](/assets/img/cicd/cicd17.png)

![cicd](/assets/img/cicd/cicd18.png)

Build 完成后点击步骤进入是可以跟随链接提示查看构建效果的

![cicd](/assets/img/cicd/cicd19.png)

经历了所有环节，成功构建的效果

![cicd](/assets/img/cicd/cicd20.png)

## 静态资源的 pipeline as code

首先拉取代码，做些修改，再上传

~~~
[vagrant@auto ~]$ git clone  http://10.1.0.165:10080/root/assets.git
正克隆到 'assets'...
remote: Enumerating objects: 29, done.
remote: Counting objects: 100% (29/29), done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 29 (delta 3), reused 0 (delta 0)
Unpacking objects: 100% (29/29), done.
[vagrant@auto ~]$ cd assets/
[vagrant@auto assets]$ ls
assets  Dockerfile  Jenkinsfile  README.md
[vagrant@auto assets]$ echo new_css.css > assets/css/new_css.css
[vagrant@auto assets]$ cat assets/css/new_css.css
new_css.css
[vagrant@auto assets]$ git add . 
[vagrant@auto assets]$ git commit -m "for test"

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'vagrant@auto.(none)')
[vagrant@auto assets]$ git config --global user.email "yyghdfz@163.com"
[vagrant@auto assets]$ git config --global user.name "root"
[vagrant@auto assets]$ git commit -m "for test"
[master 6ffc19f] for test
 1 file changed, 1 insertion(+)
 create mode 100644 assets/css/new_css.css
[vagrant@auto assets]$ git push 
warning: push.default 未设置，它的默认值将会在 Git 2.0 由 'matching'
修改为 'simple'。若要不再显示本信息并在其默认值改变后维持当前使用习惯，
进行如下设置：

  git config --global push.default matching

若要不再显示本信息并从现在开始采用新的使用习惯，设置：

  git config --global push.default simple

参见 'git help config' 并查找 'push.default' 以获取更多信息。
（'simple' 模式由 Git 1.7.11 版本引入。如果您有时要使用老版本的 Git，
为保持兼容，请用 'current' 代替 'simple' 模式）

Counting objects: 8, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 420 bytes | 0 bytes/s, done.
Total 5 (delta 2), reused 0 (delta 0)
Username for 'http://10.1.0.165:10080': root
Password for 'http://root@10.1.0.165:10080': 
To http://10.1.0.165:10080/root/assets.git
   4a38670..6ffc19f  master -> master
[vagrant@auto assets]$ 
~~~


Pipeline 会自动触发构建操作

并且在需要人工确认的地方进行等待

![cicd](/assets/img/cicd/cicd21.png)

![cicd](/assets/img/cicd/cicd22.png)

人工确认之前，可以跟随构建结果链接查看变更后的效果

![cicd](/assets/img/cicd/cicd23.png)

查看变更是否达到预期

![cicd](/assets/img/cicd/cicd25.png)

经历了所有环节，成功构建的效果

![cicd](/assets/img/cicd/cicd24.png)


## 动态资源的发布

直接上源码 zip 包，和对应的发布环境，点构建

![cicd](/assets/img/cicd/cicd26.png)

![cicd](/assets/img/cicd/cicd05.png)

这里需要说明的是，我对 java(和 Prevayler) 不熟悉，就没有用来研究这个了，用了 nodejs 替代动态 web 的效果

## 动态资源的 pipeline as code

首先拉取代码，做些修改，再上传

~~~
[vagrant@auto ~]$ git clone http://10.1.0.165:10080/root/dynamic_web.git 
正克隆到 'dynamic_web'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 12 (delta 4), reused 0 (delta 0)
Unpacking objects: 100% (12/12), done.
[vagrant@auto ~]$ cd dynamic_web/
[vagrant@auto dynamic_web]$ ls
Dockerfile  Jenkinsfile  README.md  server.js
[vagrant@auto dynamic_web]$ cat server.js 
var http = require('http');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.write("abc");
  response.write(" | ");
  response.write(process.env.HOSTNAME);
  response.end(": Hello World!  |v3| \n");
};
var www = http.createServer(handleRequest);
www.listen(8080);
[vagrant@auto dynamic_web]$ vim server.js 
[vagrant@auto dynamic_web]$ cat server.js 
var http = require('http');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.write("123");
  response.write(" | ");
  response.write(process.env.HOSTNAME);
  response.end(": Hello World!  |v4| \n");
};
var www = http.createServer(handleRequest);
www.listen(8080);
[vagrant@auto dynamic_web]$ git add .; git commit -m "for dy test"
[master 88d7419] for dy test
 1 file changed, 2 insertions(+), 2 deletions(-)
[vagrant@auto dynamic_web]$ git push 
warning: push.default 未设置，它的默认值将会在 Git 2.0 由 'matching'
修改为 'simple'。若要不再显示本信息并在其默认值改变后维持当前使用习惯，
进行如下设置：

  git config --global push.default matching

若要不再显示本信息并从现在开始采用新的使用习惯，设置：

  git config --global push.default simple

参见 'git help config' 并查找 'push.default' 以获取更多信息。
（'simple' 模式由 Git 1.7.11 版本引入。如果您有时要使用老版本的 Git，
为保持兼容，请用 'current' 代替 'simple' 模式）

Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 285 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
Username for 'http://10.1.0.165:10080': root
Password for 'http://root@10.1.0.165:10080': 
To http://10.1.0.165:10080/root/dynamic_web.git
   92aa507..88d7419  master -> master
[vagrant@auto dynamic_web]$ 
~~~

Pipeline 会自动触发构建操作

并且在需要人工确认的地方进行等待

![cicd](/assets/img/cicd/cicd27.png)

人工确认之前，可以跟随构建结果的链接查看变更后的效果

![cicd](/assets/img/cicd/cicd28.png)

查看变更是否达到预期

![cicd](/assets/img/cicd/cicd29.png)

经历了所有环节，成功构建的效果

![cicd](/assets/img/cicd/cicd30.png)


## 应用初始化

也就是将应用恢复到初始状态

![cicd](/assets/img/cicd/cicd06.png)

其中第二步需要人工确认

## 查看集群监控

![cicd](/assets/img/cicd/cicd07.png)

![cicd](/assets/img/cicd/cicd08.png)

跟随链接可以看到监控界面(此链接不会发生变化)

![cicd](/assets/img/cicd/cicd09.png)

![cicd](/assets/img/cicd/cicd10.png)

监控里可以看到不同 namespace 中的 pod 资源占用量

主要有 CPU MEM DISK NET 用量信息

更高要求的情况下可以考虑构建 prometheus，同时通过脚本定制一些特殊对象

加入关键指标的报警

## 进行实例扩容

选中要扩容的对象指定扩容到多少实例

![cicd](/assets/img/cicd/cicd11.png)

![cicd](/assets/img/cicd/cicd12.png)

## 源码版本管理

比起前面一个版本，这个版本集成了源码管理的功能

![cicd](/assets/img/cicd/cicd31.png)

通过这个管理控制台，可以简单便捷地实现源码的常见管理

也可以提供给其它项目使用

# 其它

## 高可用

* 通过k8s集群扩容到多宿主机
* 前端考虑加上软硬件 LB
* 存储使用ceph 或其它分布式存储

## 日志

* 对接ELK
* 建议与开发约定输出日志格式全都为 json 

## 审核策略

* 通过 intpu 的 submitter 来控制确认有权限审批的对象 


## 说明

* 修改完代码将静态或动态代码包上传构建后，直接跟随链接就可以看到变化后的结果 (如非重置环境，访问的端口不会改变)
* 由于对 java 不熟悉，这里使用的 nodejs 来完成的动态 web 演示

# 附

## 安装过程

~~~
nothing@pc:~/vagrant/cicd$ time vagrant up 
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'centos/7'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'centos/7' is up to date...
==> default: Setting the name of the VM: cicd_default_1566493542945_67034
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: hostonly
    default: Adapter 3: hostonly
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: 
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default: 
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
[default] No Virtualbox Guest Additions installation found.
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirror.jdcloud.com
 * updates: mirror.jdcloud.com
Package binutils-2.27-34.base.el7.x86_64 already installed and latest version
Package 1:make-3.82-23.el7.x86_64 already installed and latest version
Package bzip2-1.0.6-13.el7.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package gcc.x86_64 0:4.8.5-36.el7_6.2 will be installed
--> Processing Dependency: cpp = 4.8.5-36.el7_6.2 for package: gcc-4.8.5-36.el7_6.2.x86_64
--> Processing Dependency: glibc-devel >= 2.2.90-12 for package: gcc-4.8.5-36.el7_6.2.x86_64
--> Processing Dependency: libmpfr.so.4()(64bit) for package: gcc-4.8.5-36.el7_6.2.x86_64
--> Processing Dependency: libmpc.so.3()(64bit) for package: gcc-4.8.5-36.el7_6.2.x86_64
---> Package kernel-devel.x86_64 0:3.10.0-957.12.2.el7 will be installed
---> Package kernel-devel.x86_64 0:3.10.0-957.27.2.el7 will be installed
---> Package perl.x86_64 4:5.16.3-294.el7_6 will be installed
--> Processing Dependency: perl-libs = 4:5.16.3-294.el7_6 for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Socket) >= 1.3 for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Scalar::Util) >= 1.10 for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl-macros for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl-libs for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(threads::shared) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(threads) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(constant) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Time::Local) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Time::HiRes) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Storable) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Socket) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Scalar::Util) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Pod::Simple::XHTML) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Pod::Simple::Search) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Getopt::Long) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Filter::Util::Call) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(File::Temp) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(File::Spec::Unix) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(File::Spec::Functions) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(File::Spec) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(File::Path) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Exporter) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Cwd) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: perl(Carp) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Processing Dependency: libperl.so()(64bit) for package: 4:perl-5.16.3-294.el7_6.x86_64
--> Running transaction check
---> Package cpp.x86_64 0:4.8.5-36.el7_6.2 will be installed
---> Package glibc-devel.x86_64 0:2.17-260.el7_6.6 will be installed
--> Processing Dependency: glibc-headers = 2.17-260.el7_6.6 for package: glibc-devel-2.17-260.el7_6.6.x86_64
--> Processing Dependency: glibc = 2.17-260.el7_6.6 for package: glibc-devel-2.17-260.el7_6.6.x86_64
--> Processing Dependency: glibc-headers for package: glibc-devel-2.17-260.el7_6.6.x86_64
---> Package libmpc.x86_64 0:1.0.1-3.el7 will be installed
---> Package mpfr.x86_64 0:3.1.1-4.el7 will be installed
---> Package perl-Carp.noarch 0:1.26-244.el7 will be installed
---> Package perl-Exporter.noarch 0:5.68-3.el7 will be installed
---> Package perl-File-Path.noarch 0:2.09-2.el7 will be installed
---> Package perl-File-Temp.noarch 0:0.23.01-3.el7 will be installed
---> Package perl-Filter.x86_64 0:1.49-3.el7 will be installed
---> Package perl-Getopt-Long.noarch 0:2.40-3.el7 will be installed
--> Processing Dependency: perl(Pod::Usage) >= 1.14 for package: perl-Getopt-Long-2.40-3.el7.noarch
--> Processing Dependency: perl(Text::ParseWords) for package: perl-Getopt-Long-2.40-3.el7.noarch
---> Package perl-PathTools.x86_64 0:3.40-5.el7 will be installed
---> Package perl-Pod-Simple.noarch 1:3.28-4.el7 will be installed
--> Processing Dependency: perl(Pod::Escapes) >= 1.04 for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
--> Processing Dependency: perl(Encode) for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
---> Package perl-Scalar-List-Utils.x86_64 0:1.27-248.el7 will be installed
---> Package perl-Socket.x86_64 0:2.010-4.el7 will be installed
---> Package perl-Storable.x86_64 0:2.45-3.el7 will be installed
---> Package perl-Time-HiRes.x86_64 4:1.9725-3.el7 will be installed
---> Package perl-Time-Local.noarch 0:1.2300-2.el7 will be installed
---> Package perl-constant.noarch 0:1.27-2.el7 will be installed
---> Package perl-libs.x86_64 4:5.16.3-294.el7_6 will be installed
---> Package perl-macros.x86_64 4:5.16.3-294.el7_6 will be installed
---> Package perl-threads.x86_64 0:1.87-4.el7 will be installed
---> Package perl-threads-shared.x86_64 0:1.43-6.el7 will be installed
--> Running transaction check
---> Package glibc.x86_64 0:2.17-260.el7_6.5 will be updated
--> Processing Dependency: glibc = 2.17-260.el7_6.5 for package: glibc-common-2.17-260.el7_6.5.x86_64
---> Package glibc.x86_64 0:2.17-260.el7_6.6 will be an update
---> Package glibc-headers.x86_64 0:2.17-260.el7_6.6 will be installed
--> Processing Dependency: kernel-headers >= 2.2.1 for package: glibc-headers-2.17-260.el7_6.6.x86_64
--> Processing Dependency: kernel-headers for package: glibc-headers-2.17-260.el7_6.6.x86_64
---> Package perl-Encode.x86_64 0:2.51-7.el7 will be installed
---> Package perl-Pod-Escapes.noarch 1:1.04-294.el7_6 will be installed
---> Package perl-Pod-Usage.noarch 0:1.63-3.el7 will be installed
--> Processing Dependency: perl(Pod::Text) >= 3.15 for package: perl-Pod-Usage-1.63-3.el7.noarch
--> Processing Dependency: perl-Pod-Perldoc for package: perl-Pod-Usage-1.63-3.el7.noarch
---> Package perl-Text-ParseWords.noarch 0:3.29-4.el7 will be installed
--> Running transaction check
---> Package glibc-common.x86_64 0:2.17-260.el7_6.5 will be updated
---> Package glibc-common.x86_64 0:2.17-260.el7_6.6 will be an update
---> Package kernel-headers.x86_64 0:3.10.0-957.27.2.el7 will be installed
---> Package perl-Pod-Perldoc.noarch 0:3.20-4.el7 will be installed
--> Processing Dependency: perl(parent) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
--> Processing Dependency: perl(HTTP::Tiny) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
---> Package perl-podlators.noarch 0:2.5.1-3.el7 will be installed
--> Running transaction check
---> Package perl-HTTP-Tiny.noarch 0:0.033-3.el7 will be installed
---> Package perl-parent.noarch 1:0.225-244.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                   Arch      Version                   Repository  Size
================================================================================
Installing:
 gcc                       x86_64    4.8.5-36.el7_6.2          updates     16 M
 kernel-devel              x86_64    3.10.0-957.12.2.el7       updates     17 M
 kernel-devel              x86_64    3.10.0-957.27.2.el7       updates     17 M
 perl                      x86_64    4:5.16.3-294.el7_6        updates    8.0 M
Installing for dependencies:
 cpp                       x86_64    4.8.5-36.el7_6.2          updates    5.9 M
 glibc-devel               x86_64    2.17-260.el7_6.6          updates    1.1 M
 glibc-headers             x86_64    2.17-260.el7_6.6          updates    684 k
 kernel-headers            x86_64    3.10.0-957.27.2.el7       updates    8.0 M
 libmpc                    x86_64    1.0.1-3.el7               base        51 k
 mpfr                      x86_64    3.1.1-4.el7               base       203 k
 perl-Carp                 noarch    1.26-244.el7              base        19 k
 perl-Encode               x86_64    2.51-7.el7                base       1.5 M
 perl-Exporter             noarch    5.68-3.el7                base        28 k
 perl-File-Path            noarch    2.09-2.el7                base        26 k
 perl-File-Temp            noarch    0.23.01-3.el7             base        56 k
 perl-Filter               x86_64    1.49-3.el7                base        76 k
 perl-Getopt-Long          noarch    2.40-3.el7                base        56 k
 perl-HTTP-Tiny            noarch    0.033-3.el7               base        38 k
 perl-PathTools            x86_64    3.40-5.el7                base        82 k
 perl-Pod-Escapes          noarch    1:1.04-294.el7_6          updates     51 k
 perl-Pod-Perldoc          noarch    3.20-4.el7                base        87 k
 perl-Pod-Simple           noarch    1:3.28-4.el7              base       216 k
 perl-Pod-Usage            noarch    1.63-3.el7                base        27 k
 perl-Scalar-List-Utils    x86_64    1.27-248.el7              base        36 k
 perl-Socket               x86_64    2.010-4.el7               base        49 k
 perl-Storable             x86_64    2.45-3.el7                base        77 k
 perl-Text-ParseWords      noarch    3.29-4.el7                base        14 k
 perl-Time-HiRes           x86_64    4:1.9725-3.el7            base        45 k
 perl-Time-Local           noarch    1.2300-2.el7              base        24 k
 perl-constant             noarch    1.27-2.el7                base        19 k
 perl-libs                 x86_64    4:5.16.3-294.el7_6        updates    688 k
 perl-macros               x86_64    4:5.16.3-294.el7_6        updates     44 k
 perl-parent               noarch    1:0.225-244.el7           base        12 k
 perl-podlators            noarch    2.5.1-3.el7               base       112 k
 perl-threads              x86_64    1.87-4.el7                base        49 k
 perl-threads-shared       x86_64    1.43-6.el7                base        39 k
Updating for dependencies:
 glibc                     x86_64    2.17-260.el7_6.6          updates    3.7 M
 glibc-common              x86_64    2.17-260.el7_6.6          updates     12 M

Transaction Summary
================================================================================
Install  4 Packages (+32 Dependent packages)
Upgrade             (  2 Dependent packages)

Total download size: 92 M
Downloading packages:
Delta RPMs reduced 3.7 M of updates to 769 k (79% saved)
Public key for glibc-devel-2.17-260.el7_6.6.x86_64.rpm is not installed
warning: /var/cache/yum/x86_64/7/updates/packages/glibc-devel-2.17-260.el7_6.6.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for libmpc-1.0.1-3.el7.x86_64.rpm is not installed
--------------------------------------------------------------------------------
Total                                              6.9 MB/s |  89 MB  00:12     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-6.1810.2.el7.centos.x86_64 (@anaconda)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : glibc-common-2.17-260.el7_6.6.x86_64                        1/40 
  Updating   : glibc-2.17-260.el7_6.6.x86_64                               2/40 
  Installing : mpfr-3.1.1-4.el7.x86_64                                     3/40 
  Installing : libmpc-1.0.1-3.el7.x86_64                                   4/40 
  Installing : cpp-4.8.5-36.el7_6.2.x86_64                                 5/40 
  Installing : 1:perl-parent-0.225-244.el7.noarch                          6/40 
  Installing : perl-HTTP-Tiny-0.033-3.el7.noarch                           7/40 
  Installing : perl-podlators-2.5.1-3.el7.noarch                           8/40 
  Installing : perl-Pod-Perldoc-3.20-4.el7.noarch                          9/40 
  Installing : 1:perl-Pod-Escapes-1.04-294.el7_6.noarch                   10/40 
  Installing : perl-Encode-2.51-7.el7.x86_64                              11/40 
  Installing : perl-Text-ParseWords-3.29-4.el7.noarch                     12/40 
  Installing : perl-Pod-Usage-1.63-3.el7.noarch                           13/40 
  Installing : 4:perl-libs-5.16.3-294.el7_6.x86_64                        14/40 
  Installing : 4:perl-macros-5.16.3-294.el7_6.x86_64                      15/40 
  Installing : perl-threads-1.87-4.el7.x86_64                             16/40 
  Installing : perl-Storable-2.45-3.el7.x86_64                            17/40 
  Installing : perl-Exporter-5.68-3.el7.noarch                            18/40 
  Installing : perl-constant-1.27-2.el7.noarch                            19/40 
  Installing : perl-Time-Local-1.2300-2.el7.noarch                        20/40 
  Installing : perl-Socket-2.010-4.el7.x86_64                             21/40 
  Installing : perl-Carp-1.26-244.el7.noarch                              22/40 
  Installing : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      23/40 
  Installing : perl-threads-shared-1.43-6.el7.x86_64                      24/40 
  Installing : perl-PathTools-3.40-5.el7.x86_64                           25/40 
  Installing : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 26/40 
  Installing : 1:perl-Pod-Simple-3.28-4.el7.noarch                        27/40 
  Installing : perl-File-Temp-0.23.01-3.el7.noarch                        28/40 
  Installing : perl-File-Path-2.09-2.el7.noarch                           29/40 
  Installing : perl-Filter-1.49-3.el7.x86_64                              30/40 
  Installing : perl-Getopt-Long-2.40-3.el7.noarch                         31/40 
  Installing : 4:perl-5.16.3-294.el7_6.x86_64                             32/40 
  Installing : kernel-headers-3.10.0-957.27.2.el7.x86_64                  33/40 
  Installing : glibc-headers-2.17-260.el7_6.6.x86_64                      34/40 
  Installing : glibc-devel-2.17-260.el7_6.6.x86_64                        35/40 
  Installing : gcc-4.8.5-36.el7_6.2.x86_64                                36/40 
  Installing : kernel-devel-3.10.0-957.27.2.el7.x86_64                    37/40 
  Installing : kernel-devel-3.10.0-957.12.2.el7.x86_64                    38/40 
  Cleanup    : glibc-common-2.17-260.el7_6.5.x86_64                       39/40 
  Cleanup    : glibc-2.17-260.el7_6.5.x86_64                              40/40 
  Verifying  : perl-HTTP-Tiny-0.033-3.el7.noarch                           1/40 
  Verifying  : perl-threads-shared-1.43-6.el7.x86_64                       2/40 
  Verifying  : perl-Storable-2.45-3.el7.x86_64                             3/40 
  Verifying  : 1:perl-Pod-Escapes-1.04-294.el7_6.noarch                    4/40 
  Verifying  : perl-threads-1.87-4.el7.x86_64                              5/40 
  Verifying  : perl-Exporter-5.68-3.el7.noarch                             6/40 
  Verifying  : perl-constant-1.27-2.el7.noarch                             7/40 
  Verifying  : perl-PathTools-3.40-5.el7.x86_64                            8/40 
  Verifying  : glibc-2.17-260.el7_6.6.x86_64                               9/40 
  Verifying  : kernel-headers-3.10.0-957.27.2.el7.x86_64                  10/40 
  Verifying  : gcc-4.8.5-36.el7_6.2.x86_64                                11/40 
  Verifying  : kernel-devel-3.10.0-957.27.2.el7.x86_64                    12/40 
  Verifying  : 1:perl-parent-0.225-244.el7.noarch                         13/40 
  Verifying  : 4:perl-libs-5.16.3-294.el7_6.x86_64                        14/40 
  Verifying  : perl-File-Temp-0.23.01-3.el7.noarch                        15/40 
  Verifying  : 1:perl-Pod-Simple-3.28-4.el7.noarch                        16/40 
  Verifying  : perl-Time-Local-1.2300-2.el7.noarch                        17/40 
  Verifying  : glibc-headers-2.17-260.el7_6.6.x86_64                      18/40 
  Verifying  : 4:perl-macros-5.16.3-294.el7_6.x86_64                      19/40 
  Verifying  : perl-Socket-2.010-4.el7.x86_64                             20/40 
  Verifying  : perl-Carp-1.26-244.el7.noarch                              21/40 
  Verifying  : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      22/40 
  Verifying  : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 23/40 
  Verifying  : glibc-devel-2.17-260.el7_6.6.x86_64                        24/40 
  Verifying  : libmpc-1.0.1-3.el7.x86_64                                  25/40 
  Verifying  : perl-Pod-Usage-1.63-3.el7.noarch                           26/40 
  Verifying  : kernel-devel-3.10.0-957.12.2.el7.x86_64                    27/40 
  Verifying  : perl-Encode-2.51-7.el7.x86_64                              28/40 
  Verifying  : perl-Pod-Perldoc-3.20-4.el7.noarch                         29/40 
  Verifying  : perl-podlators-2.5.1-3.el7.noarch                          30/40 
  Verifying  : perl-File-Path-2.09-2.el7.noarch                           31/40 
  Verifying  : mpfr-3.1.1-4.el7.x86_64                                    32/40 
  Verifying  : perl-Filter-1.49-3.el7.x86_64                              33/40 
  Verifying  : perl-Getopt-Long-2.40-3.el7.noarch                         34/40 
  Verifying  : perl-Text-ParseWords-3.29-4.el7.noarch                     35/40 
  Verifying  : 4:perl-5.16.3-294.el7_6.x86_64                             36/40 
  Verifying  : cpp-4.8.5-36.el7_6.2.x86_64                                37/40 
  Verifying  : glibc-common-2.17-260.el7_6.6.x86_64                       38/40 
  Verifying  : glibc-common-2.17-260.el7_6.5.x86_64                       39/40 
  Verifying  : glibc-2.17-260.el7_6.5.x86_64                              40/40 

Installed:
  gcc.x86_64 0:4.8.5-36.el7_6.2                                                 
  kernel-devel.x86_64 0:3.10.0-957.12.2.el7                                     
  kernel-devel.x86_64 0:3.10.0-957.27.2.el7                                     
  perl.x86_64 4:5.16.3-294.el7_6                                                

Dependency Installed:
  cpp.x86_64 0:4.8.5-36.el7_6.2                                                 
  glibc-devel.x86_64 0:2.17-260.el7_6.6                                         
  glibc-headers.x86_64 0:2.17-260.el7_6.6                                       
  kernel-headers.x86_64 0:3.10.0-957.27.2.el7                                   
  libmpc.x86_64 0:1.0.1-3.el7                                                   
  mpfr.x86_64 0:3.1.1-4.el7                                                     
  perl-Carp.noarch 0:1.26-244.el7                                               
  perl-Encode.x86_64 0:2.51-7.el7                                               
  perl-Exporter.noarch 0:5.68-3.el7                                             
  perl-File-Path.noarch 0:2.09-2.el7                                            
  perl-File-Temp.noarch 0:0.23.01-3.el7                                         
  perl-Filter.x86_64 0:1.49-3.el7                                               
  perl-Getopt-Long.noarch 0:2.40-3.el7                                          
  perl-HTTP-Tiny.noarch 0:0.033-3.el7                                           
  perl-PathTools.x86_64 0:3.40-5.el7                                            
  perl-Pod-Escapes.noarch 1:1.04-294.el7_6                                      
  perl-Pod-Perldoc.noarch 0:3.20-4.el7                                          
  perl-Pod-Simple.noarch 1:3.28-4.el7                                           
  perl-Pod-Usage.noarch 0:1.63-3.el7                                            
  perl-Scalar-List-Utils.x86_64 0:1.27-248.el7                                  
  perl-Socket.x86_64 0:2.010-4.el7                                              
  perl-Storable.x86_64 0:2.45-3.el7                                             
  perl-Text-ParseWords.noarch 0:3.29-4.el7                                      
  perl-Time-HiRes.x86_64 4:1.9725-3.el7                                         
  perl-Time-Local.noarch 0:1.2300-2.el7                                         
  perl-constant.noarch 0:1.27-2.el7                                             
  perl-libs.x86_64 4:5.16.3-294.el7_6                                           
  perl-macros.x86_64 4:5.16.3-294.el7_6                                         
  perl-parent.noarch 1:0.225-244.el7                                            
  perl-podlators.noarch 0:2.5.1-3.el7                                           
  perl-threads.x86_64 0:1.87-4.el7                                              
  perl-threads-shared.x86_64 0:1.43-6.el7                                       

Dependency Updated:
  glibc.x86_64 0:2.17-260.el7_6.6     glibc-common.x86_64 0:2.17-260.el7_6.6    

Complete!
Copy iso file /usr/share/virtualbox/VBoxGuestAdditions.iso into the box /tmp/VBoxGuestAdditions.iso
Mounting Virtualbox Guest Additions ISO to: /mnt
mount: /dev/loop0 is write-protected, mounting read-only
Installing Virtualbox Guest Additions 5.2.24 - guest version is unknown
Verifying archive integrity... All good.
Uncompressing VirtualBox 5.2.24 Guest Additions for Linux........
VirtualBox Guest Additions installer
Copying additional installer modules ...
Installing additional modules ...
VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel modules.  This may take a while.
VirtualBox Guest Additions: To build modules for other installed kernels, run
VirtualBox Guest Additions:   /sbin/rcvboxadd quicksetup <version>
VirtualBox Guest Additions: Building the modules for kernel 3.10.0-957.12.2.el7.x86_64.
VirtualBox Guest Additions: Starting.
Redirecting to /bin/systemctl start vboxadd.service
Redirecting to /bin/systemctl start vboxadd-service.service
Unmounting Virtualbox Guest Additions ISO from: /mnt
==> default: Checking for guest additions in VM...
==> default: Setting hostname...
==> default: Configuring and enabling network interfaces...
==> default: Rsyncing folder: /home/nothing/vagrant/cicd/ => /vagrant
==> default: Running provisioner: file...
==> default: Running provisioner: shell...
    default: Running: /tmp/vagrant-shell20190823-9209-jpqep9.bash
    default: Loaded plugins: fastestmirror
    default: Loading mirror speeds from cached hostfile
    default:  * base: mirrors.tuna.tsinghua.edu.cn
    default:  * extras: mirror.jdcloud.com
    default:  * updates: mirror.jdcloud.com
    default: Resolving Dependencies
    default: --> Running transaction check
    default: ---> Package java-1.8.0-openjdk.x86_64 1:1.8.0.222.b10-0.el7_6 will be installed
    default: --> Processing Dependency: java-1.8.0-openjdk-headless(x86-64) = 1:1.8.0.222.b10-0.el7_6 for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: xorg-x11-fonts-Type1 for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libjvm.so(SUNWprivate_1.1)(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libjpeg.so.62(LIBJPEG_6.2)(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libjava.so(SUNWprivate_1.1)(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libasound.so.2(ALSA_0.9.0rc4)(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libasound.so.2(ALSA_0.9)(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libXcomposite(x86-64) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: gtk2(x86-64) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: fontconfig(x86-64) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libjvm.so()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libjpeg.so.62()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libjava.so()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libgif.so.4()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libasound.so.2()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libXtst.so.6()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libXrender.so.1()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libXi.so.6()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libXext.so.6()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: libX11.so.6()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Running transaction check
    default: ---> Package alsa-lib.x86_64 0:1.1.6-2.el7 will be installed
    default: ---> Package fontconfig.x86_64 0:2.13.0-4.3.el7 will be installed
    default: --> Processing Dependency: fontpackages-filesystem for package: fontconfig-2.13.0-4.3.el7.x86_64
    default: --> Processing Dependency: dejavu-sans-fonts for package: fontconfig-2.13.0-4.3.el7.x86_64
    default: ---> Package giflib.x86_64 0:4.1.6-9.el7 will be installed
    default: --> Processing Dependency: libSM.so.6()(64bit) for package: giflib-4.1.6-9.el7.x86_64
    default: --> Processing Dependency: libICE.so.6()(64bit) for package: giflib-4.1.6-9.el7.x86_64
    default: ---> Package gtk2.x86_64 0:2.24.31-1.el7 will be installed
    default: --> Processing Dependency: pango >= 1.20.0-1 for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libtiff >= 3.6.1 for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libXrandr >= 1.2.99.4-2 for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: atk >= 1.29.4-2 for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: hicolor-icon-theme for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: gtk-update-icon-cache for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libpangoft2-1.0.so.0()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libpangocairo-1.0.so.0()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libpango-1.0.so.0()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libgdk_pixbuf-2.0.so.0()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libcairo.so.2()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libatk-1.0.so.0()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libXrandr.so.2()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libXinerama.so.1()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libXfixes.so.3()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libXdamage.so.1()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: --> Processing Dependency: libXcursor.so.1()(64bit) for package: gtk2-2.24.31-1.el7.x86_64
    default: ---> Package java-1.8.0-openjdk-headless.x86_64 1:1.8.0.222.b10-0.el7_6 will be installed
    default: --> Processing Dependency: tzdata-java >= 2015d for package: 1:java-1.8.0-openjdk-headless-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: copy-jdk-configs >= 3.3 for package: 1:java-1.8.0-openjdk-headless-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: pcsc-lite-libs(x86-64) for package: 1:java-1.8.0-openjdk-headless-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: lksctp-tools(x86-64) for package: 1:java-1.8.0-openjdk-headless-1.8.0.222.b10-0.el7_6.x86_64
    default: --> Processing Dependency: jpackage-utils for package: 1:java-1.8.0-openjdk-headless-1.8.0.222.b10-0.el7_6.x86_64
    default: ---> Package libX11.x86_64 0:1.6.5-2.el7 will be installed
    default: --> Processing Dependency: libX11-common >= 1.6.5-2.el7 for package: libX11-1.6.5-2.el7.x86_64
    default: --> Processing Dependency: libxcb.so.1()(64bit) for package: libX11-1.6.5-2.el7.x86_64
    default: ---> Package libXcomposite.x86_64 0:0.4.4-4.1.el7 will be installed
    default: ---> Package libXext.x86_64 0:1.3.3-3.el7 will be installed
    default: ---> Package libXi.x86_64 0:1.7.9-1.el7 will be installed
    default: ---> Package libXrender.x86_64 0:0.9.10-1.el7 will be installed
    default: ---> Package libXtst.x86_64 0:1.2.3-1.el7 will be installed
    default: ---> Package libjpeg-turbo.x86_64 0:1.2.90-6.el7 will be installed
    default: ---> Package xorg-x11-fonts-Type1.noarch 0:7.5-9.el7 will be installed
    default: --> Processing Dependency: ttmkfdir for package: xorg-x11-fonts-Type1-7.5-9.el7.noarch
    default: --> Processing Dependency: ttmkfdir for package: xorg-x11-fonts-Type1-7.5-9.el7.noarch
    default: --> Processing Dependency: mkfontdir for package: xorg-x11-fonts-Type1-7.5-9.el7.noarch
    default: --> Processing Dependency: mkfontdir for package: xorg-x11-fonts-Type1-7.5-9.el7.noarch
    default: --> Running transaction check
    default: ---> Package atk.x86_64 0:2.28.1-1.el7 will be installed
    default: ---> Package cairo.x86_64 0:1.15.12-3.el7 will be installed
    default: --> Processing Dependency: libpixman-1.so.0()(64bit) for package: cairo-1.15.12-3.el7.x86_64
    default: --> Processing Dependency: libGL.so.1()(64bit) for package: cairo-1.15.12-3.el7.x86_64
    default: --> Processing Dependency: libEGL.so.1()(64bit) for package: cairo-1.15.12-3.el7.x86_64
    default: ---> Package copy-jdk-configs.noarch 0:3.3-10.el7_5 will be installed
    default: ---> Package dejavu-sans-fonts.noarch 0:2.33-6.el7 will be installed
    default: --> Processing Dependency: dejavu-fonts-common = 2.33-6.el7 for package: dejavu-sans-fonts-2.33-6.el7.noarch
    default: ---> Package fontpackages-filesystem.noarch 0:1.44-8.el7 will be installed
    default: ---> Package gdk-pixbuf2.x86_64 0:2.36.12-3.el7 will be installed
    default: --> Processing Dependency: libjasper.so.1()(64bit) for package: gdk-pixbuf2-2.36.12-3.el7.x86_64
    default: ---> Package gtk-update-icon-cache.x86_64 0:3.22.30-3.el7 will be installed
    default: ---> Package hicolor-icon-theme.noarch 0:0.12-7.el7 will be installed
    default: ---> Package javapackages-tools.noarch 0:3.4.1-11.el7 will be installed
    default: --> Processing Dependency: python-javapackages = 3.4.1-11.el7 for package: javapackages-tools-3.4.1-11.el7.noarch
    default: ---> Package libICE.x86_64 0:1.0.9-9.el7 will be installed
    default: ---> Package libSM.x86_64 0:1.2.2-2.el7 will be installed
    default: ---> Package libX11-common.noarch 0:1.6.5-2.el7 will be installed
    default: ---> Package libXcursor.x86_64 0:1.1.15-1.el7 will be installed
    default: ---> Package libXdamage.x86_64 0:1.1.4-4.1.el7 will be installed
    default: ---> Package libXfixes.x86_64 0:5.0.3-1.el7 will be installed
    default: ---> Package libXinerama.x86_64 0:1.1.3-2.1.el7 will be installed
    default: ---> Package libXrandr.x86_64 0:1.5.1-2.el7 will be installed
    default: ---> Package libtiff.x86_64 0:4.0.3-27.el7_3 will be installed
    default: --> Processing Dependency: libjbig.so.2.0()(64bit) for package: libtiff-4.0.3-27.el7_3.x86_64
    default: ---> Package libxcb.x86_64 0:1.13-1.el7 will be installed
    default: --> Processing Dependency: libXau.so.6()(64bit) for package: libxcb-1.13-1.el7.x86_64
    default: ---> Package lksctp-tools.x86_64 0:1.0.17-2.el7 will be installed
    default: ---> Package pango.x86_64 0:1.42.4-2.el7_6 will be installed
    default: --> Processing Dependency: libthai(x86-64) >= 0.1.9 for package: pango-1.42.4-2.el7_6.x86_64
    default: --> Processing Dependency: libXft(x86-64) >= 2.0.0 for package: pango-1.42.4-2.el7_6.x86_64
    default: --> Processing Dependency: harfbuzz(x86-64) >= 1.4.2 for package: pango-1.42.4-2.el7_6.x86_64
    default: --> Processing Dependency: fribidi(x86-64) >= 1.0 for package: pango-1.42.4-2.el7_6.x86_64
    default: --> Processing Dependency: libthai.so.0(LIBTHAI_0.1)(64bit) for package: pango-1.42.4-2.el7_6.x86_64
    default: --> Processing Dependency: libthai.so.0()(64bit) for package: pango-1.42.4-2.el7_6.x86_64
    default: --> Processing Dependency: libharfbuzz.so.0()(64bit) for package: pango-1.42.4-2.el7_6.x86_64
    default: --> Processing Dependency: libfribidi.so.0()(64bit) for package: pango-1.42.4-2.el7_6.x86_64
    default: --> Processing Dependency: libXft.so.2()(64bit) for package: pango-1.42.4-2.el7_6.x86_64
    default: ---> Package pcsc-lite-libs.x86_64 0:1.8.8-8.el7 will be installed
    default: ---> Package ttmkfdir.x86_64 0:3.0.9-42.el7 will be installed
    default: ---> Package tzdata-java.noarch 0:2019b-1.el7 will be installed
    default: ---> Package xorg-x11-font-utils.x86_64 1:7.5-21.el7 will be installed
    default: --> Processing Dependency: libfontenc.so.1()(64bit) for package: 1:xorg-x11-font-utils-7.5-21.el7.x86_64
    default: --> Running transaction check
    default: ---> Package dejavu-fonts-common.noarch 0:2.33-6.el7 will be installed
    default: ---> Package fribidi.x86_64 0:1.0.2-1.el7 will be installed
    default: ---> Package harfbuzz.x86_64 0:1.7.5-2.el7 will be installed
    default: --> Processing Dependency: libgraphite2.so.3()(64bit) for package: harfbuzz-1.7.5-2.el7.x86_64
    default: ---> Package jasper-libs.x86_64 0:1.900.1-33.el7 will be installed
    default: ---> Package jbigkit-libs.x86_64 0:2.0-11.el7 will be installed
    default: ---> Package libXau.x86_64 0:1.0.8-2.1.el7 will be installed
    default: ---> Package libXft.x86_64 0:2.3.2-2.el7 will be installed
    default: ---> Package libfontenc.x86_64 0:1.1.3-3.el7 will be installed
    default: ---> Package libglvnd-egl.x86_64 1:1.0.1-0.8.git5baa1e5.el7 will be installed
    default: --> Processing Dependency: libglvnd(x86-64) = 1:1.0.1-0.8.git5baa1e5.el7 for package: 1:libglvnd-egl-1.0.1-0.8.git5baa1e5.el7.x86_64
    default: --> Processing Dependency: mesa-libEGL(x86-64) >= 13.0.4-1 for package: 1:libglvnd-egl-1.0.1-0.8.git5baa1e5.el7.x86_64
    default: --> Processing Dependency: libGLdispatch.so.0()(64bit) for package: 1:libglvnd-egl-1.0.1-0.8.git5baa1e5.el7.x86_64
    default: ---> Package libglvnd-glx.x86_64 1:1.0.1-0.8.git5baa1e5.el7 will be installed
    default: --> Processing Dependency: mesa-libGL(x86-64) >= 13.0.4-1 for package: 1:libglvnd-glx-1.0.1-0.8.git5baa1e5.el7.x86_64
    default: ---> Package libthai.x86_64 0:0.1.14-9.el7 will be installed
    default: ---> Package pixman.x86_64 0:0.34.0-1.el7 will be installed
    default: ---> Package python-javapackages.noarch 0:3.4.1-11.el7 will be installed
    default: --> Processing Dependency: python-lxml for package: python-javapackages-3.4.1-11.el7.noarch
    default: --> Running transaction check
    default: ---> Package graphite2.x86_64 0:1.3.10-1.el7_3 will be installed
    default: ---> Package libglvnd.x86_64 1:1.0.1-0.8.git5baa1e5.el7 will be installed
    default: ---> Package mesa-libEGL.x86_64 0:18.0.5-4.el7_6 will be installed
    default: --> Processing Dependency: mesa-libgbm = 18.0.5-4.el7_6 for package: mesa-libEGL-18.0.5-4.el7_6.x86_64
    default: --> Processing Dependency: libxshmfence.so.1()(64bit) for package: mesa-libEGL-18.0.5-4.el7_6.x86_64
    default: --> Processing Dependency: libwayland-server.so.0()(64bit) for package: mesa-libEGL-18.0.5-4.el7_6.x86_64
    default: --> Processing Dependency: libwayland-client.so.0()(64bit) for package: mesa-libEGL-18.0.5-4.el7_6.x86_64
    default: --> Processing Dependency: libglapi.so.0()(64bit) for package: mesa-libEGL-18.0.5-4.el7_6.x86_64
    default: --> Processing Dependency: libgbm.so.1()(64bit) for package: mesa-libEGL-18.0.5-4.el7_6.x86_64
    default: ---> Package mesa-libGL.x86_64 0:18.0.5-4.el7_6 will be installed
    default: --> Processing Dependency: libXxf86vm.so.1()(64bit) for package: mesa-libGL-18.0.5-4.el7_6.x86_64
    default: ---> Package python-lxml.x86_64 0:3.2.1-4.el7 will be installed
    default: --> Running transaction check
    default: ---> Package libXxf86vm.x86_64 0:1.1.4-1.el7 will be installed
    default: ---> Package libwayland-client.x86_64 0:1.15.0-1.el7 will be installed
    default: ---> Package libwayland-server.x86_64 0:1.15.0-1.el7 will be installed
    default: ---> Package libxshmfence.x86_64 0:1.2-1.el7 will be installed
    default: ---> Package mesa-libgbm.x86_64 0:18.0.5-4.el7_6 will be installed
    default: ---> Package mesa-libglapi.x86_64 0:18.0.5-4.el7_6 will be installed
    default: --> Finished Dependency Resolution
    default: 
    default: Dependencies Resolved
    default: 
    default: ================================================================================
    default:  Package                     Arch   Version                       Repository
    default:                                                                            Size
    default: ================================================================================
    default: Installing:
    default:  java-1.8.0-openjdk          x86_64 1:1.8.0.222.b10-0.el7_6       updates 274 k
    default: Installing for dependencies:
    default:  alsa-lib                    x86_64 1.1.6-2.el7                   base    424 k
    default:  atk                         x86_64 2.28.1-1.el7                  base    263 k
    default:  cairo                       x86_64 1.15.12-3.el7                 base    741 k
    default:  copy-jdk-configs            noarch 3.3-10.el7_5                  base     21 k
    default:  dejavu-fonts-common         noarch 2.33-6.el7                    base     64 k
    default:  dejavu-sans-fonts           noarch 2.33-6.el7                    base    1.4 M
    default:  fontconfig                  x86_64 2.13.0-4.3.el7                base    254 k
    default:  fontpackages-filesystem     noarch 1.44-8.el7                    base    9.9 k
    default:  fribidi                     x86_64 1.0.2-1.el7                   base     79 k
    default:  gdk-pixbuf2                 x86_64 2.36.12-3.el7                 base    570 k
    default:  giflib                      x86_64 4.1.6-9.el7                   base     40 k
    default:  graphite2                   x86_64 1.3.10-1.el7_3                base    115 k
    default:  gtk-update-icon-cache       x86_64 3.22.30-3.el7                 base     28 k
    default:  gtk2                        x86_64 2.24.31-1.el7                 base    3.4 M
    default:  harfbuzz                    x86_64 1.7.5-2.el7                   base    267 k
    default:  hicolor-icon-theme          noarch 0.12-7.el7                    base     42 k
    default:  jasper-libs                 x86_64 1.900.1-33.el7                base    150 k
    default:  java-1.8.0-openjdk-headless x86_64 1:1.8.0.222.b10-0.el7_6       updates  32 M
    default:  javapackages-tools          noarch 3.4.1-11.el7                  base     73 k
    default:  jbigkit-libs                x86_64 2.0-11.el7                    base     46 k
    default:  libICE                      x86_64 1.0.9-9.el7                   base     66 k
    default:  libSM                       x86_64 1.2.2-2.el7                   base     39 k
    default:  libX11                      x86_64 1.6.5-2.el7                   base    606 k
    default:  libX11-common               noarch 1.6.5-2.el7                   base    164 k
    default:  libXau                      x86_64 1.0.8-2.1.el7                 base     29 k
    default:  libXcomposite               x86_64 0.4.4-4.1.el7                 base     22 k
    default:  libXcursor                  x86_64 1.1.15-1.el7                  base     30 k
    default:  libXdamage                  x86_64 1.1.4-4.1.el7                 base     20 k
    default:  libXext                     x86_64 1.3.3-3.el7                   base     39 k
    default:  libXfixes                   x86_64 5.0.3-1.el7                   base     18 k
    default:  libXft                      x86_64 2.3.2-2.el7                   base     58 k
    default:  libXi                       x86_64 1.7.9-1.el7                   base     40 k
    default:  libXinerama                 x86_64 1.1.3-2.1.el7                 base     14 k
    default:  libXrandr                   x86_64 1.5.1-2.el7                   base     27 k
    default:  libXrender                  x86_64 0.9.10-1.el7                  base     26 k
    default:  libXtst                     x86_64 1.2.3-1.el7                   base     20 k
    default:  libXxf86vm                  x86_64 1.1.4-1.el7                   base     18 k
    default:  libfontenc                  x86_64 1.1.3-3.el7                   base     31 k
    default:  libglvnd                    x86_64 1:1.0.1-0.8.git5baa1e5.el7    base     89 k
    default:  libglvnd-egl                x86_64 1:1.0.1-0.8.git5baa1e5.el7    base     44 k
    default:  libglvnd-glx                x86_64 1:1.0.1-0.8.git5baa1e5.el7    base    125 k
    default:  libjpeg-turbo               x86_64 1.2.90-6.el7                  base    134 k
    default:  libthai                     x86_64 0.1.14-9.el7                  base    187 k
    default:  libtiff                     x86_64 4.0.3-27.el7_3                base    170 k
    default:  libwayland-client           x86_64 1.15.0-1.el7                  base     33 k
    default:  libwayland-server           x86_64 1.15.0-1.el7                  base     39 k
    default:  libxcb                      x86_64 1.13-1.el7                    base    214 k
    default:  libxshmfence                x86_64 1.2-1.el7                     base    7.2 k
    default:  lksctp-tools                x86_64 1.0.17-2.el7                  base     88 k
    default:  mesa-libEGL                 x86_64 18.0.5-4.el7_6                updates 102 k
    default:  mesa-libGL                  x86_64 18.0.5-4.el7_6                updates 162 k
    default:  mesa-libgbm                 x86_64 18.0.5-4.el7_6                updates  38 k
    default:  mesa-libglapi               x86_64 18.0.5-4.el7_6                updates  44 k
    default:  pango                       x86_64 1.42.4-2.el7_6                updates 280 k
    default:  pcsc-lite-libs              x86_64 1.8.8-8.el7                   base     34 k
    default:  pixman                      x86_64 0.34.0-1.el7                  base    248 k
    default:  python-javapackages         noarch 3.4.1-11.el7                  base     31 k
    default:  python-lxml                 x86_64 3.2.1-4.el7                   base    758 k
    default:  ttmkfdir                    x86_64 3.0.9-42.el7                  base     48 k
    default:  tzdata-java                 noarch 2019b-1.el7                   updates 187 k
    default:  xorg-x11-font-utils         x86_64 1:7.5-21.el7                  base    104 k
    default:  xorg-x11-fonts-Type1        noarch 7.5-9.el7                     base    521 k
    default: 
    default: Transaction Summary
    default: ================================================================================
    default: Install  1 Package (+62 Dependent packages)
    default: Total download size: 45 M
    default: Installed size: 147 M
    default: Downloading packages:
    default: --------------------------------------------------------------------------------
    default: Total                                              1.9 MB/s |  45 MB  00:24     
    default: Running transaction check
    default: Running transaction test
    default: Transaction test succeeded
    default: Running transaction
    default:   Installing : libjpeg-turbo-1.2.90-6.el7.x86_64                           1/63
    default:  
    default:   Installing : mesa-libglapi-18.0.5-4.el7_6.x86_64                         2/63
    default:  
    default:   Installing : libxshmfence-1.2-1.el7.x86_64                               3/63
    default:  
    default:   Installing : 1:libglvnd-1.0.1-0.8.git5baa1e5.el7.x86_64                  4/63
    default:  
    default:   Installing : fontpackages-filesystem-1.44-8.el7.noarch                   5/63
    default:  
    default:   Installing : libICE-1.0.9-9.el7.x86_64                                   6/63
    default:  
    default:   Installing : libwayland-server-1.15.0-1.el7.x86_64                       7/63
    default:  
    default:   Installing : mesa-libgbm-18.0.5-4.el7_6.x86_64                           8/63
    default:  
    default:   Installing : libSM-1.2.2-2.el7.x86_64                                    9/63
    default:  
    default:   Installing : dejavu-fonts-common-2.33-6.el7.noarch                      10/63
    default:  
    default:   Installing : dejavu-sans-fonts-2.33-6.el7.noarch                        11/63
    default:  
    default:   Installing : fontconfig-2.13.0-4.3.el7.x86_64                           12/63
    default:  
    default:   Installing : jasper-libs-1.900.1-33.el7.x86_64                          13/63
    default:  
    default:   Installing : pixman-0.34.0-1.el7.x86_64                                 14/63
    default:  
    default:   Installing : copy-jdk-configs-3.3-10.el7_5.noarch                       15/63
    default:  
    default:   Installing : libfontenc-1.1.3-3.el7.x86_64                              16/63
    default:  
    default:   Installing : 1:xorg-x11-font-utils-7.5-21.el7.x86_64                    17/63
    default:  
    default:   Installing : libthai-0.1.14-9.el7.x86_64                                18/63
    default:  
    default:   Installing : python-lxml-3.2.1-4.el7.x86_64                             19/63
    default:  
    default:   Installing : python-javapackages-3.4.1-11.el7.noarch                    20/63
    default:  
    default:   Installing : javapackages-tools-3.4.1-11.el7.noarch                     21/63
    default:  
    default:   Installing : libX11-common-1.6.5-2.el7.noarch                           22/63
    default:  
    default:   Installing : graphite2-1.3.10-1.el7_3.x86_64                            23/63
    default:  
    default:   Installing : harfbuzz-1.7.5-2.el7.x86_64                                24/63
    default:  
    default:   Installing : libXau-1.0.8-2.1.el7.x86_64                                25/63
    default:  
    default:   Installing : libxcb-1.13-1.el7.x86_64                                   26/63
    default:  
    default:   Installing : libX11-1.6.5-2.el7.x86_64                                  27/63
    default:  
    default:   Installing : libXext-1.3.3-3.el7.x86_64                                 28/63
    default:  
    default:   Installing : libXrender-0.9.10-1.el7.x86_64                             29/63
    default:  
    default:   Installing : libXfixes-5.0.3-1.el7.x86_64                               30/63
    default:  
    default:   Installing : libXi-1.7.9-1.el7.x86_64                                   31/63
    default:  
    default:   Installing : libXdamage-1.1.4-4.1.el7.x86_64                            32/63
    default:  
    default:   Installing : libXcomposite-0.4.4-4.1.el7.x86_64                         33/63
    default:  
    default:   Installing : libXtst-1.2.3-1.el7.x86_64                                 34/63
    default:  
    default:   Installing : libXcursor-1.1.15-1.el7.x86_64                             35/63
    default:  
    default:   Installing : libXft-2.3.2-2.el7.x86_64                                  36/63
    default:  
    default:   Installing : libXrandr-1.5.1-2.el7.x86_64                               37/63
    default:  
    default:   Installing : libXinerama-1.1.3-2.1.el7.x86_64                           38/63
    default:  
    default:   Installing : libXxf86vm-1.1.4-1.el7.x86_64                              39/63
    default:  
    default:   Installing : mesa-libGL-18.0.5-4.el7_6.x86_64                           40/63
    default:  
    default:   Installing : 1:libglvnd-glx-1.0.1-0.8.git5baa1e5.el7.x86_64             41/63
    default:  
    default:   Installing : giflib-4.1.6-9.el7.x86_64                                  42/63
    default:  
    default:   Installing : jbigkit-libs-2.0-11.el7.x86_64                             43/63
    default:  
    default:   Installing : libtiff-4.0.3-27.el7_3.x86_64                              44/63
    default:  
    default:   Installing : gdk-pixbuf2-2.36.12-3.el7.x86_64                           45/63
    default:  
    default:   Installing : gtk-update-icon-cache-3.22.30-3.el7.x86_64                 46/63
    default:  
    default:   Installing : atk-2.28.1-1.el7.x86_64                                    47/63
    default:  
    default:   Installing : pcsc-lite-libs-1.8.8-8.el7.x86_64                          48/63
    default:  
    default:   Installing : fribidi-1.0.2-1.el7.x86_64                                 49/63
    default:  
    default:   Installing : lksctp-tools-1.0.17-2.el7.x86_64                           50/63
    default:  
    default:   Installing : alsa-lib-1.1.6-2.el7.x86_64                                51/63
    default:  
    default:   Installing : tzdata-java-2019b-1.el7.noarch                             52/63
    default:  
    default:   Installing : 1:java-1.8.0-openjdk-headless-1.8.0.222.b10-0.el7_6.x86_   53/63
    default:  
    default:   Installing : libwayland-client-1.15.0-1.el7.x86_64                      54/63
    default:  
    default:   Installing : 1:libglvnd-egl-1.0.1-0.8.git5baa1e5.el7.x86_64             55/63
    default:  
    default:   Installing : mesa-libEGL-18.0.5-4.el7_6.x86_64                          56/63
    default:  
    default:   Installing : cairo-1.15.12-3.el7.x86_64                                 57/63
    default:  
    default:   Installing : pango-1.42.4-2.el7_6.x86_64                                58/63
    default:  
    default:   Installing : hicolor-icon-theme-0.12-7.el7.noarch                       59/63
    default:  
    default:   Installing : gtk2-2.24.31-1.el7.x86_64                                  60/63
    default:  
    default:   Installing : ttmkfdir-3.0.9-42.el7.x86_64                               61/63
    default:  
    default:   Installing : xorg-x11-fonts-Type1-7.5-9.el7.noarch                      62/63
    default:  
    default:   Installing : 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64          63/63
    default:  
    default:   Verifying  : 1:java-1.8.0-openjdk-headless-1.8.0.222.b10-0.el7_6.x86_    1/63
    default:  
    default:   Verifying  : libXext-1.3.3-3.el7.x86_64                                  2/63
    default:  
    default:   Verifying  : libXi-1.7.9-1.el7.x86_64                                    3/63
    default:  
    default:   Verifying  : libtiff-4.0.3-27.el7_3.x86_64                               4/63
    default:  
    default:   Verifying  : fontconfig-2.13.0-4.3.el7.x86_64                            5/63
    default:  
    default:   Verifying  : giflib-4.1.6-9.el7.x86_64                                   6/63
    default:  
    default:   Verifying  : libXinerama-1.1.3-2.1.el7.x86_64                            7/63
    default:  
    default:   Verifying  : libXrender-0.9.10-1.el7.x86_64                              8/63
    default:  
    default:   Verifying  : javapackages-tools-3.4.1-11.el7.noarch                      9/63
    default:  
    default:   Verifying  : 1:xorg-x11-font-utils-7.5-21.el7.x86_64                    10/63
    default:  
    default:   Verifying  : libXxf86vm-1.1.4-1.el7.x86_64                              11/63
    default:  
    default:   Verifying  : libwayland-server-1.15.0-1.el7.x86_64                      12/63
    default:  
    default:   Verifying  : libXcursor-1.1.15-1.el7.x86_64                             13/63
    default:  
    default:   Verifying  : libICE-1.0.9-9.el7.x86_64                                  14/63
    default:  
    default:   Verifying  : pango-1.42.4-2.el7_6.x86_64                                15/63
    default:  
    default:   Verifying  : fontpackages-filesystem-1.44-8.el7.noarch                  16/63
    default:  
    default:   Verifying  : ttmkfdir-3.0.9-42.el7.x86_64                               17/63
    default:  
    default:   Verifying  : libjpeg-turbo-1.2.90-6.el7.x86_64                          18/63
    default:  
    default:   Verifying  : hicolor-icon-theme-0.12-7.el7.noarch                       19/63
    default:  
    default:   Verifying  : libwayland-client-1.15.0-1.el7.x86_64                      20/63
    default:  
    default:   Verifying  : gdk-pixbuf2-2.36.12-3.el7.x86_64                           21/63
    default:  
    default:   Verifying  : gtk2-2.24.31-1.el7.x86_64                                  22/63
    default:  
    default:   Verifying  : gtk-update-icon-cache-3.22.30-3.el7.x86_64                 23/63
    default:  
    default:   Verifying  : python-javapackages-3.4.1-11.el7.noarch                    24/63
    default:  
    default:   Verifying  : tzdata-java-2019b-1.el7.noarch                             25/63
    default:  
    default:   Verifying  : mesa-libgbm-18.0.5-4.el7_6.x86_64                          26/63
    default:  
    default:   Verifying  : dejavu-fonts-common-2.33-6.el7.noarch                      27/63
    default:  
    default:   Verifying  : alsa-lib-1.1.6-2.el7.x86_64                                28/63
    default:  
    default:   Verifying  : libXcomposite-0.4.4-4.1.el7.x86_64                         29/63
    default:  
    default:   Verifying  : libXtst-1.2.3-1.el7.x86_64                                 30/63
    default:  
    default:   Verifying  : 1:libglvnd-1.0.1-0.8.git5baa1e5.el7.x86_64                 31/63
    default:  
    default:   Verifying  : libxcb-1.13-1.el7.x86_64                                   32/63
    default:  
    default:   Verifying  : libXft-2.3.2-2.el7.x86_64                                  33/63
    default:  
    default:   Verifying  : 1:libglvnd-egl-1.0.1-0.8.git5baa1e5.el7.x86_64             34/63
    default:  
    default:   Verifying  : lksctp-tools-1.0.17-2.el7.x86_64                           35/63
    default:  
    default:   Verifying  : 1:java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64          36/63
    default:  
    default:   Verifying  : mesa-libEGL-18.0.5-4.el7_6.x86_64                          37/63
    default:  
    default:   Verifying  : mesa-libGL-18.0.5-4.el7_6.x86_64                           38/63
    default:  
    default:   Verifying  : xorg-x11-fonts-Type1-7.5-9.el7.noarch                      39/63
    default:  
    default:   Verifying  : harfbuzz-1.7.5-2.el7.x86_64                                40/63
    default:  
    default:   Verifying  : fribidi-1.0.2-1.el7.x86_64                                 41/63
    default:  
    default:   Verifying  : libX11-1.6.5-2.el7.x86_64                                  42/63
    default:  
    default:   Verifying  : 1:libglvnd-glx-1.0.1-0.8.git5baa1e5.el7.x86_64             43/63
    default:  
    default:   Verifying  : dejavu-sans-fonts-2.33-6.el7.noarch                        44/63
    default:  
    default:   Verifying  : libXrandr-1.5.1-2.el7.x86_64                               45/63
    default:  
    default:   Verifying  : pcsc-lite-libs-1.8.8-8.el7.x86_64                          46/63
    default:  
    default:   Verifying  : atk-2.28.1-1.el7.x86_64                                    47/63
    default:  
    default:   Verifying  : jbigkit-libs-2.0-11.el7.x86_64                             48/63
    default:  
    default:   Verifying  : cairo-1.15.12-3.el7.x86_64                                 49/63
    default:  
    default:   Verifying  : libxshmfence-1.2-1.el7.x86_64                              50/63
    default:  
    default:   Verifying  : libXau-1.0.8-2.1.el7.x86_64                                51/63
    default:  
    default:   Verifying  : libSM-1.2.2-2.el7.x86_64                                   52/63
    default:  
    default:   Verifying  : jasper-libs-1.900.1-33.el7.x86_64                          53/63
    default:  
    default:   Verifying  : graphite2-1.3.10-1.el7_3.x86_64                            54/63
    default:  
    default:   Verifying  : libX11-common-1.6.5-2.el7.noarch                           55/63
    default:  
    default:   Verifying  : python-lxml-3.2.1-4.el7.x86_64                             56/63
    default:  
    default:   Verifying  : libthai-0.1.14-9.el7.x86_64                                57/63
    default:  
    default:   Verifying  : libXdamage-1.1.4-4.1.el7.x86_64                            58/63
    default:  
    default:   Verifying  : libXfixes-5.0.3-1.el7.x86_64                               59/63
    default:  
    default:   Verifying  : libfontenc-1.1.3-3.el7.x86_64                              60/63
    default:  
    default:   Verifying  : mesa-libglapi-18.0.5-4.el7_6.x86_64                        61/63
    default:  
    default:   Verifying  : copy-jdk-configs-3.3-10.el7_5.noarch                       62/63
    default:  
    default:   Verifying  : pixman-0.34.0-1.el7.x86_64                                 63/63
    default:  
    default: 
    default: Installed:
    default:   java-1.8.0-openjdk.x86_64 1:1.8.0.222.b10-0.el7_6                             
    default: 
    default: Dependency Installed:
    default:   alsa-lib.x86_64 0:1.1.6-2.el7                                                 
    default:   atk.x86_64 0:2.28.1-1.el7                                                     
    default:   cairo.x86_64 0:1.15.12-3.el7                                                  
    default:   copy-jdk-configs.noarch 0:3.3-10.el7_5                                        
    default:   dejavu-fonts-common.noarch 0:2.33-6.el7                                       
    default:   dejavu-sans-fonts.noarch 0:2.33-6.el7                                         
    default:   fontconfig.x86_64 0:2.13.0-4.3.el7                                            
    default:   fontpackages-filesystem.noarch 0:1.44-8.el7                                   
    default:   fribidi.x86_64 0:1.0.2-1.el7                                                  
    default:   gdk-pixbuf2.x86_64 0:2.36.12-3.el7                                            
    default:   giflib.x86_64 0:4.1.6-9.el7                                                   
    default:   graphite2.x86_64 0:1.3.10-1.el7_3                                             
    default:   gtk-update-icon-cache.x86_64 0:3.22.30-3.el7                                  
    default:   gtk2.x86_64 0:2.24.31-1.el7                                                   
    default:   harfbuzz.x86_64 0:1.7.5-2.el7                                                 
    default:   hicolor-icon-theme.noarch 0:0.12-7.el7                                        
    default:   jasper-libs.x86_64 0:1.900.1-33.el7                                           
    default:   java-1.8.0-openjdk-headless.x86_64 1:1.8.0.222.b10-0.el7_6                    
    default:   javapackages-tools.noarch 0:3.4.1-11.el7                                      
    default:   jbigkit-libs.x86_64 0:2.0-11.el7                                              
    default:   libICE.x86_64 0:1.0.9-9.el7                                                   
    default:   libSM.x86_64 0:1.2.2-2.el7                                                    
    default:   libX11.x86_64 0:1.6.5-2.el7                                                   
    default:   libX11-common.noarch 0:1.6.5-2.el7                                            
    default:   libXau.x86_64 0:1.0.8-2.1.el7                                                 
    default:   libXcomposite.x86_64 0:0.4.4-4.1.el7                                          
    default:   libXcursor.x86_64 0:1.1.15-1.el7                                              
    default:   libXdamage.x86_64 0:1.1.4-4.1.el7                                             
    default:   libXext.x86_64 0:1.3.3-3.el7                                                  
    default:   libXfixes.x86_64 0:5.0.3-1.el7                                                
    default:   libXft.x86_64 0:2.3.2-2.el7                                                   
    default:   libXi.x86_64 0:1.7.9-1.el7                                                    
    default:   libXinerama.x86_64 0:1.1.3-2.1.el7                                            
    default:   libXrandr.x86_64 0:1.5.1-2.el7                                                
    default:   libXrender.x86_64 0:0.9.10-1.el7                                              
    default:   libXtst.x86_64 0:1.2.3-1.el7                                                  
    default:   libXxf86vm.x86_64 0:1.1.4-1.el7                                               
    default:   libfontenc.x86_64 0:1.1.3-3.el7                                               
    default:   libglvnd.x86_64 1:1.0.1-0.8.git5baa1e5.el7                                    
    default:   libglvnd-egl.x86_64 1:1.0.1-0.8.git5baa1e5.el7                                
    default:   libglvnd-glx.x86_64 1:1.0.1-0.8.git5baa1e5.el7                                
    default:   libjpeg-turbo.x86_64 0:1.2.90-6.el7                                           
    default:   libthai.x86_64 0:0.1.14-9.el7                                                 
    default:   libtiff.x86_64 0:4.0.3-27.el7_3                                               
    default:   libwayland-client.x86_64 0:1.15.0-1.el7                                       
    default:   libwayland-server.x86_64 0:1.15.0-1.el7                                       
    default:   libxcb.x86_64 0:1.13-1.el7                                                    
    default:   libxshmfence.x86_64 0:1.2-1.el7                                               
    default:   lksctp-tools.x86_64 0:1.0.17-2.el7                                            
    default:   mesa-libEGL.x86_64 0:18.0.5-4.el7_6                                           
    default:   mesa-libGL.x86_64 0:18.0.5-4.el7_6                                            
    default:   mesa-libgbm.x86_64 0:18.0.5-4.el7_6                                           
    default:   mesa-libglapi.x86_64 0:18.0.5-4.el7_6                                         
    default:   pango.x86_64 0:1.42.4-2.el7_6                                                 
    default:   pcsc-lite-libs.x86_64 0:1.8.8-8.el7                                           
    default:   pixman.x86_64 0:0.34.0-1.el7                                                  
    default:   python-javapackages.noarch 0:3.4.1-11.el7                                     
    default:   python-lxml.x86_64 0:3.2.1-4.el7                                              
    default:   ttmkfdir.x86_64 0:3.0.9-42.el7                                                
    default:   tzdata-java.noarch 0:2019b-1.el7                                              
    default:   xorg-x11-font-utils.x86_64 1:7.5-21.el7                                       
    default:   xorg-x11-fonts-Type1.noarch 0:7.5-9.el7                                       
    default: Complete!
    default: Loaded plugins: fastestmirror
    default: Loading mirror speeds from cached hostfile
    default:  * base: mirrors.tuna.tsinghua.edu.cn
    default:  * extras: mirror.jdcloud.com
    default:  * updates: mirror.jdcloud.com
    default: Resolving Dependencies
    default: --> Running transaction check
    default: ---> Package curl.x86_64 0:7.29.0-51.el7 will be updated
    default: ---> Package curl.x86_64 0:7.29.0-51.el7_6.3 will be an update
    default: --> Processing Dependency: libcurl = 7.29.0-51.el7_6.3 for package: curl-7.29.0-51.el7_6.3.x86_64
    default: ---> Package git.x86_64 0:1.8.3.1-20.el7 will be installed
    default: --> Processing Dependency: perl-Git = 1.8.3.1-20.el7 for package: git-1.8.3.1-20.el7.x86_64
    default: --> Processing Dependency: perl(Term::ReadKey) for package: git-1.8.3.1-20.el7.x86_64
    default: --> Processing Dependency: perl(Git) for package: git-1.8.3.1-20.el7.x86_64
    default: --> Processing Dependency: perl(Error) for package: git-1.8.3.1-20.el7.x86_64
    default: ---> Package net-tools.x86_64 0:2.0-0.24.20131004git.el7 will be installed
    default: ---> Package unzip.x86_64 0:6.0-19.el7 will be installed
    default: ---> Package vim-enhanced.x86_64 2:7.4.160-6.el7_6 will be installed
    default: --> Processing Dependency: vim-common = 2:7.4.160-6.el7_6 for package: 2:vim-enhanced-7.4.160-6.el7_6.x86_64
    default: --> Processing Dependency: libgpm.so.2()(64bit) for package: 2:vim-enhanced-7.4.160-6.el7_6.x86_64
    default: ---> Package zip.x86_64 0:3.0-11.el7 will be installed
    default: --> Running transaction check
    default: ---> Package gpm-libs.x86_64 0:1.20.7-5.el7 will be installed
    default: ---> Package libcurl.x86_64 0:7.29.0-51.el7 will be updated
    default: ---> Package libcurl.x86_64 0:7.29.0-51.el7_6.3 will be an update
    default: ---> Package perl-Error.noarch 1:0.17020-2.el7 will be installed
    default: ---> Package perl-Git.noarch 0:1.8.3.1-20.el7 will be installed
    default: ---> Package perl-TermReadKey.x86_64 0:2.30-20.el7 will be installed
    default: ---> Package vim-common.x86_64 2:7.4.160-6.el7_6 will be installed
    default: --> Processing Dependency: vim-filesystem for package: 2:vim-common-7.4.160-6.el7_6.x86_64
    default: --> Running transaction check
    default: ---> Package vim-filesystem.x86_64 2:7.4.160-6.el7_6 will be installed
    default: --> Finished Dependency Resolution
    default: 
    default: Dependencies Resolved
    default: 
    default: ================================================================================
    default:  Package              Arch       Version                      Repository   Size
    default: ================================================================================
    default: Installing:
    default:  git                  x86_64     1.8.3.1-20.el7               updates     4.4 M
    default:  net-tools            x86_64     2.0-0.24.20131004git.el7     base        306 k
    default:  unzip                x86_64     6.0-19.el7                   base        170 k
    default:  vim-enhanced         x86_64     2:7.4.160-6.el7_6            updates     1.0 M
    default:  zip                  x86_64     3.0-11.el7                   base        260 k
    default: Updating:
    default:  curl                 x86_64     7.29.0-51.el7_6.3            updates     269 k
    default: Installing for dependencies:
    default:  gpm-libs             x86_64     1.20.7-5.el7                 base         32 k
    default:  perl-Error           noarch     1:0.17020-2.el7              base         32 k
    default:  perl-Git             noarch     1.8.3.1-20.el7               updates      55 k
    default:  perl-TermReadKey     x86_64     2.30-20.el7                  base         31 k
    default:  vim-common           x86_64     2:7.4.160-6.el7_6            updates     5.9 M
    default:  vim-filesystem       x86_64     2:7.4.160-6.el7_6            updates      10 k
    default: Updating for dependencies:
    default:  libcurl              x86_64     7.29.0-51.el7_6.3            updates     222 k
    default: 
    default: Transaction Summary
    default: ================================================================================
    default: Install  5 Packages (+6 Dependent packages)
    default: Upgrade  1 Package  (+1 Dependent package)
    default: Total download size: 13 M
    default: Downloading packages:
    default: Delta RPMs reduced 492 k of updates to 206 k (57% saved)
    default: --------------------------------------------------------------------------------
    default: Total                                              7.5 MB/s |  12 MB  00:01     
    default: Running transaction check
    default: Running transaction test
    default: Transaction test succeeded
    default: Running transaction
    default:   Updating   : libcurl-7.29.0-51.el7_6.3.x86_64                            1/15
    default:  
    default:   Installing : 1:perl-Error-0.17020-2.el7.noarch                           2/15
    default:  
    default:   Installing : gpm-libs-1.20.7-5.el7.x86_64                                3/15
    default:  
    default:   Installing : 2:vim-filesystem-7.4.160-6.el7_6.x86_64                     4/15
    default:  
    default:   Installing : 2:vim-common-7.4.160-6.el7_6.x86_64                         5/15
    default:  
    default:   Installing : perl-TermReadKey-2.30-20.el7.x86_64                         6/15
    default:  
    default:   Installing : git-1.8.3.1-20.el7.x86_64                                   7/15
    default:  
    default:   Installing : perl-Git-1.8.3.1-20.el7.noarch                              8/15
    default:  
    default:   Installing : 2:vim-enhanced-7.4.160-6.el7_6.x86_64                       9/15
    default:  
    default:   Updating   : curl-7.29.0-51.el7_6.3.x86_64                              10/15
    default:  
    default:   Installing : unzip-6.0-19.el7.x86_64                                    11/15
    default:  
    default:   Installing : zip-3.0-11.el7.x86_64                                      12/15
    default:  
    default:   Installing : net-tools-2.0-0.24.20131004git.el7.x86_64                  13/15
    default:  
    default:   Cleanup    : curl-7.29.0-51.el7.x86_64                                  14/15
    default:  
    default:   Cleanup    : libcurl-7.29.0-51.el7.x86_64                               15/15
    default:  
    default:   Verifying  : 2:vim-enhanced-7.4.160-6.el7_6.x86_64                       1/15
    default:  
    default:   Verifying  : net-tools-2.0-0.24.20131004git.el7.x86_64                   2/15
    default:  
    default:   Verifying  : perl-Git-1.8.3.1-20.el7.noarch                              3/15
    default:  
    default:   Verifying  : 1:perl-Error-0.17020-2.el7.noarch                           4/15
    default:  
    default:   Verifying  : perl-TermReadKey-2.30-20.el7.x86_64                         5/15
    default:  
    default:   Verifying  : git-1.8.3.1-20.el7.x86_64                                   6/15
    default:  
    default:   Verifying  : zip-3.0-11.el7.x86_64                                       7/15
    default:  
    default:   Verifying  : curl-7.29.0-51.el7_6.3.x86_64                               8/15
    default:  
    default:   Verifying  : 2:vim-filesystem-7.4.160-6.el7_6.x86_64                     9/15
    default:  
    default:   Verifying  : unzip-6.0-19.el7.x86_64                                    10/15
    default:  
    default:   Verifying  : 2:vim-common-7.4.160-6.el7_6.x86_64                        11/15
    default:  
    default:   Verifying  : libcurl-7.29.0-51.el7_6.3.x86_64                           12/15
    default:  
    default:   Verifying  : gpm-libs-1.20.7-5.el7.x86_64                               13/15
    default:  
    default:   Verifying  : curl-7.29.0-51.el7.x86_64                                  14/15
    default:  
    default:   Verifying  : libcurl-7.29.0-51.el7.x86_64                               15/15
    default:  
    default: 
    default: Installed:
    default:   git.x86_64 0:1.8.3.1-20.el7    net-tools.x86_64 0:2.0-0.24.20131004git.el7   
    default:   unzip.x86_64 0:6.0-19.el7      vim-enhanced.x86_64 2:7.4.160-6.el7_6         
    default:   zip.x86_64 0:3.0-11.el7       
    default: 
    default: Dependency Installed:
    default:   gpm-libs.x86_64 0:1.20.7-5.el7       perl-Error.noarch 1:0.17020-2.el7       
    default:   perl-Git.noarch 0:1.8.3.1-20.el7     perl-TermReadKey.x86_64 0:2.30-20.el7   
    default:   vim-common.x86_64 2:7.4.160-6.el7_6  vim-filesystem.x86_64 2:7.4.160-6.el7_6 
    default: 
    default: Updated:
    default:   curl.x86_64 0:7.29.0-51.el7_6.3                                               
    default: 
    default: Dependency Updated:
    default:   libcurl.x86_64 0:7.29.0-51.el7_6.3                                            
    default: Complete!
    default: Loaded plugins: fastestmirror
    default: Examining jenkins-2.176.2-1.1.noarch.rpm: jenkins-2.176.2-1.1.noarch
    default: Marking jenkins-2.176.2-1.1.noarch.rpm to be installed
    default: Resolving Dependencies
    default: --> Running transaction check
    default: ---> Package jenkins.noarch 0:2.176.2-1.1 will be installed
    default: --> Finished Dependency Resolution
    default: 
    default: Dependencies Resolved
    default: 
    default: ================================================================================
    default:  Package     Arch       Version           Repository                       Size
    default: ================================================================================
    default: Installing:
    default:  jenkins     noarch     2.176.2-1.1       /jenkins-2.176.2-1.1.noarch      74 M
    default: 
    default: Transaction Summary
    default: ================================================================================
    default: Install  1 Package
    default: Total size: 74 M
    default: Installed size: 74 M
    default: Downloading packages:
    default: Running transaction check
    default: Running transaction test
    default: Transaction test succeeded
    default: Running transaction
    default:   Installing : jenkins-2.176.2-1.1.noarch                                   1/1
    default:  
    default:   Verifying  : jenkins-2.176.2-1.1.noarch                                   1/1
    default:  
    default: 
    default: Installed:
    default:   jenkins.noarch 0:2.176.2-1.1                                                  
    default: Complete!
    default: Stopping jenkins (via systemctl):  
    default: [  OK  ]
    default: Archive:  /tmp/package/jenkins.zip
    default:    creating: /var/lib/jenkins/
    default:    creating: /var/lib/jenkins/.java/
    default:    creating: /var/lib/jenkins/.java/fonts/
    default:    creating: /var/lib/jenkins/.java/fonts/1.8.0_222/
    default:   inflating: /var/lib/jenkins/.java/fonts/1.8.0_222/fcinfo-1-cicd-RedHat-7.6.1810-en.properties  
    default:   inflating: /var/lib/jenkins/.java/fonts/1.8.0_222/fcinfo-1-auto-RedHat-7.6.1810-en.properties  
    default:   inflating: /var/lib/jenkins/secret.key  
    default:  extracting: /var/lib/jenkins/secret.key.not-so-secret  
    default:    creating: /var/lib/jenkins/plugins/
    default:   inflating: /var/lib/jenkins/plugins/jdk-tool.jpi  
    default:   inflating: /var/lib/jenkins/plugins/branch-api.jpi  
    default:   inflating: /var/lib/jenkins/plugins/script-security.jpi  
    default:   inflating: /var/lib/jenkins/plugins/lockable-resources.jpi  
    default:   inflating: /var/lib/jenkins/plugins/command-launcher.jpi  
    default:    creating: /var/lib/jenkins/plugins/momentjs/
    default:    creating: /var/lib/jenkins/plugins/momentjs/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/momentjs/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/momentjs/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/momentjs/META-INF/maven/org.jenkins-ci.ui/
    default:    creating: /var/lib/jenkins/plugins/momentjs/META-INF/maven/org.jenkins-ci.ui/momentjs/
    default:   inflating: /var/lib/jenkins/plugins/momentjs/META-INF/maven/org.jenkins-ci.ui/momentjs/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/momentjs/META-INF/maven/org.jenkins-ci.ui/momentjs/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/momentjs/jsmodules/
    default:   inflating: /var/lib/jenkins/plugins/momentjs/jsmodules/momentjs2.js  
    default: 
    default:    creating: /var/lib/jenkins/plugins/momentjs/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/momentjs/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/momentjs/WEB-INF/lib/momentjs.jar  
    default:   inflating: /var/lib/jenkins/plugins/momentjs/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/momentjs/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/structs.jpi  
    default:   inflating: /var/lib/jenkins/plugins/workflow-aggregator.jpi  
    default:   inflating: /var/lib/jenkins/plugins/workflow-step-api.jpi  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/token-macro.jpi  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-scm-step.jpi  
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-view/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-view/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-view/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-view/META-INF/maven/org.jenkins-ci.plugins.pipeline-stage-view/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-view/META-INF/maven/org.jenkins-ci.plugins.pipeline-stage-view/pipeline-stage-view/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/META-INF/maven/org.jenkins-ci.plugins.pipeline-stage-view/pipeline-stage-view/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/META-INF/maven/org.jenkins-ci.plugins.pipeline-stage-view/pipeline-stage-view/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-view/images/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/images/loading.gif  
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-view/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-view/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/WEB-INF/lib/pipeline-stage-view.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/WEB-INF/licenses.xml  
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-view/jsmodules/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/jsmodules/stageview.js  
    default: 
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-view/fonts/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/fonts/glyphicons-halflings-regular.woff  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/fonts/glyphicons-halflings-regular.eot  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/fonts/glyphicons-halflings-regular.ttf  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view/fonts/glyphicons-halflings-regular.svg  
    default: 
    default:  extracting: /var/lib/jenkins/plugins/pipeline-stage-view/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/credentials.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/branch-api/
    default:    creating: /var/lib/jenkins/plugins/branch-api/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/branch-api/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/branch-api/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/branch-api/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/branch-api/META-INF/maven/org.jenkins-ci.plugins/branch-api/
    default:   inflating: /var/lib/jenkins/plugins/branch-api/META-INF/maven/org.jenkins-ci.plugins/branch-api/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/branch-api/META-INF/maven/org.jenkins-ci.plugins/branch-api/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/branch-api/images/
    default:    creating: /var/lib/jenkins/plugins/branch-api/images/16x16/
    default:  extracting: /var/lib/jenkins/plugins/branch-api/images/16x16/organization-folder.png  
    default:    creating: /var/lib/jenkins/plugins/branch-api/images/32x32/
    default:  extracting: /var/lib/jenkins/plugins/branch-api/images/32x32/organization-folder.png  
    default:    creating: /var/lib/jenkins/plugins/branch-api/images/24x24/
    default:  extracting: /var/lib/jenkins/plugins/branch-api/images/24x24/organization-folder.png  
    default:    creating: /var/lib/jenkins/plugins/branch-api/images/48x48/
    default:  extracting: /var/lib/jenkins/plugins/branch-api/images/48x48/organization-folder.png  
    default:    creating: /var/lib/jenkins/plugins/branch-api/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/branch-api/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/branch-api/WEB-INF/lib/branch-api.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/branch-api/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/branch-api/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/pipeline-build-step/
    default:    creating: /var/lib/jenkins/plugins/pipeline-build-step/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-build-step/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-build-step/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-build-step/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-build-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-build-step/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-build-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-build-step/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-build-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-build-step/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-build-step/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-build-step/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-build-step/WEB-INF/lib/pipeline-build-step.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-build-step/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-build-step/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api.jpi  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/config-file-provider.jpi  
    default:   inflating: /var/lib/jenkins/plugins/ssh-credentials.jpi  
    default:   inflating: /var/lib/jenkins/plugins/jsch.jpi  
    default:    creating: /var/lib/jenkins/plugins/plain-credentials/
    default:    creating: /var/lib/jenkins/plugins/plain-credentials/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/plain-credentials/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/plain-credentials/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/plain-credentials/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/plain-credentials/META-INF/maven/org.jenkins-ci.plugins/plain-credentials/
    default:   inflating: /var/lib/jenkins/plugins/plain-credentials/META-INF/maven/org.jenkins-ci.plugins/plain-credentials/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/plain-credentials/META-INF/maven/org.jenkins-ci.plugins/plain-credentials/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/plain-credentials/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/plain-credentials/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/plain-credentials/WEB-INF/lib/plain-credentials.jar  
    default:   inflating: /var/lib/jenkins/plugins/plain-credentials/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/plain-credentials/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/git-client.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/credentials-binding/
    default:    creating: /var/lib/jenkins/plugins/credentials-binding/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/credentials-binding/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/credentials-binding/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/credentials-binding/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/credentials-binding/META-INF/maven/org.jenkins-ci.plugins/credentials-binding/
    default:   inflating: /var/lib/jenkins/plugins/credentials-binding/META-INF/maven/org.jenkins-ci.plugins/credentials-binding/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/credentials-binding/META-INF/maven/org.jenkins-ci.plugins/credentials-binding/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/credentials-binding/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/credentials-binding/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/credentials-binding/WEB-INF/lib/credentials-binding.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/credentials-binding/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/credentials-binding/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/scm-api.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/git/
    default:    creating: /var/lib/jenkins/plugins/git/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/git/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/git/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/git/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/git/META-INF/maven/org.jenkins-ci.plugins/git/
    default:   inflating: /var/lib/jenkins/plugins/git/META-INF/maven/org.jenkins-ci.plugins/git/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/git/META-INF/maven/org.jenkins-ci.plugins/git/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/git/icons/
    default:   inflating: /var/lib/jenkins/plugins/git/icons/git-32x32.png  
    default:   inflating: /var/lib/jenkins/plugins/git/icons/git-22x22.png  
    default:   inflating: /var/lib/jenkins/plugins/git/icons/git-48x48.png  
    default:    creating: /var/lib/jenkins/plugins/git/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/git/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/git/WEB-INF/lib/httpcore-4.4.9.jar  
    default:   inflating: /var/lib/jenkins/plugins/git/WEB-INF/lib/commons-net-3.6.jar  
    default:   inflating: /var/lib/jenkins/plugins/git/WEB-INF/lib/git.jar  
    default:   inflating: /var/lib/jenkins/plugins/git/WEB-INF/lib/httpclient-4.5.5.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/git/WEB-INF/lib/joda-time-2.9.5.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/git/WEB-INF/licenses.xml  
    default:   inflating: /var/lib/jenkins/plugins/git/extraRepo.html  
    default:   inflating: /var/lib/jenkins/plugins/git/sparseCheckoutPaths.html  
    default:   inflating: /var/lib/jenkins/plugins/git/gitPublisher_ja.html  
    default:   inflating: /var/lib/jenkins/plugins/git/gitPublisher.html  
    default:  extracting: /var/lib/jenkins/plugins/git/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/display-url-api.jpi  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-api/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-api/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-api/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-api/META-INF/maven/org.jenkinsci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-api/META-INF/maven/org.jenkinsci.plugins/pipeline-model-api/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/META-INF/maven/org.jenkinsci.plugins/pipeline-model-api/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/META-INF/maven/org.jenkinsci.plugins/pipeline-model-api/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/lib/json-schema-core-1.0.4.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/lib/jackson-datatype-json-org-2.9.8.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/lib/rhino-1.7R4.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/lib/json-20171018.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/lib/joda-time-2.1.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/lib/mailapi-1.4.3.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/lib/json-schema-validator-2.0.4.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/lib/pipeline-model-api.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/lib/libphonenumber-5.3.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-model-api/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/mailer.jpi  
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/META-INF/maven/org.jenkins-ci.plugins/cloudbees-folder/
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/META-INF/maven/org.jenkins-ci.plugins/cloudbees-folder/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/META-INF/maven/org.jenkins-ci.plugins/cloudbees-folder/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/WEB-INF/lib/cloudbees-folder.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/WEB-INF/licenses.xml  
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/images/
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/images/24x24/
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/images/24x24/folder-disabled.png  
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/images/24x24/move.png  
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/images/24x24/folder.png  
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/images/16x16/
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/images/16x16/folder-disabled.png  
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/images/16x16/move.png  
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/images/16x16/folder.png  
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/images/48x48/
    default:  extracting: /var/lib/jenkins/plugins/cloudbees-folder/images/48x48/folder-disabled.png  
    default:  extracting: /var/lib/jenkins/plugins/cloudbees-folder/images/48x48/move.png  
    default:  extracting: /var/lib/jenkins/plugins/cloudbees-folder/images/48x48/folder.png  
    default:    creating: /var/lib/jenkins/plugins/cloudbees-folder/images/32x32/
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder/images/32x32/folder-disabled.png  
    default:  extracting: /var/lib/jenkins/plugins/cloudbees-folder/images/32x32/move.png  
    default:  extracting: /var/lib/jenkins/plugins/cloudbees-folder/images/32x32/folder.png  
    default:  extracting: /var/lib/jenkins/plugins/cloudbees-folder/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/workflow-api.jpi  
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-multibranch/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-multibranch/
    default:   inflating: /var/lib/jenkins/plugins/workflow-multibranch/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-multibranch/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/workflow-multibranch/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-multibranch/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-multibranch/WEB-INF/lib/workflow-multibranch.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-multibranch/WEB-INF/licenses.xml  
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/images/
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/images/16x16/
    default:   inflating: /var/lib/jenkins/plugins/workflow-multibranch/images/16x16/pipelinemultibranchproject.png  
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/images/32x32/
    default:  extracting: /var/lib/jenkins/plugins/workflow-multibranch/images/32x32/pipelinemultibranchproject.png  
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/images/48x48/
    default:  extracting: /var/lib/jenkins/plugins/workflow-multibranch/images/48x48/pipelinemultibranchproject.png  
    default:    creating: /var/lib/jenkins/plugins/workflow-multibranch/images/24x24/
    default:  extracting: /var/lib/jenkins/plugins/workflow-multibranch/images/24x24/pipelinemultibranchproject.png  
    default:  extracting: /var/lib/jenkins/plugins/workflow-multibranch/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/junit.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/lockable-resources/
    default:    creating: /var/lib/jenkins/plugins/lockable-resources/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/lockable-resources/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/lockable-resources/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/lockable-resources/META-INF/maven/org.6wind.jenkins/
    default:    creating: /var/lib/jenkins/plugins/lockable-resources/META-INF/maven/org.6wind.jenkins/lockable-resources/
    default:   inflating: /var/lib/jenkins/plugins/lockable-resources/META-INF/maven/org.6wind.jenkins/lockable-resources/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/lockable-resources/META-INF/maven/org.6wind.jenkins/lockable-resources/pom.properties  
    default: 
    default:    creating: /var/lib/jenkins/plugins/lockable-resources/img/
    default:  extracting: /var/lib/jenkins/plugins/lockable-resources/img/device-24x24.png  
    default: 
    default:    creating: /var/lib/jenkins/plugins/lockable-resources/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/lockable-resources/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/lockable-resources/WEB-INF/lib/annotation-indexer-1.4.jar  
    default:   inflating: /var/lib/jenkins/plugins/lockable-resources/WEB-INF/lib/lockable-resources.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/lockable-resources/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/lockable-resources/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/bouncycastle-api.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/workflow-aggregator/
    default:    creating: /var/lib/jenkins/plugins/workflow-aggregator/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-aggregator/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-aggregator/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-aggregator/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-aggregator/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-aggregator/
    default:   inflating: /var/lib/jenkins/plugins/workflow-aggregator/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-aggregator/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/workflow-aggregator/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-aggregator/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/workflow-aggregator/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-aggregator/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-aggregator/WEB-INF/lib/workflow-aggregator.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-aggregator/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/workflow-aggregator/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project.jpi  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-extensions.jpi  
    default:    creating: /var/lib/jenkins/plugins/token-macro/
    default:    creating: /var/lib/jenkins/plugins/token-macro/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/token-macro/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/token-macro/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/token-macro/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/token-macro/META-INF/maven/org.jenkins-ci.plugins/token-macro/
    default:   inflating: /var/lib/jenkins/plugins/token-macro/META-INF/maven/org.jenkins-ci.plugins/token-macro/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/token-macro/META-INF/maven/org.jenkins-ci.plugins/token-macro/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/token-macro/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/token-macro/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/token-macro/WEB-INF/lib/json-path-2.4.0.jar  
    default:   inflating: /var/lib/jenkins/plugins/token-macro/WEB-INF/lib/json-smart-2.3.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/token-macro/WEB-INF/lib/token-macro.jar  
    default:   inflating: /var/lib/jenkins/plugins/token-macro/WEB-INF/lib/parboiled-core-1.1.8.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/token-macro/WEB-INF/lib/parboiled-java-1.1.8.jar  
    default:   inflating: /var/lib/jenkins/plugins/token-macro/WEB-INF/lib/accessors-smart-1.2.jar  
    default:   inflating: /var/lib/jenkins/plugins/token-macro/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/token-macro/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/workflow-support.jpi  
    default:    creating: /var/lib/jenkins/plugins/docker-commons/
    default:    creating: /var/lib/jenkins/plugins/docker-commons/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/docker-commons/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/docker-commons/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/docker-commons/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/docker-commons/META-INF/maven/org.jenkins-ci.plugins/docker-commons/
    default:   inflating: /var/lib/jenkins/plugins/docker-commons/META-INF/maven/org.jenkins-ci.plugins/docker-commons/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/docker-commons/META-INF/maven/org.jenkins-ci.plugins/docker-commons/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/docker-commons/images/
    default:    creating: /var/lib/jenkins/plugins/docker-commons/images/16x16/
    default:  extracting: /var/lib/jenkins/plugins/docker-commons/images/16x16/docker.png  
    default:    creating: /var/lib/jenkins/plugins/docker-commons/images/24x24/
    default:  extracting: /var/lib/jenkins/plugins/docker-commons/images/24x24/docker.png  
    default:    creating: /var/lib/jenkins/plugins/docker-commons/images/32x32/
    default:  extracting: /var/lib/jenkins/plugins/docker-commons/images/32x32/docker.png  
    default:    creating: /var/lib/jenkins/plugins/docker-commons/images/48x48/
    default:  extracting: /var/lib/jenkins/plugins/docker-commons/images/48x48/docker.png  
    default:    creating: /var/lib/jenkins/plugins/docker-commons/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/docker-commons/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/docker-commons/WEB-INF/lib/docker-commons.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/docker-commons/WEB-INF/lib/multiline-secrets-ui-1.0.jar  
    default:   inflating: /var/lib/jenkins/plugins/docker-commons/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/docker-commons/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/workflow-job.jpi  
    default:    creating: /var/lib/jenkins/plugins/config-file-provider/
    default:    creating: /var/lib/jenkins/plugins/config-file-provider/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/config-file-provider/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/config-file-provider/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/config-file-provider/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/config-file-provider/META-INF/maven/org.jenkins-ci.plugins/config-file-provider/
    default:   inflating: /var/lib/jenkins/plugins/config-file-provider/META-INF/maven/org.jenkins-ci.plugins/config-file-provider/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/config-file-provider/META-INF/maven/org.jenkins-ci.plugins/config-file-provider/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/config-file-provider/images/
    default:   inflating: /var/lib/jenkins/plugins/config-file-provider/images/mvn_s.svg  
    default:  extracting: /var/lib/jenkins/plugins/config-file-provider/images/cfg_logo.png  
    default:   inflating: /var/lib/jenkins/plugins/config-file-provider/images/cfg_logo.svg  
    default:  extracting: /var/lib/jenkins/plugins/config-file-provider/images/mvn_s.png  
    default:    creating: /var/lib/jenkins/plugins/config-file-provider/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/config-file-provider/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/config-file-provider/WEB-INF/lib/config-file-provider.jar  
    default:   inflating: /var/lib/jenkins/plugins/config-file-provider/WEB-INF/lib/findbugs-annotations-1.3.9-1.jar  
    default:   inflating: /var/lib/jenkins/plugins/config-file-provider/WEB-INF/licenses.xml  
    default:   inflating: /var/lib/jenkins/plugins/config-file-provider/help-globalConfig.html  
    default:  extracting: /var/lib/jenkins/plugins/config-file-provider/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/jdk-tool/
    default:    creating: /var/lib/jenkins/plugins/jdk-tool/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/jdk-tool/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/jdk-tool/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/jdk-tool/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/jdk-tool/META-INF/maven/org.jenkins-ci.plugins/jdk-tool/
    default:   inflating: /var/lib/jenkins/plugins/jdk-tool/META-INF/maven/org.jenkins-ci.plugins/jdk-tool/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/jdk-tool/META-INF/maven/org.jenkins-ci.plugins/jdk-tool/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/jdk-tool/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/jdk-tool/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/jdk-tool/WEB-INF/lib/jdk-tool.jar  
    default:   inflating: /var/lib/jenkins/plugins/jdk-tool/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/jdk-tool/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/script-security/
    default:    creating: /var/lib/jenkins/plugins/script-security/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/script-security/META-INF/MANIFEST.MF  
    default: 
    default:    creating: /var/lib/jenkins/plugins/script-security/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/script-security/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/script-security/META-INF/maven/org.jenkins-ci.plugins/script-security/
    default:   inflating: /var/lib/jenkins/plugins/script-security/META-INF/maven/org.jenkins-ci.plugins/script-security/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/script-security/META-INF/maven/org.jenkins-ci.plugins/script-security/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/script-security/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/script-security/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/script-security/WEB-INF/lib/groovy-sandbox-1.22.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/script-security/WEB-INF/lib/error_prone_annotations-2.3.3.jar  
    default:   inflating: /var/lib/jenkins/plugins/script-security/WEB-INF/lib/checker-qual-2.6.0.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/script-security/WEB-INF/lib/script-security.jar  
    default:   inflating: /var/lib/jenkins/plugins/script-security/WEB-INF/lib/caffeine-2.7.0.jar  
    default:   inflating: /var/lib/jenkins/plugins/script-security/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/script-security/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/command-launcher/
    default:    creating: /var/lib/jenkins/plugins/command-launcher/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/command-launcher/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/command-launcher/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/command-launcher/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/command-launcher/META-INF/maven/org.jenkins-ci.plugins/command-launcher/
    default:   inflating: /var/lib/jenkins/plugins/command-launcher/META-INF/maven/org.jenkins-ci.plugins/command-launcher/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/command-launcher/META-INF/maven/org.jenkins-ci.plugins/command-launcher/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/command-launcher/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/command-launcher/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/command-launcher/WEB-INF/lib/command-launcher.jar  
    default:   inflating: /var/lib/jenkins/plugins/command-launcher/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/command-launcher/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/structs/
    default:    creating: /var/lib/jenkins/plugins/structs/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/structs/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/structs/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/structs/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/structs/META-INF/maven/org.jenkins-ci.plugins/structs/
    default:   inflating: /var/lib/jenkins/plugins/structs/META-INF/maven/org.jenkins-ci.plugins/structs/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/structs/META-INF/maven/org.jenkins-ci.plugins/structs/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/structs/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/structs/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/structs/WEB-INF/lib/structs.jar  
    default:   inflating: /var/lib/jenkins/plugins/structs/WEB-INF/lib/symbol-annotation-1.20.jar  
    default:   inflating: /var/lib/jenkins/plugins/structs/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/structs/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/workflow-step-api/
    default:    creating: /var/lib/jenkins/plugins/workflow-step-api/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-step-api/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-step-api/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-step-api/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-step-api/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-step-api/
    default:   inflating: /var/lib/jenkins/plugins/workflow-step-api/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-step-api/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/workflow-step-api/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-step-api/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/workflow-step-api/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-step-api/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-step-api/WEB-INF/lib/workflow-step-api.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-step-api/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/workflow-step-api/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/workflow-scm-step/
    default:    creating: /var/lib/jenkins/plugins/workflow-scm-step/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-scm-step/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-scm-step/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-scm-step/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-scm-step/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-scm-step/
    default:   inflating: /var/lib/jenkins/plugins/workflow-scm-step/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-scm-step/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/workflow-scm-step/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-scm-step/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/workflow-scm-step/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-scm-step/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-scm-step/WEB-INF/lib/workflow-scm-step.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-scm-step/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/workflow-scm-step/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/credentials/
    default:    creating: /var/lib/jenkins/plugins/credentials/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/credentials/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/credentials/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/credentials/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/credentials/META-INF/maven/org.jenkins-ci.plugins/credentials/
    default:   inflating: /var/lib/jenkins/plugins/credentials/META-INF/maven/org.jenkins-ci.plugins/credentials/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/credentials/META-INF/maven/org.jenkins-ci.plugins/credentials/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/credentials/images/
    default:    creating: /var/lib/jenkins/plugins/credentials/images/16x16/
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/16x16/domain.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/16x16/new-credential.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/16x16/certificate.png  
    default:   inflating: /var/lib/jenkins/plugins/credentials/images/16x16/move.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/16x16/credentials.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/16x16/folder-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/16x16/credential.png  
    default:   inflating: /var/lib/jenkins/plugins/credentials/images/16x16/userpass.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/16x16/system-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/16x16/user-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/16x16/new-domain.png  
    default:    creating: /var/lib/jenkins/plugins/credentials/images/32x32/
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/domain.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/new-credential.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/certificate.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/move.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/credentials.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/folder-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/credential.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/userpass.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/system-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/user-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/32x32/new-domain.png  
    default:    creating: /var/lib/jenkins/plugins/credentials/images/24x24/
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/24x24/domain.png  
    default: 
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/24x24/new-credential.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/24x24/certificate.png  
    default:   inflating: /var/lib/jenkins/plugins/credentials/images/24x24/move.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/24x24/credentials.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/24x24/folder-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/24x24/credential.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/24x24/userpass.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/24x24/system-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/24x24/user-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/24x24/new-domain.png  
    default:    creating: /var/lib/jenkins/plugins/credentials/images/48x48/
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/domain.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/new-credential.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/certificate.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/move.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/credentials.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/folder-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/credential.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/userpass.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/system-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/user-store.png  
    default:  extracting: /var/lib/jenkins/plugins/credentials/images/48x48/new-domain.png  
    default:    creating: /var/lib/jenkins/plugins/credentials/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/credentials/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/credentials/WEB-INF/lib/org.abego.treelayout.core-1.0.1.jar  
    default:   inflating: /var/lib/jenkins/plugins/credentials/WEB-INF/lib/credentials.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/credentials/WEB-INF/lib/antlr4-runtime-4.5.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/credentials/WEB-INF/licenses.xml  
    default:    creating: /var/lib/jenkins/plugins/credentials/help/
    default:    creating: /var/lib/jenkins/plugins/credentials/help/domain/
    default:   inflating: /var/lib/jenkins/plugins/credentials/help/domain/description_fr.html  
    default:   inflating: /var/lib/jenkins/plugins/credentials/help/domain/name_ja.html  
    default:   inflating: /var/lib/jenkins/plugins/credentials/help/domain/specification_ja.html  
    default:   inflating: /var/lib/jenkins/plugins/credentials/help/domain/specification.html  
    default:   inflating: /var/lib/jenkins/plugins/credentials/help/domain/description.html  
    default:   inflating: /var/lib/jenkins/plugins/credentials/help/domain/name_fr.html  
    default:   inflating: /var/lib/jenkins/plugins/credentials/help/domain/description_ja.html  
    default:   inflating: /var/lib/jenkins/plugins/credentials/help/domain/name.html  
    default:   inflating: /var/lib/jenkins/plugins/credentials/help/domain/specification_fr.html  
    default:  extracting: /var/lib/jenkins/plugins/credentials/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/
    default:    creating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/META-INF/maven/org.jenkins-ci.plugins/apache-httpcomponents-client-4-api/
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/META-INF/maven/org.jenkins-ci.plugins/apache-httpcomponents-client-4-api/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/META-INF/maven/org.jenkins-ci.plugins/apache-httpcomponents-client-4-api/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/lib/apache-httpcomponents-client-4-api.jar  
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/lib/fluent-hc-4.5.5.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/lib/httpasyncclient-4.1.3.jar  
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/lib/httpasyncclient-cache-4.1.3.jar  
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/lib/httpclient-4.5.5.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/lib/httpclient-cache-4.5.5.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/lib/httpcore-4.4.9.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/lib/httpcore-nio-4.4.6.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/lib/httpmime-4.5.5.jar  
    default:   inflating: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/apache-httpcomponents-client-4-api/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/ssh-credentials/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/META-INF/maven/org.jenkins-ci.plugins/ssh-credentials/
    default:   inflating: /var/lib/jenkins/plugins/ssh-credentials/META-INF/maven/org.jenkins-ci.plugins/ssh-credentials/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/ssh-credentials/META-INF/maven/org.jenkins-ci.plugins/ssh-credentials/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/images/
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/images/16x16/
    default:   inflating: /var/lib/jenkins/plugins/ssh-credentials/images/16x16/ssh-key.png  
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/images/32x32/
    default:  extracting: /var/lib/jenkins/plugins/ssh-credentials/images/32x32/ssh-key.png  
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/images/24x24/
    default:  extracting: /var/lib/jenkins/plugins/ssh-credentials/images/24x24/ssh-key.png  
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/images/48x48/
    default:  extracting: /var/lib/jenkins/plugins/ssh-credentials/images/48x48/ssh-key.png  
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/ssh-credentials/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/ssh-credentials/WEB-INF/lib/ssh-credentials.jar  
    default:   inflating: /var/lib/jenkins/plugins/ssh-credentials/WEB-INF/lib/multiline-secrets-ui-1.0.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ssh-credentials/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/ssh-credentials/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/jsch/
    default:    creating: /var/lib/jenkins/plugins/jsch/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/jsch/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/jsch/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/jsch/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/jsch/META-INF/maven/org.jenkins-ci.plugins/jsch/
    default:   inflating: /var/lib/jenkins/plugins/jsch/META-INF/maven/org.jenkins-ci.plugins/jsch/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/jsch/META-INF/maven/org.jenkins-ci.plugins/jsch/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/jsch/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/jsch/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/jsch/WEB-INF/lib/jsch.jar  
    default:   inflating: /var/lib/jenkins/plugins/jsch/WEB-INF/lib/jsch-0.1.55.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/jsch/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/jsch/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/git-client/
    default:    creating: /var/lib/jenkins/plugins/git-client/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/git-client/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/git-client/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/git-client/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/git-client/META-INF/maven/org.jenkins-ci.plugins/git-client/
    default:   inflating: /var/lib/jenkins/plugins/git-client/META-INF/maven/org.jenkins-ci.plugins/git-client/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/git-client/META-INF/maven/org.jenkins-ci.plugins/git-client/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/git-client/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/git-client/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/git-client/WEB-INF/lib/org.eclipse.jgit.http.apache-4.5.7.201904151645-r.jar  
    default:   inflating: /var/lib/jenkins/plugins/git-client/WEB-INF/lib/JavaEWAH-0.7.9.jar  
    default:   inflating: /var/lib/jenkins/plugins/git-client/WEB-INF/lib/jsr305-3.0.2.jar  
    default:   inflating: /var/lib/jenkins/plugins/git-client/WEB-INF/lib/org.eclipse.jgit-4.5.7.201904151645-r.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/git-client/WEB-INF/lib/git-client.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/git-client/WEB-INF/lib/org.eclipse.jgit.http.server-4.5.7.201904151645-r.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/git-client/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/git-client/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/scm-api/
    default:    creating: /var/lib/jenkins/plugins/scm-api/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/scm-api/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/scm-api/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/scm-api/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/scm-api/META-INF/maven/org.jenkins-ci.plugins/scm-api/
    default:   inflating: /var/lib/jenkins/plugins/scm-api/META-INF/maven/org.jenkins-ci.plugins/scm-api/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/scm-api/META-INF/maven/org.jenkins-ci.plugins/scm-api/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/scm-api/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/scm-api/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/scm-api/WEB-INF/lib/scm-api.jar  
    default:   inflating: /var/lib/jenkins/plugins/scm-api/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/scm-api/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/display-url-api/
    default:    creating: /var/lib/jenkins/plugins/display-url-api/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/display-url-api/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/display-url-api/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/display-url-api/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/display-url-api/META-INF/maven/org.jenkins-ci.plugins/display-url-api/
    default:   inflating: /var/lib/jenkins/plugins/display-url-api/META-INF/maven/org.jenkins-ci.plugins/display-url-api/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/display-url-api/META-INF/maven/org.jenkins-ci.plugins/display-url-api/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/display-url-api/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/display-url-api/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/display-url-api/WEB-INF/lib/display-url-api.jar  
    default:   inflating: /var/lib/jenkins/plugins/display-url-api/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/display-url-api/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/mailer/
    default:    creating: /var/lib/jenkins/plugins/mailer/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/mailer/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/mailer/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/mailer/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/mailer/META-INF/maven/org.jenkins-ci.plugins/mailer/
    default:   inflating: /var/lib/jenkins/plugins/mailer/META-INF/maven/org.jenkins-ci.plugins/mailer/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/mailer/META-INF/maven/org.jenkins-ci.plugins/mailer/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/mailer/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/mailer/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/mailer/WEB-INF/lib/mailer.jar  
    default:   inflating: /var/lib/jenkins/plugins/mailer/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/mailer/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/workflow-api/
    default:    creating: /var/lib/jenkins/plugins/workflow-api/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-api/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-api/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-api/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-api/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-api/
    default:   inflating: /var/lib/jenkins/plugins/workflow-api/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-api/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/workflow-api/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-api/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/workflow-api/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-api/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-api/WEB-INF/lib/workflow-api.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-api/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/workflow-api/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/junit/
    default:    creating: /var/lib/jenkins/plugins/junit/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/junit/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/junit/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/junit/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/junit/META-INF/maven/org.jenkins-ci.plugins/junit/
    default:   inflating: /var/lib/jenkins/plugins/junit/META-INF/maven/org.jenkins-ci.plugins/junit/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/junit/META-INF/maven/org.jenkins-ci.plugins/junit/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/junit/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/junit/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/junit/WEB-INF/lib/junit.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/junit/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/junit/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/bouncycastle-api/
    default:    creating: /var/lib/jenkins/plugins/bouncycastle-api/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/bouncycastle-api/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/bouncycastle-api/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/bouncycastle-api/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/bouncycastle-api/META-INF/maven/org.jenkins-ci.plugins/bouncycastle-api/
    default:   inflating: /var/lib/jenkins/plugins/bouncycastle-api/META-INF/maven/org.jenkins-ci.plugins/bouncycastle-api/pom.xml  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/bouncycastle-api/META-INF/maven/org.jenkins-ci.plugins/bouncycastle-api/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/bouncycastle-api/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/bouncycastle-api/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/bouncycastle-api/WEB-INF/lib/bouncycastle-api.jar  
    default:   inflating: /var/lib/jenkins/plugins/bouncycastle-api/WEB-INF/lib/bcprov-jdk15on-1.60.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/bouncycastle-api/WEB-INF/lib/bcpkix-jdk15on-1.60.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/bouncycastle-api/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/bouncycastle-api/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/matrix-project/
    default:    creating: /var/lib/jenkins/plugins/matrix-project/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/matrix-project/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/matrix-project/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/matrix-project/META-INF/maven/org.jenkins-ci.plugins/matrix-project/
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/META-INF/maven/org.jenkins-ci.plugins/matrix-project/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/META-INF/maven/org.jenkins-ci.plugins/matrix-project/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/matrix-project/images/
    default:    creating: /var/lib/jenkins/plugins/matrix-project/images/48x48/
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/images/48x48/matrixproject.png  
    default:    creating: /var/lib/jenkins/plugins/matrix-project/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/matrix-project/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/WEB-INF/lib/matrix-project.jar  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/WEB-INF/licenses.xml  
    default:    creating: /var/lib/jenkins/plugins/matrix-project/help/
    default:    creating: /var/lib/jenkins/plugins/matrix-project/help/matrix/
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/combinationfilter_ja.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/jdk_nl.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/jdk_ja.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/jdk_tr.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/jdk_de.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/combinationfilter.html  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/jdk_ru.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/jdk.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/axes_fr.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/combinationfilter_de.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/axes_pt_BR.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/axes_zh_TW.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/combinationfilter_fr.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/axes.html  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/axes_de.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/axes_ru.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/jdk_fr.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/combinationfilter_zh_TW.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/axes_ja.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/axes_nl.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/jdk_pt_BR.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/jdk_zh_TW.html  
    default:   inflating: /var/lib/jenkins/plugins/matrix-project/help/matrix/axes_tr.html  
    default:  extracting: /var/lib/jenkins/plugins/matrix-project/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/workflow-support/
    default:    creating: /var/lib/jenkins/plugins/workflow-support/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-support/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-support/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-support/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-support/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-support/
    default:   inflating: /var/lib/jenkins/plugins/workflow-support/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-support/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/workflow-support/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-support/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/workflow-support/images/
    default:    creating: /var/lib/jenkins/plugins/workflow-support/images/32x32/
    default:  extracting: /var/lib/jenkins/plugins/workflow-support/images/32x32/terminal.png  
    default:    creating: /var/lib/jenkins/plugins/workflow-support/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-support/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-support/WEB-INF/lib/jboss-marshalling-2.0.6.Final.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-support/WEB-INF/lib/workflow-support.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-support/WEB-INF/lib/jboss-marshalling-river-2.0.6.Final.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-support/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/workflow-support/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/gitlab-plugin/
    default:    creating: /var/lib/jenkins/plugins/gitlab-plugin/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/gitlab-plugin/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/gitlab-plugin/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/gitlab-plugin/META-INF/maven/org.jenkins-ci.plugins/gitlab-plugin/
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/META-INF/maven/org.jenkins-ci.plugins/gitlab-plugin/pom.xml  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/META-INF/maven/org.jenkins-ci.plugins/gitlab-plugin/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/gitlab-plugin/images/
    default:    creating: /var/lib/jenkins/plugins/gitlab-plugin/images/24x24/
    default:  extracting: /var/lib/jenkins/plugins/gitlab-plugin/images/24x24/gitlab.png  
    default:    creating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/jackson-module-jaxb-annotations-2.6.5.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/jboss-logging-3.1.4.GA.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/jackson-jaxrs-json-provider-2.6.5.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/httpcore-4.3.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/activation-1.1.1.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/guava-18.0.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/jackson-databind-2.6.5.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/jackson-core-2.6.5.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/httpclient-4.3.1.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/commons-codec-1.6.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/jboss-annotations-api_1.2_spec-1.0.0.Final.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/JavaEWAH-0.7.9.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/gitlab-plugin-1.5.12.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/jackson-jaxrs-base-2.6.5.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/resteasy-client-3.0.16.Final.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/jboss-jaxrs-api_2.0_spec-1.0.0.Final.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/jackson-annotations-2.6.0.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/resteasy-jaxrs-3.0.16.Final.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/symbol-annotation-1.5.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/annotation-indexer-1.9.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/jsch-0.1.50.jar  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/WEB-INF/lib/org.eclipse.jgit-3.5.2.201411120430-r.jar  
    default:    creating: /var/lib/jenkins/plugins/gitlab-plugin/help/
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/help/help-noteRegex.html  
    default:  extracting: /var/lib/jenkins/plugins/gitlab-plugin/help/help-connection.html  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/help/help-filterBranchesByRegex.html  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/help/help-project.html  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/help/help-secretToken.html  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/help/help-send-result-to-gitlab.html  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/help/help-messagesOnResult.html  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/help/help-pendingBuildName.html  
    default:  extracting: /var/lib/jenkins/plugins/gitlab-plugin/help/help-gitalbBranchBuilds.html  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/help/help-buildName.html  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/help/help-noBranchFiltering.html  
    default:   inflating: /var/lib/jenkins/plugins/gitlab-plugin/help/help-allowedBranches.html  
    default:  extracting: /var/lib/jenkins/plugins/gitlab-plugin/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/git.bak  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/git.jpi  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/git-server.jpi  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-milestone-step.jpi  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-extensions/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-extensions/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-extensions/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-extensions/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-extensions/META-INF/maven/org.jenkinsci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-extensions/META-INF/maven/org.jenkinsci.plugins/pipeline-model-extensions/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-extensions/META-INF/maven/org.jenkinsci.plugins/pipeline-model-extensions/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-extensions/META-INF/maven/org.jenkinsci.plugins/pipeline-model-extensions/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-extensions/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-extensions/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-extensions/WEB-INF/lib/pipeline-model-extensions.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-extensions/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-model-extensions/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/durable-task/
    default:    creating: /var/lib/jenkins/plugins/durable-task/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/durable-task/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/durable-task/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/durable-task/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/durable-task/META-INF/maven/org.jenkins-ci.plugins/durable-task/
    default:   inflating: /var/lib/jenkins/plugins/durable-task/META-INF/maven/org.jenkins-ci.plugins/durable-task/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/durable-task/META-INF/maven/org.jenkins-ci.plugins/durable-task/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/durable-task/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/durable-task/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/durable-task/WEB-INF/lib/durable-task.jar  
    default:   inflating: /var/lib/jenkins/plugins/durable-task/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/durable-task/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/workflow-basic-steps/
    default:    creating: /var/lib/jenkins/plugins/workflow-basic-steps/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-basic-steps/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-basic-steps/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-basic-steps/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-basic-steps/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-basic-steps/
    default:   inflating: /var/lib/jenkins/plugins/workflow-basic-steps/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-basic-steps/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/workflow-basic-steps/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-basic-steps/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/workflow-basic-steps/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-basic-steps/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-basic-steps/WEB-INF/lib/workflow-basic-steps.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-basic-steps/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/workflow-basic-steps/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/docker-workflow/
    default:    creating: /var/lib/jenkins/plugins/docker-workflow/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/docker-workflow/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/docker-workflow/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/docker-workflow/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/docker-workflow/META-INF/maven/org.jenkins-ci.plugins/docker-workflow/
    default:   inflating: /var/lib/jenkins/plugins/docker-workflow/META-INF/maven/org.jenkins-ci.plugins/docker-workflow/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/docker-workflow/META-INF/maven/org.jenkins-ci.plugins/docker-workflow/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/docker-workflow/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/docker-workflow/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/docker-workflow/WEB-INF/lib/docker-workflow.jar  
    default:   inflating: /var/lib/jenkins/plugins/docker-workflow/WEB-INF/lib/jsr305-3.0.2.jar  
    default:   inflating: /var/lib/jenkins/plugins/docker-workflow/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/docker-workflow/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps.jpi  
    default:    creating: /var/lib/jenkins/plugins/pipeline-rest-api/
    default:    creating: /var/lib/jenkins/plugins/pipeline-rest-api/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-rest-api/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-rest-api/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-rest-api/META-INF/maven/org.jenkins-ci.plugins.pipeline-stage-view/
    default:    creating: /var/lib/jenkins/plugins/pipeline-rest-api/META-INF/maven/org.jenkins-ci.plugins.pipeline-stage-view/pipeline-rest-api/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-rest-api/META-INF/maven/org.jenkins-ci.plugins.pipeline-stage-view/pipeline-rest-api/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-rest-api/META-INF/maven/org.jenkins-ci.plugins.pipeline-stage-view/pipeline-rest-api/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-rest-api/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-rest-api/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-rest-api/WEB-INF/lib/pipeline-rest-api.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-rest-api/WEB-INF/lib/groovy-sandbox-1.10.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-rest-api/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-rest-api/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-input-step.jpi  
    default:    creating: /var/lib/jenkins/plugins/git-server/
    default:    creating: /var/lib/jenkins/plugins/git-server/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/git-server/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/git-server/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/git-server/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/git-server/META-INF/maven/org.jenkins-ci.plugins/git-server/
    default:   inflating: /var/lib/jenkins/plugins/git-server/META-INF/maven/org.jenkins-ci.plugins/git-server/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/git-server/META-INF/maven/org.jenkins-ci.plugins/git-server/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/git-server/images/
    default:    creating: /var/lib/jenkins/plugins/git-server/images/48x48/
    default:  extracting: /var/lib/jenkins/plugins/git-server/images/48x48/git.png  
    default:    creating: /var/lib/jenkins/plugins/git-server/images/16x16/
    default:  extracting: /var/lib/jenkins/plugins/git-server/images/16x16/git.png  
    default:    creating: /var/lib/jenkins/plugins/git-server/images/24x24/
    default:  extracting: /var/lib/jenkins/plugins/git-server/images/24x24/git.png  
    default:    creating: /var/lib/jenkins/plugins/git-server/images/32x32/
    default:  extracting: /var/lib/jenkins/plugins/git-server/images/32x32/git.png  
    default:    creating: /var/lib/jenkins/plugins/git-server/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/git-server/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/git-server/WEB-INF/lib/git-server.jar  
    default:   inflating: /var/lib/jenkins/plugins/git-server/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/git-server/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-step.jpi  
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib.jpi  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-graph-analysis.jpi  
    default:    creating: /var/lib/jenkins/plugins/pipeline-milestone-step/
    default:    creating: /var/lib/jenkins/plugins/pipeline-milestone-step/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-milestone-step/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-milestone-step/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-milestone-step/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-milestone-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-milestone-step/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-milestone-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-milestone-step/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-milestone-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-milestone-step/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-milestone-step/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-milestone-step/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-milestone-step/WEB-INF/lib/pipeline-milestone-step.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-milestone-step/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-milestone-step/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-rest-api.jpi  
    default:   inflating: /var/lib/jenkins/plugins/handlebars.jpi  
    default:   inflating: /var/lib/jenkins/plugins/momentjs.jpi  
    default:    creating: /var/lib/jenkins/plugins/jquery-detached/
    default:    creating: /var/lib/jenkins/plugins/jquery-detached/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/jquery-detached/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/jquery-detached/META-INF/maven/org.jenkins-ci.ui/
    default:    creating: /var/lib/jenkins/plugins/jquery-detached/META-INF/maven/org.jenkins-ci.ui/jquery-detached/
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/META-INF/maven/org.jenkins-ci.ui/jquery-detached/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/META-INF/maven/org.jenkins-ci.ui/jquery-detached/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/
    default:    creating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/
    default:    creating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-bg_diagonals-thick_18_b81900_40x40.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-bg_diagonals-thick_20_666666_40x40.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-bg_flat_10_000000_40x100.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-bg_glass_100_f6f6f6_1x400.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-bg_glass_100_fdf5ce_1x400.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-bg_glass_65_ffffff_1x400.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-bg_gloss-wave_35_f6a828_500x100.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-bg_highlight-soft_100_eeeeee_1x100.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-bg_highlight-soft_75_ffe45c_1x100.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-icons_222222_256x240.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-icons_228ef1_256x240.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-icons_ef8c08_256x240.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-icons_ffd27a_256x240.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/images/ui-icons_ffffff_256x240.png  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1/style.css  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jquery2.js  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/jsmodules/jqueryui1.js  
    default: 
    default:    creating: /var/lib/jenkins/plugins/jquery-detached/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/jquery-detached/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/WEB-INF/lib/jquery-detached.jar  
    default:   inflating: /var/lib/jenkins/plugins/jquery-detached/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/jquery-detached/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-view.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/jackson2-api/
    default:    creating: /var/lib/jenkins/plugins/jackson2-api/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/jackson2-api/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/jackson2-api/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/jackson2-api/META-INF/maven/org.jenkins-ci.plugins/jackson2-api/
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/META-INF/maven/org.jenkins-ci.plugins/jackson2-api/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/META-INF/maven/org.jenkins-ci.plugins/jackson2-api/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/lib/jackson-core-2.9.9.jar  
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/lib/jackson2-api.jar  
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/lib/jackson-module-jaxb-annotations-2.9.9.jar  
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/lib/jackson-annotations-2.9.0.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/lib/jackson-datatype-jsr310-2.9.9.jar  
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/lib/jackson-module-parameter-names-2.9.9.jar  
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/lib/jackson-datatype-jdk8-2.9.9.jar  
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/lib/jackson-databind-2.9.9.1.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/jackson2-api/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/jackson2-api/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-build-step.jpi  
    default:    creating: /var/lib/jenkins/plugins/ace-editor/
    default:    creating: /var/lib/jenkins/plugins/ace-editor/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/ace-editor/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/ace-editor/META-INF/maven/org.jenkins-ci.ui/
    default:    creating: /var/lib/jenkins/plugins/ace-editor/META-INF/maven/org.jenkins-ci.ui/ace-editor/
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/META-INF/maven/org.jenkins-ci.ui/ace-editor/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/META-INF/maven/org.jenkins-ci.ui/ace-editor/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/ace-editor/jsmodules/
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/jsmodules/ace-editor-119.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/jsmodules/ace-editor-122.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/jsmodules/test.js  
    default:    creating: /var/lib/jenkins/plugins/ace-editor/packs/
    default:    creating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/
    default:    creating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/abap.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/abc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/actionscript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/ada.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/apache_conf.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/applescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/asciidoc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/assembly_x86.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/autohotkey.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/batchfile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/c9search.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/c_cpp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/cirru.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/clojure.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/cobol.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/coffee.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/coldfusion.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/csharp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/css.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/curly.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/d.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/dart.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/diff.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/django.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/dockerfile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/dot.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/eiffel.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/ejs.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/elixir.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/elm.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/erlang.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/forth.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/ftl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/gcode.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/gherkin.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/gitignore.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/glsl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/golang.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/groovy.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/haml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/handlebars.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/haskell.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/haxe.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/html.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/html_ruby.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/ini.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/io.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/jack.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/jade.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/java.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/javascript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/json.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/jsoniq.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/jsp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/jsx.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/julia.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/latex.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/lean.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/less.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/liquid.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/lisp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/live_script.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/livescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/logiql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/lsl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/lua.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/luapage.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/lucene.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/makefile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/markdown.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/mask.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/matlab.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/mel.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/mips_assembler.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/mipsassembler.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/mushcode.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/mysql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/nix.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/objectivec.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/ocaml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/pascal.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/perl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/pgsql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/php.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/plain_text.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/powershell.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/praat.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/prolog.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/properties.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/protobuf.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/python.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/r.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/rdoc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/rhtml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/ruby.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/rust.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/sass.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/scad.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/scala.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/scheme.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/scss.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/sh.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/sjs.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/smarty.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/snippets.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/soy_template.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/space.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/sql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/stylus.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/svg.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/tcl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/tex.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/text.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/textile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/toml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/twig.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/typescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/vala.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/vbscript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/velocity.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/verilog.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/vhdl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/xml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/xquery.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/snippets/yaml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ace.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-beautify.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-chromevox.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-elastic_tabstops_lite.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-emmet.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-error_marker.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-keybinding_menu.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-language_tools.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-linking.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-modelist.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-old_ie.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-searchbox.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-settings_menu.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-spellcheck.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-split.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-static_highlight.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-statusbar.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-textarea.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-themelist.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/ext-whitespace.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/keybinding-emacs.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/keybinding-vim.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-abap.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-abc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-actionscript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-ada.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-apache_conf.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-applescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-asciidoc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-assembly_x86.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-autohotkey.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-batchfile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-c9search.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-c_cpp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-cirru.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-clojure.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-cobol.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-coffee.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-coldfusion.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-csharp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-css.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-curly.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-d.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-dart.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-diff.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-django.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-dockerfile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-dot.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-eiffel.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-ejs.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-elixir.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-elm.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-erlang.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-forth.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-ftl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-gcode.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-gherkin.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-gitignore.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-glsl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-golang.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-groovy.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-haml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-handlebars.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-haskell.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-haxe.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-html.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-html_ruby.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-ini.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-io.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-jack.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-jade.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-java.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-javascript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-json.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-jsoniq.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-jsp.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-jsx.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-julia.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-latex.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-lean.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-less.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-liquid.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-lisp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-live_script.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-livescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-logiql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-lsl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-lua.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-luapage.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-lucene.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-makefile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-markdown.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-mask.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-matlab.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-mel.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-mips_assembler.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-mipsassembler.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-mushcode.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-mysql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-nix.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-objectivec.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-ocaml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-pascal.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-perl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-pgsql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-php.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-plain_text.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-powershell.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-praat.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-prolog.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-properties.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-protobuf.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-python.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-r.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-rdoc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-rhtml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-ruby.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-rust.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-sass.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-scad.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-scala.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-scheme.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-scss.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-sh.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-sjs.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-smarty.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-snippets.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-soy_template.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-space.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-sql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-stylus.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-svg.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-tcl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-tex.js  
    default:  extracting: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-text.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-textile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-toml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-twig.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-typescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-vala.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-vbscript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-velocity.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-verilog.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-vhdl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-xml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-xquery.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/mode-yaml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-ambiance.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-chaos.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-chrome.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-clouds.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-clouds_midnight.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-cobalt.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-crimson_editor.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-dawn.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-dreamweaver.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-eclipse.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-github.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-idle_fingers.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-katzenmilch.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-kr_theme.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-kuroir.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-merbivore.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-merbivore_soft.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-mono_industrial.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-monokai.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-pastel_on_dark.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-solarized_dark.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-solarized_light.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-terminal.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-textmate.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-tomorrow.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-tomorrow_night.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-tomorrow_night_blue.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-tomorrow_night_bright.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-tomorrow_night_eighties.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-twilight.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-vibrant_ink.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/theme-xcode.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/worker-coffee.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/worker-css.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/worker-html.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/worker-javascript.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/worker-json.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/worker-lua.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/worker-php.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/worker-xml.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-119/worker-xquery.js  
    default: 
    default:    creating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/
    default:    creating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/abap.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/abc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/actionscript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/ada.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/apache_conf.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/applescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/asciidoc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/assembly_x86.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/autohotkey.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/batchfile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/c9search.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/c_cpp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/cirru.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/clojure.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/cobol.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/coffee.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/coldfusion.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/csharp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/css.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/curly.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/d.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/dart.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/diff.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/django.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/dockerfile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/dot.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/eiffel.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/ejs.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/elixir.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/elm.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/erlang.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/forth.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/ftl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/gcode.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/gherkin.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/gitignore.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/glsl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/golang.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/groovy.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/haml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/handlebars.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/haskell.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/haxe.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/html.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/html_elixir.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/html_ruby.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/ini.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/io.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/jack.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/jade.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/java.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/javascript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/json.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/jsoniq.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/jsp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/jsx.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/julia.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/latex.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/lean.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/less.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/liquid.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/lisp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/live_script.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/livescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/logiql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/lsl.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/lua.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/luapage.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/lucene.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/makefile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/markdown.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/mask.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/matlab.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/maze.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/mel.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/mips_assembler.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/mipsassembler.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/mushcode.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/mysql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/nix.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/objectivec.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/ocaml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/pascal.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/perl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/pgsql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/php.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/plain_text.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/powershell.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/praat.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/prolog.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/properties.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/protobuf.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/python.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/r.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/rdoc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/rhtml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/ruby.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/rust.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/sass.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/scad.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/scala.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/scheme.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/scss.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/sh.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/sjs.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/smarty.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/snippets.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/soy_template.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/space.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/sql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/sqlserver.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/stylus.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/svg.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/swift.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/swig.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/tcl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/tex.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/text.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/textile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/toml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/twig.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/typescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/vala.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/vbscript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/velocity.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/verilog.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/vhdl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/xml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/xquery.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/snippets/yaml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ace.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-beautify.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-chromevox.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-elastic_tabstops_lite.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-emmet.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-error_marker.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-keybinding_menu.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-language_tools.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-linking.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-modelist.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-old_ie.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-searchbox.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-settings_menu.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-spellcheck.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-split.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-static_highlight.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-statusbar.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-textarea.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-themelist.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/ext-whitespace.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/keybinding-emacs.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/keybinding-vim.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-abap.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-abc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-actionscript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-ada.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-apache_conf.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-applescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-asciidoc.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-assembly_x86.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-autohotkey.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-batchfile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-c9search.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-c_cpp.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-cirru.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-clojure.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-cobol.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-coffee.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-coldfusion.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-csharp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-css.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-curly.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-d.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-dart.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-diff.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-django.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-dockerfile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-dot.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-eiffel.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-ejs.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-elixir.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-elm.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-erlang.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-forth.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-ftl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-gcode.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-gherkin.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-gitignore.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-glsl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-golang.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-groovy.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-haml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-handlebars.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-haskell.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-haxe.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-html.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-html_elixir.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-html_ruby.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-ini.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-io.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-jack.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-jade.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-java.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-javascript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-json.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-jsoniq.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-jsp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-jsx.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-julia.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-latex.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-lean.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-less.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-liquid.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-lisp.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-live_script.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-livescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-logiql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-lsl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-lua.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-luapage.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-lucene.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-makefile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-markdown.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-mask.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-matlab.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-mavens_mate_log.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-maze.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-mel.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-mips_assembler.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-mipsassembler.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-mushcode.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-mysql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-nix.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-objectivec.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-ocaml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-pascal.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-perl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-pgsql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-php.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-plain_text.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-powershell.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-praat.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-prolog.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-properties.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-protobuf.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-python.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-r.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-rdoc.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-rhtml.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-ruby.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-rust.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-sass.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-scad.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-scala.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-scheme.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-scss.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-sh.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-sjs.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-smarty.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-snippets.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-soy_template.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-space.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-sql.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-sqlserver.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-stylus.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-svg.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-swift.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-swig.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-tcl.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-tex.js  
    default:  extracting: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-text.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-textile.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-toml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-twig.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-typescript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-vala.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-vbscript.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-velocity.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-verilog.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-vhdl.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-xml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-xquery.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/mode-yaml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-ambiance.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-chaos.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-chrome.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-clouds.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-clouds_midnight.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-cobalt.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-crimson_editor.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-dawn.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-dreamweaver.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-eclipse.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-github.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-idle_fingers.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-iplastic.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-katzenmilch.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-kr_theme.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-kuroir.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-merbivore.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-merbivore_soft.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-mono_industrial.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-monokai.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-pastel_on_dark.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-solarized_dark.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-solarized_light.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-sqlserver.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-terminal.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-textmate.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-tomorrow.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-tomorrow_night.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-tomorrow_night_blue.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-tomorrow_night_bright.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-tomorrow_night_eighties.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-twilight.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-vibrant_ink.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/theme-xcode.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/worker-coffee.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/worker-css.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/worker-html.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/worker-javascript.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/worker-json.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/worker-lua.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/worker-php.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/worker-xml.js  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/ace-editor-122/worker-xquery.js  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/packs/README.md  
    default:    creating: /var/lib/jenkins/plugins/ace-editor/test-snippets/
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/test-snippets/groovy.js  
    default:    creating: /var/lib/jenkins/plugins/ace-editor/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/ace-editor/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/WEB-INF/lib/ace-editor.jar  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/WEB-INF/licenses.xml  
    default:   inflating: /var/lib/jenkins/plugins/ace-editor/test.html  
    default:  extracting: /var/lib/jenkins/plugins/ace-editor/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/plain-credentials.jpi  
    default:    creating: /var/lib/jenkins/plugins/workflow-cps/
    default:    creating: /var/lib/jenkins/plugins/workflow-cps/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-cps/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-cps/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-cps/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-cps/
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-cps/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-cps/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/workflow-cps/snippets/
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/snippets/workflow.js  
    default:    creating: /var/lib/jenkins/plugins/workflow-cps/images/
    default:    creating: /var/lib/jenkins/plugins/workflow-cps/images/24x24/
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/images/24x24/README.md  
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/images/24x24/pause.png  
    default:    creating: /var/lib/jenkins/plugins/workflow-cps/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-cps/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/WEB-INF/lib/groovy-cps-1.30.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/WEB-INF/lib/jsr305-3.0.2.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/WEB-INF/lib/diff4j-1.2.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/WEB-INF/lib/workflow-cps.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/workflow-cps/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/credentials-binding.jpi  
    default:    creating: /var/lib/jenkins/plugins/pipeline-input-step/
    default:    creating: /var/lib/jenkins/plugins/pipeline-input-step/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-input-step/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-input-step/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-input-step/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-input-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-input-step/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-input-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-input-step/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-input-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-input-step/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-input-step/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-input-step/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-input-step/WEB-INF/lib/pipeline-input-step.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-input-step/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-input-step/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-api.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-step/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-step/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-step/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-step/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-step/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-stage-step/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-stage-step/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-step/META-INF/maven/org.jenkins-ci.plugins/pipeline-stage-step/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-step/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-step/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-step/WEB-INF/lib/pipeline-stage-step.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-step/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-stage-step/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/cloudbees-folder.jpi  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-multibranch.jpi  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata.jpi  
    default:   inflating: /var/lib/jenkins/plugins/authentication-tokens.jpi  
    default:    creating: /var/lib/jenkins/plugins/workflow-cps-global-lib/
    default:    creating: /var/lib/jenkins/plugins/workflow-cps-global-lib/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-cps-global-lib/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-cps-global-lib/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-cps-global-lib/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-cps-global-lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-cps-global-lib/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-cps-global-lib/pom.properties  
    default: 
    default:    creating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/httpasyncclient-cache-4.1.3.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/workflow-cps-global-lib.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/httpcore-4.4.9.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/httpclient-4.5.5.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/fluent-hc-4.5.5.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/httpclient-cache-4.5.5.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/httpasyncclient-4.1.3.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/httpmime-4.5.5.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/ivy-2.4.0.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/commons-lang3-3.7.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/lib/httpcore-nio-4.4.6.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-cps-global-lib/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/workflow-cps-global-lib/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/docker-commons.jpi  
    default:   inflating: /var/lib/jenkins/plugins/durable-task.jpi  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent.jpi  
    default:   inflating: /var/lib/jenkins/plugins/workflow-durable-task-step.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/pipeline-graph-analysis/
    default:    creating: /var/lib/jenkins/plugins/pipeline-graph-analysis/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-graph-analysis/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-graph-analysis/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-graph-analysis/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-graph-analysis/META-INF/maven/org.jenkins-ci.plugins/pipeline-graph-analysis/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-graph-analysis/META-INF/maven/org.jenkins-ci.plugins/pipeline-graph-analysis/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-graph-analysis/META-INF/maven/org.jenkins-ci.plugins/pipeline-graph-analysis/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-graph-analysis/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-graph-analysis/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-graph-analysis/WEB-INF/lib/pipeline-graph-analysis.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-graph-analysis/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-graph-analysis/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/workflow-basic-steps.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/handlebars/
    default:    creating: /var/lib/jenkins/plugins/handlebars/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/handlebars/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/handlebars/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/handlebars/META-INF/maven/org.jenkins-ci.ui/
    default:    creating: /var/lib/jenkins/plugins/handlebars/META-INF/maven/org.jenkins-ci.ui/handlebars/
    default:   inflating: /var/lib/jenkins/plugins/handlebars/META-INF/maven/org.jenkins-ci.ui/handlebars/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/handlebars/META-INF/maven/org.jenkins-ci.ui/handlebars/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/handlebars/jsmodules/
    default:   inflating: /var/lib/jenkins/plugins/handlebars/jsmodules/handlebars3.js  
    default: 
    default:    creating: /var/lib/jenkins/plugins/handlebars/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/handlebars/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/handlebars/WEB-INF/lib/handlebars.jar  
    default:   inflating: /var/lib/jenkins/plugins/handlebars/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/handlebars/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/docker-workflow.jpi  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-definition.jpi  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults.jpi  
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/META-INF/maven/org.jenkinsci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/META-INF/maven/org.jenkinsci.plugins/pipeline-stage-tags-metadata/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/META-INF/maven/org.jenkinsci.plugins/pipeline-stage-tags-metadata/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/META-INF/maven/org.jenkinsci.plugins/pipeline-stage-tags-metadata/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/WEB-INF/lib/pipeline-stage-tags-metadata.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-stage-tags-metadata/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/authentication-tokens/
    default:    creating: /var/lib/jenkins/plugins/authentication-tokens/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/authentication-tokens/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/authentication-tokens/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/authentication-tokens/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/authentication-tokens/META-INF/maven/org.jenkins-ci.plugins/authentication-tokens/
    default:   inflating: /var/lib/jenkins/plugins/authentication-tokens/META-INF/maven/org.jenkins-ci.plugins/authentication-tokens/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/authentication-tokens/META-INF/maven/org.jenkins-ci.plugins/authentication-tokens/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/authentication-tokens/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/authentication-tokens/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/authentication-tokens/WEB-INF/lib/authentication-tokens.jar  
    default:   inflating: /var/lib/jenkins/plugins/authentication-tokens/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/authentication-tokens/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/META-INF/maven/org.jenkinsci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/META-INF/maven/org.jenkinsci.plugins/pipeline-model-declarative-agent/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/META-INF/maven/org.jenkinsci.plugins/pipeline-model-declarative-agent/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/META-INF/maven/org.jenkinsci.plugins/pipeline-model-declarative-agent/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/WEB-INF/lib/pipeline-model-declarative-agent.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-model-declarative-agent/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/workflow-durable-task-step/
    default:    creating: /var/lib/jenkins/plugins/workflow-durable-task-step/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-durable-task-step/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-durable-task-step/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-durable-task-step/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-durable-task-step/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-durable-task-step/
    default:   inflating: /var/lib/jenkins/plugins/workflow-durable-task-step/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-durable-task-step/pom.xml  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-durable-task-step/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-durable-task-step/pom.properties  
    default: 
    default:    creating: /var/lib/jenkins/plugins/workflow-durable-task-step/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-durable-task-step/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-durable-task-step/WEB-INF/lib/workflow-durable-task-step.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/workflow-durable-task-step/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/workflow-durable-task-step/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-definition/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-definition/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-definition/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-definition/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-definition/META-INF/maven/org.jenkinsci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-definition/META-INF/maven/org.jenkinsci.plugins/pipeline-model-definition/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-definition/META-INF/maven/org.jenkinsci.plugins/pipeline-model-definition/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-definition/META-INF/maven/org.jenkinsci.plugins/pipeline-model-definition/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-definition/images/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-definition/images/24x24/
    default:  extracting: /var/lib/jenkins/plugins/pipeline-model-definition/images/24x24/restart-stage.png  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-definition/images/48x48/
    default:  extracting: /var/lib/jenkins/plugins/pipeline-model-definition/images/48x48/restart-stage.png  
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-definition/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-model-definition/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-definition/WEB-INF/lib/annotation-indexer-1.12.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-definition/WEB-INF/lib/pipeline-model-definition.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-definition/WEB-INF/lib/joda-time-2.9.5.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/pipeline-model-definition/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-model-definition/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/
    default:    creating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/META-INF/maven/org.jenkins-ci.plugins/pipeline-multibranch-defaults/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/META-INF/maven/org.jenkins-ci.plugins/pipeline-multibranch-defaults/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/META-INF/maven/org.jenkins-ci.plugins/pipeline-multibranch-defaults/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/images/
    default:    creating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/images/48x48/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/images/48x48/pipelinemultibranchdefaultsproject.png  
    default:    creating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/WEB-INF/lib/pipeline-multibranch-defaults.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/WEB-INF/lib/annotation-indexer-1.11.jar  
    default:   inflating: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/pipeline-multibranch-defaults/.timestamp2  
    default:    creating: /var/lib/jenkins/plugins/workflow-job/
    default:    creating: /var/lib/jenkins/plugins/workflow-job/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/workflow-job/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/workflow-job/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/workflow-job/META-INF/maven/org.jenkins-ci.plugins.workflow/
    default:    creating: /var/lib/jenkins/plugins/workflow-job/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-job/
    default:   inflating: /var/lib/jenkins/plugins/workflow-job/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-job/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/workflow-job/META-INF/maven/org.jenkins-ci.plugins.workflow/workflow-job/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/workflow-job/images/
    default:    creating: /var/lib/jenkins/plugins/workflow-job/images/48x48/
    default:   inflating: /var/lib/jenkins/plugins/workflow-job/images/48x48/pipelinejob.png  
    default:    creating: /var/lib/jenkins/plugins/workflow-job/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/workflow-job/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/workflow-job/WEB-INF/lib/workflow-job.jar  
    default:   inflating: /var/lib/jenkins/plugins/workflow-job/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/workflow-job/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook.jpi  
    default: 
    default:    creating: /var/lib/jenkins/plugins/gogs-webhook/
    default:    creating: /var/lib/jenkins/plugins/gogs-webhook/META-INF/
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/META-INF/MANIFEST.MF  
    default:    creating: /var/lib/jenkins/plugins/gogs-webhook/META-INF/maven/
    default:    creating: /var/lib/jenkins/plugins/gogs-webhook/META-INF/maven/org.jenkins-ci.plugins/
    default:    creating: /var/lib/jenkins/plugins/gogs-webhook/META-INF/maven/org.jenkins-ci.plugins/gogs-webhook/
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/META-INF/maven/org.jenkins-ci.plugins/gogs-webhook/pom.xml  
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/META-INF/maven/org.jenkins-ci.plugins/gogs-webhook/pom.properties  
    default:    creating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/
    default:    creating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/jsr305-1.3.9.jar  
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/gogs-webhook.jar  
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/error_prone_annotations-2.1.3.jar  
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/httpclient-4.5.3.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/checker-compat-qual-2.0.0.jar  
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/commons-codec-1.10.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/j2objc-annotations-1.1.jar  
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/guava-25.0-jre.jar  
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/httpcore-4.4.6.jar  
    default: 
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/lib/findbugs-annotations-1.3.9-1.jar  
    default:   inflating: /var/lib/jenkins/plugins/gogs-webhook/WEB-INF/licenses.xml  
    default:  extracting: /var/lib/jenkins/plugins/gogs-webhook/.timestamp2  
    default:   inflating: /var/lib/jenkins/plugins/workflow-job.bak  
    default:    creating: /var/lib/jenkins/jobs/
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/1/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/1/fileParameters/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/1/fileParameters/assets_path  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/2/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/2/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/2/fileParameters/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/2/fileParameters/assets_path  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/3/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/3/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/3/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/3/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/4/changelog.xml  
    default: 
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/4/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/4/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/5/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/5/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/5/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/5/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/5/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/5/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/6/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/6/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/6/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/6/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/6/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/6/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/7/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/7/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/7/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/7/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/7/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/7/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/8/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/8/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/8/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/8/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/8/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/8/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/9/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/9/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/9/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/9/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/9/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/9/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/10/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/10/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/10/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/10/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/10/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/10/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/11/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/11/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/11/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/11/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/11/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/11/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/12/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/12/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/12/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/12/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/12/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/12/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/13/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/13/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/13/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/13/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/13/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/13/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/14/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/14/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/14/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/14/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/14/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/14/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastFailedBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastFailedBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastFailedBuild/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastFailedBuild/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastFailedBuild/fileParameters/assets.zip  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastFailedBuild/build.xml  
    default: 
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastUnsuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastUnsuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastUnsuccessfulBuild/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastUnsuccessfulBuild/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastUnsuccessfulBuild/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastUnsuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/15/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/15/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/15/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/15/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/15/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/15/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/16/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/16/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/16/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/16/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/16/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/16/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/17/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/17/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/17/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/17/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/17/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/17/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/18/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/18/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/18/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/18/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/18/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/18/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/19/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/19/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/19/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/19/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/19/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/19/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/20/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/20/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/20/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/20/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/20/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/20/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/21/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/21/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/21/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/21/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/21/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/21/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/22/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/22/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/22/changelog.xml  
    default: 
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/22/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/22/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/22/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastStableBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastStableBuild/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastStableBuild/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastStableBuild/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastSuccessfulBuild/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastSuccessfulBuild/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastSuccessfulBuild/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/builds/lastSuccessfulBuild/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_monitor_v1/
    default:    creating: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/1/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/lastStableBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/lastStableBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/lastSuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1/builds/lastSuccessfulBuild/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_monitor_v1/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1/
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/1/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/2/log  
    default: 
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/2/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/3/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/3/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/4/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/5/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/5/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/5/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/5/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/lastStableBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/lastStableBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/lastSuccessfulBuild/changelog.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/builds/lastSuccessfulBuild/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/1/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/1/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/1/fileParameters/dynamic.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/2/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/2/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/2/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/2/fileParameters/dynamic.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/3/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/3/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/3/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/3/fileParameters/dynamic.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/4/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/4/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/4/fileParameters/dynamic.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/5/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/5/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/5/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/5/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/5/fileParameters/dynamic.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/5/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/6/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/6/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/6/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/6/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/6/fileParameters/dynamic.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/6/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastStableBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastStableBuild/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastStableBuild/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastStableBuild/fileParameters/dynamic.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastSuccessfulBuild/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastSuccessfulBuild/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastSuccessfulBuild/fileParameters/dynamic.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/builds/lastSuccessfulBuild/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/1/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/2/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/2/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/3/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/3/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/4/changelog.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/lastStableBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/lastStableBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/lastSuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/builds/lastSuccessfulBuild/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/config.xml  
    default: 
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nginx_env_v1/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/1/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/2/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/2/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/3/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/3/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/4/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/5/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/5/log  
    default: 
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/5/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/5/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/lastStableBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/lastStableBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/lastSuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/builds/lastSuccessfulBuild/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_nodejs_env_v1/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/1/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/1/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/1/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/2/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/2/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/3/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/3/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/4/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/5/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/5/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/5/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/5/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/6/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/6/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/6/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/6/build.xml  
    default: 
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/7/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/7/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/7/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/7/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/8/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/8/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/8/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/8/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/8/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/8/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/9/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/9/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/9/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/9/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/10/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/10/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/10/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/10/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/11/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/11/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/11/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/11/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/12/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/12/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/12/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/12/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/13/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/13/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/13/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/13/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastStableBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastStableBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastSuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastSuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastFailedBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastFailedBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastFailedBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastFailedBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastUnsuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastUnsuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastUnsuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/lastUnsuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/18/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/18/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/18/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/18/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/19/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/19/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/19/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/19/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/20/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/20/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/20/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/20/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/21/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/21/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/21/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/builds/21/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/lastSuccessful/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/lastSuccessful/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/lastSuccessful/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/lastSuccessful/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/lastStable/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/lastStable/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/lastStable/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_test_v1/lastStable/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/1/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/2/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/2/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/3/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/3/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/4/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/4/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/4/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/5/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/5/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/5/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/5/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/6/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/6/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/6/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/6/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/7/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/7/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/7/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/7/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastStableBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastStableBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastSuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastSuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastFailedBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastFailedBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastFailedBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastFailedBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastUnsuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastUnsuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastUnsuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/lastUnsuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/9/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/9/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/9/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/9/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/10/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/10/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/10/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/10/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/11/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/11/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/11/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/11/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/12/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/12/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/12/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/builds/12/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/lastSuccessful/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/lastSuccessful/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/lastSuccessful/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/lastSuccessful/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/lastStable/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/lastStable/log  
    default: 
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/lastStable/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_stag_v1/lastStable/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/1/changelog.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/1/fileParameters/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/1/fileParameters/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/2/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/2/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/3/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/3/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/4/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/5/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/5/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/5/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/5/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/lastStableBuild/log  
    default: 
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/lastStableBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/lastSuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/builds/lastSuccessfulBuild/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/lastSuccessful/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/lastSuccessful/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/lastSuccessful/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/lastSuccessful/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/lastStable/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/lastStable/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/lastStable/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_build_prod_v1/lastStable/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/1/workflow/26.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/21.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/2/workflow/23.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/15.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/3/workflow/31.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/4/workflow/31.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastFailedBuild/workflow/31.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/5.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/28.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/29.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastUnsuccessfulBuild/workflow/31.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/31.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/32.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/51.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/workflow/55.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/5/assets.zip  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/workflow/52.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/6/assets.zip  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/log  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/8.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/workflow/21.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/7/assets.zip  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/workflow/21.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/8/assets.zip  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/workflow/52.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/9/assets.zip  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/workflow/52.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/10/assets.zip  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/45.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/workflow/52.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastStableBuild/assets.zip  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/33.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/workflow/52.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/builds/lastSuccessfulBuild/assets.zip  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/workflow/52.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastSuccessful/assets.zip  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/workflow/52.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v2_165_pipeline/lastStable/assets.zip  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/1/workflow/12.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastFailedBuild/workflow/12.xml  
    default: 
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastUnsuccessfulBuild/workflow/12.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/57.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/59.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/61.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/65.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/23.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/workflow/67.xml  
    default: 
    default:  extracting: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/2/changelog5477182396505605674.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/57.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/59.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/61.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/65.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/33.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/workflow/67.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastStableBuild/changelog5477182396505605674.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/log  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/57.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/59.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/61.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/65.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/67.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/builds/lastSuccessfulBuild/changelog5477182396505605674.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/57.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/59.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/61.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/65.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/28.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/workflow/67.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastSuccessful/changelog5477182396505605674.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/57.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/59.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/61.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/65.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/workflow/67.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_asset_v3_pipeline_as_code/lastStable/changelog5477182396505605674.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/57.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/59.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/5.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/61.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/65.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/workflow/67.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/1/changelog7753363365895566412.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/57.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/59.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/61.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/65.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/workflow/67.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/2/changelog625589012362112109.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/57.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/59.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/61.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/65.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/workflow/67.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastStableBuild/changelog625589012362112109.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/57.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/59.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/61.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/65.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/40.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/workflow/67.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/builds/lastSuccessfulBuild/changelog625589012362112109.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/57.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/59.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/61.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/65.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/22.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/workflow/67.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastSuccessful/changelog625589012362112109.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/56.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/57.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/58.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/59.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/60.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/62.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/61.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/63.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/64.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/65.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/38.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/39.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/40.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/41.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/42.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/43.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/44.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/45.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/46.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/47.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/48.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/49.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/50.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/51.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/52.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/53.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/54.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/55.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/66.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/workflow/67.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v3_pipeline_as_code/lastStable/changelog625589012362112109.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1_165/
    default:    creating: /var/lib/jenkins/jobs/cicd_assets_v1_165/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1_165/builds/legacyIds  
    default:   inflating: /var/lib/jenkins/jobs/cicd_assets_v1_165/config.xml  
    default: 
    default:  extracting: /var/lib/jenkins/jobs/cicd_assets_v1_165/nextBuildNumber  
    default: 
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1_165/
    default:    creating: /var/lib/jenkins/jobs/cicd_dynamic_v1_165/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1_165/builds/legacyIds  
    default:   inflating: /var/lib/jenkins/jobs/cicd_dynamic_v1_165/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_dynamic_v1_165/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1_165/
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v1_165/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1_165/builds/legacyIds  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v1_165/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v1_165/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/1/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/2/workflow/27.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/8.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/3/workflow/30.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/4/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/5/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/5/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/5/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/5/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/6/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/6/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/6/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/6/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/7/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/7/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/7/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/7/build.xml  
    default: 
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/8/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/8/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/8/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/8/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/9/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/9/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/9/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/9/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/log-index  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/10/workflow/29.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/18.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/28.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/11/workflow/31.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/4.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/5.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/12/workflow/31.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/13/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/13/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/13/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/13/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastFailedBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastFailedBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastFailedBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastFailedBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/16.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/17.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/25.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/14/workflow/29.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/log  
    default: 
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/21.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastUnsuccessfulBuild/workflow/29.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/15/workflow/37.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/16/workflow/37.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/log  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/log-index  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/7.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/11.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastStableBuild/workflow/37.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/log  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/log-index  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/2.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/3.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/4.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/5.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/6.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/7.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/8.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/9.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/10.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/11.xml  
    default: 
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/12.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/13.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/14.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/15.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/16.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/17.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/18.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/19.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/20.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/21.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/22.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/23.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/24.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/25.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/26.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/27.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/28.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/29.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/30.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/31.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/32.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/33.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/34.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/35.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/36.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/builds/lastSuccessfulBuild/workflow/37.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_init_env_v2/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_init_env_v2/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/
    default:    creating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/1/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/lastStableBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/lastStableBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/lastSuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/builds/lastSuccessfulBuild/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_monitor_v1_165/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_monitor_v1_165/nextBuildNumber  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/legacyIds  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/1/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/1/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/1/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/1/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/2/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/2/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/2/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/2/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/3/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/3/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/3/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/3/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/4/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/4/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/4/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/4/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/5/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/5/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/5/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/5/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastFailedBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastFailedBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastFailedBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastFailedBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastUnsuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastUnsuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastUnsuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastUnsuccessfulBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/6/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/6/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/6/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/6/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/7/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/7/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/7/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/7/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastStableBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastStableBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastStableBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastStableBuild/build.xml  
    default:    creating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastSuccessfulBuild/
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastSuccessfulBuild/log  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastSuccessfulBuild/changelog.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/builds/lastSuccessfulBuild/build.xml  
    default:   inflating: /var/lib/jenkins/jobs/cicd_scale_v1/config.xml  
    default:  extracting: /var/lib/jenkins/jobs/cicd_scale_v1/nextBuildNumber  
    default:    creating: /var/lib/jenkins/users/
    default:   inflating: /var/lib/jenkins/users/users.xml  
    default:    creating: /var/lib/jenkins/users/admin_2729160671010981902/
    default:   inflating: /var/lib/jenkins/users/admin_2729160671010981902/config.xml  
    default:    creating: /var/lib/jenkins/users/yyghdfz_6671524892573031109/
    default:   inflating: /var/lib/jenkins/users/yyghdfz_6671524892573031109/config.xml  
    default:   inflating: /var/lib/jenkins/hudson.model.UpdateCenter.xml  
    default:    creating: /var/lib/jenkins/nodes/
    default:  extracting: /var/lib/jenkins/identity.key.enc  
    default:    creating: /var/lib/jenkins/secrets/
    default:   inflating: /var/lib/jenkins/secrets/master.key  
    default:  extracting: /var/lib/jenkins/secrets/org.jenkinsci.main.modules.instance_identity.InstanceIdentity.KEY  
    default:  extracting: /var/lib/jenkins/secrets/jenkins.model.Jenkins.crumbSalt  
    default:    creating: /var/lib/jenkins/secrets/whitelisted-callables.d/
    default:   inflating: /var/lib/jenkins/secrets/whitelisted-callables.d/default.conf  
    default:    creating: /var/lib/jenkins/secrets/filepath-filters.d/
    default:   inflating: /var/lib/jenkins/secrets/filepath-filters.d/30-default.conf  
    default:  extracting: /var/lib/jenkins/secrets/slave-to-master-security-kill-switch  
    default:  extracting: /var/lib/jenkins/secrets/hudson.console.ConsoleNote.MAC  
    default:  extracting: /var/lib/jenkins/secrets/hudson.model.Job.serverCookie  
    default:  extracting: /var/lib/jenkins/secrets/hudson.console.AnnotatedLargeText.consoleAnnotator  
    default:  extracting: /var/lib/jenkins/secrets/org.jenkinsci.plugins.workflow.log.ConsoleAnnotators.consoleAnnotator  
    default:  extracting: /var/lib/jenkins/secrets/hudson.util.Secret  
    default:   inflating: /var/lib/jenkins/jenkins.telemetry.Correlator.xml  
    default:   inflating: /var/lib/jenkins/nodeMonitors.xml  
    default:  extracting: /var/lib/jenkins/jenkins.install.UpgradeWizard.state  
    default:    creating: /var/lib/jenkins/userContent/
    default:   inflating: /var/lib/jenkins/userContent/readme.txt  
    default:    creating: /var/lib/jenkins/logs/
    default:    creating: /var/lib/jenkins/logs/tasks/
    default:   inflating: /var/lib/jenkins/logs/tasks/Fingerprint cleanup.log  
    default:   inflating: /var/lib/jenkins/logs/tasks/telemetry collection.log.1  
    default:   inflating: /var/lib/jenkins/logs/tasks/telemetry collection.log  
    default:   inflating: /var/lib/jenkins/logs/tasks/Download metadata.log.5  
    default:   inflating: /var/lib/jenkins/logs/tasks/Download metadata.log.4  
    default:   inflating: /var/lib/jenkins/logs/tasks/Download metadata.log.3  
    default:   inflating: /var/lib/jenkins/logs/tasks/Download metadata.log.2  
    default:   inflating: /var/lib/jenkins/logs/tasks/Download metadata.log.1  
    default:   inflating: /var/lib/jenkins/logs/tasks/Download metadata.log  
    default:   inflating: /var/lib/jenkins/logs/tasks/Workspace clean-up.log  
    default:    creating: /var/lib/jenkins/updates/
    default:   inflating: /var/lib/jenkins/updates/default.json  
    default: 
    default:   inflating: /var/lib/jenkins/updates/hudson.tasks.Maven.MavenInstaller  
    default:   inflating: /var/lib/jenkins/updates/hudson.tools.JDKInstaller  
    default: 
    default:  extracting: /var/lib/jenkins/.lastStarted  
    default:   inflating: /var/lib/jenkins/jenkins.model.JenkinsLocationConfiguration.xml  
    default:  extracting: /var/lib/jenkins/jenkins.install.InstallUtil.lastExecVersion  
    default:  extracting: /var/lib/jenkins/.owner  
    default:  extracting: /var/lib/jenkins/org.jenkinsci.plugins.workflow.support.steps.StageStep.xml  
    default:   inflating: /var/lib/jenkins/com.dabsquared.gitlabjenkins.connection.GitLabConnectionConfig.xml  
    default:   inflating: /var/lib/jenkins/hudson.plugins.git.GitTool.xml  
    default:   inflating: /var/lib/jenkins/com.dabsquared.gitlabjenkins.GitLabPushTrigger.xml  
    default:   inflating: /var/lib/jenkins/scriptApproval.xml  
    default:   inflating: /var/lib/jenkins/config.xml  
    default:    creating: /var/lib/jenkins/workspace/
    default:    creating: /var/lib/jenkins/workspace/cicd_v1/
    default:    creating: /var/lib/jenkins/workspace/cicd_v1/assets/
    default:    creating: /var/lib/jenkins/workspace/cicd_v1/assets/js/
    default:  extracting: /var/lib/jenkins/workspace/cicd_v1/assets/js/js_test.js  
    default:    creating: /var/lib/jenkins/workspace/cicd_v1/assets/img/
    default:  extracting: /var/lib/jenkins/workspace/cicd_v1/assets/img/img_test.img  
    default:    creating: /var/lib/jenkins/workspace/cicd_v1/assets/css/
    default:  extracting: /var/lib/jenkins/workspace/cicd_v1/assets/css/css_test.css  
    default:    creating: /var/lib/jenkins/workspace/cicd_v1/src/
    default:    creating: /var/lib/jenkins/workspace/cicd_v1/src/assets/
    default:    creating: /var/lib/jenkins/workspace/cicd_v1/src/assets/css/
    default:  extracting: /var/lib/jenkins/workspace/cicd_v1/src/assets/css/css_test.css  
    default:  extracting: /var/lib/jenkins/workspace/cicd_v1/src/assets/css/css_test2.css  
    default:    creating: /var/lib/jenkins/workspace/cicd_v1/src/assets/img/
    default:  extracting: /var/lib/jenkins/workspace/cicd_v1/src/assets/img/img_test.img  
    default:    creating: /var/lib/jenkins/workspace/cicd_v1/src/assets/js/
    default:  extracting: /var/lib/jenkins/workspace/cicd_v1/src/assets/js/js_test.js  
    default:  extracting: /var/lib/jenkins/workspace/cicd_v1/src/env.txt  
    default:   inflating: /var/lib/jenkins/workspace/cicd_v1/Dockerfile  
    default:    creating: /var/lib/jenkins/workspace/cicd_ dynamic_v1/
    default:  extracting: /var/lib/jenkins/workspace/cicd_ dynamic_v1/dynamic.zip  
    default:    creating: /var/lib/jenkins/workspace/cicd_ dynamic_v1/src/
    default:   inflating: /var/lib/jenkins/workspace/cicd_ dynamic_v1/src/server.js  
    default:   inflating: /var/lib/jenkins/workspace/cicd_ dynamic_v1/Dockerfile  
    default:    creating: /var/lib/jenkins/workspace/cicd_monitor_v1/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v1/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_v1/assets.zip  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v1/src/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v1/src/assets/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v1/src/assets/css/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_v1/src/assets/css/css_test.css  
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_v1/src/assets/css/css_test2.css  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v1/src/assets/img/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_v1/src/assets/img/img_test.img  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v1/src/assets/js/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_v1/src/assets/js/js_test.js  
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_v1/src/env.txt  
    default:   inflating: /var/lib/jenkins/workspace/cicd_assets_v1/Dockerfile  
    default:    creating: /var/lib/jenkins/workspace/cicd_init_env_v1/
    default:   inflating: /var/lib/jenkins/workspace/cicd_init_env_v1/server.js  
    default:   inflating: /var/lib/jenkins/workspace/cicd_init_env_v1/Dockerfile  
    default:  extracting: /var/lib/jenkins/workspace/cicd_init_env_v1/env.txt  
    default:    creating: /var/lib/jenkins/workspace/cicd_ scale_v1/
    default:    creating: /var/lib/jenkins/workspace/cicd_init_env_v2/
    default:    creating: /var/lib/jenkins/workspace/cicd_init_env_v2@tmp/
    default:    creating: /var/lib/jenkins/workspace/cicd_init_nodejs_env_v1/
    default:   inflating: /var/lib/jenkins/workspace/cicd_init_nodejs_env_v1/server.js  
    default:   inflating: /var/lib/jenkins/workspace/cicd_init_nodejs_env_v1/Dockerfile  
    default:    creating: /var/lib/jenkins/workspace/cicd_init_nginx_env_v1/
    default:  extracting: /var/lib/jenkins/workspace/cicd_init_nginx_env_v1/env.txt  
    default:   inflating: /var/lib/jenkins/workspace/cicd_init_nginx_env_v1/Dockerfile  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v2@tmp/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_v1/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_v1/assets.zip  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_v1/src/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_v1/src/assets/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_v1/src/assets/css/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_v1/src/assets/css/css_test.css  
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_v1/src/assets/css/css_test2.css  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_v1/src/assets/img/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_v1/src/assets/img/img_test.img  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_v1/src/assets/js/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_v1/src/assets/js/js_test.js  
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_v1/src/env.txt  
    default:   inflating: /var/lib/jenkins/workspace/cicd_assets_build_v1/Dockerfile  
    default:    creating: /var/lib/jenkins/workspace/abc/
    default:    creating: /var/lib/jenkins/workspace/cicd_monitor_v1_165(查看系统监控)/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/assets.zip  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/src/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/src/assets/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/src/assets/css/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/src/assets/css/css_test.css  
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/src/assets/css/css_test2.css  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/src/assets/img/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/src/assets/img/img_test.img  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/src/assets/js/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/src/assets/js/js_test.js  
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/src/env.txt  
    default:   inflating: /var/lib/jenkins/workspace/cicd_assets_build_test_v1/Dockerfile  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/assets.zip  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/src/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/src/assets/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/src/assets/css/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/src/assets/css/css_test.css  
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/src/assets/css/css_test2.css  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/src/assets/img/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/src/assets/img/img_test.img  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/src/assets/js/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/src/assets/js/js_test.js  
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/src/env.txt  
    default:   inflating: /var/lib/jenkins/workspace/cicd_assets_build_stag_v1/Dockerfile  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v3/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v3@tmp/
    default:    creating: /var/lib/jenkins/workspace/abc@tmp/
    default:    creating: /var/lib/jenkins/workspace/cicd_scale_v1(集群资源扩容)/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/assets.zip  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/src/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/src/assets/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/src/assets/css/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/src/assets/css/css_test.css  
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/src/assets/css/css_test2.css  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/src/assets/img/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/src/assets/img/img_test.img  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/src/assets/js/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/src/assets/js/js_test.js  
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/src/env.txt  
    default:   inflating: /var/lib/jenkins/workspace/cicd_assets_build_prod_v1/Dockerfile  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v2_165(静态资源部署流水线版)@tmp/
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v2_165(静态资源部署流水线版)/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_v2_165(静态资源部署流水线版)/assets.zip  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v2/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_v2/assets.zip  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v2_165_pipeline/
    default:  extracting: /var/lib/jenkins/workspace/cicd_assets_v2_165_pipeline/assets.zip  
    default:    creating: /var/lib/jenkins/workspace/cicd_assets_v2_165_pipeline@tmp/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/refs/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/refs/heads/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/refs/heads/master  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/refs/tags/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/refs/remotes/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/refs/remotes/origin/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/refs/remotes/origin/master  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/branches/
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/description  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/hooks/
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/hooks/applypatch-msg.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/hooks/commit-msg.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/hooks/post-update.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/hooks/pre-applypatch.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/hooks/pre-commit.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/hooks/pre-push.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/hooks/pre-rebase.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/hooks/prepare-commit-msg.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/hooks/update.sample  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/info/
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/info/exclude  
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/HEAD  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/config  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/pack/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/info/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/4a/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/4a/3867048d2409c447fdede33efb8e5f50ea8a20  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/9e/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/9e/d170be79d436ebd67d2edfa8d2b047cf256920  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/7f/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/7f/f40a2d88161b99efd90e79e2caf5a2757bbd09  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/2f/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/2f/34f5a52760c27e6178ea424e3ae1e27e0445ad  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/16/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/16/ff985207c5051fba0d69c7fb6f3f1f5cc8a40c  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/b5/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/b5/ffa97c0d9dd637ddb5b5c90feff746208b908a  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/69/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/69/6e516106c63e24a3a09ce9e1969291401392b6  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/e6/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/35/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/35/51367b5cfefe1508042679cdec728d0ab5cc11  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/95/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/95/c3903ef21da3c180330812d445dfcfd24a929e  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/62/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/62/560add7af41d82c638d67c5d5e641b1e0e6553  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/31/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/31/41fa7691e733278cdfc16f49ebc18eddfd756a  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/45/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/45/e490df1c068d16eb55c06c443e19788f4a0151  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/9d/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/9d/aeafb9864cf43055ae93beb0afd6c7d144bfa4  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/7a/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/7a/92d1730677dc911a9a98b465c80568efcef446  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/d1/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/d1/3a5448f2f4f56e5bb0708fb5e2f1c417e08831  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/4d/
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/4d/7f0d8a5c679da4dc815a7d91af57fc39849683  
    default: 
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/8b/
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/8b/f2a1de43e0bbe10f812edbc999091fff08a849  
    default: 
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/39/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/39/c96b920a3ec094f5143837fe8046b1a5b92896  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/d2/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/d2/1de79d1d8ca1b5a9417b80abbda14901753ce1  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/a0/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/a0/e11093d182807ae80c18ca52e8634b35eeff4f  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/7b/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/7b/ae1d38a02b028a19aab91a63a57f63cd70abd0  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/92/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/92/956413b4f750492cf572ed575a523f5cbc20cd  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/49/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/49/2986ff316380b1986d7772b664410cc3065c42  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/bf/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/bf/dc113cef87a41574f8d7ca8074234f31dc6d5f  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/a1/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/a1/7039c3e49e7a03d091eccc73ecb1fe5192498e  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/12/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/12/fb2da73ab1b3e401d7b944b3732d6eb026bd32  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/60/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/60/c56ca3f3daa0d19d5c359c24f6922e5091f45a  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/f9/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/objects/f9/3e3a1a1525fb5b91020da86e44810c87a2d7bc  
    default: 
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/FETCH_HEAD  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/logs/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/logs/refs/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/logs/refs/remotes/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/logs/refs/remotes/origin/
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/logs/refs/remotes/origin/master  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/logs/refs/heads/
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/logs/refs/heads/master  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/logs/HEAD  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/.git/index  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/Dockerfile  
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/Jenkinsfile  
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/README.md  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/css/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/css/abc.css  
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/css/css_test.css  
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/css/css_test2.css  
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/css/css_test3.css  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/img/
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/img/cicd/
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/img/cicd/cicd01.png  
    default: 
    default:   inflating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/img/cicd/cicd12.png  
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/img/img_test.img  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/js/
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/js/js_test.js  
    default:  extracting: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code/assets/js/x.js  
    default:    creating: /var/lib/jenkins/workspace/cicd_asset_v3_pipeline_as_code@tmp/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/refs/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/refs/heads/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/refs/heads/master  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/refs/tags/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/refs/remotes/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/refs/remotes/origin/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/refs/remotes/origin/master  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/branches/
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/description  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/hooks/
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/hooks/applypatch-msg.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/hooks/commit-msg.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/hooks/post-update.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/hooks/pre-applypatch.sample  
    default: 
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/hooks/pre-commit.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/hooks/pre-push.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/hooks/pre-rebase.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/hooks/prepare-commit-msg.sample  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/hooks/update.sample  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/info/
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/info/exclude  
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/HEAD  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/config  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/pack/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/info/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/b1/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/b1/20d9b3ca7009feb9103aaa50f0a7dba6e71265  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/7d/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/7d/13c9b455b959082f76861b3a87171f87c3504e  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/53/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/53/e1144facaa9b3c641e5ed9a9a3c8cb26102e96  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/81/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/81/8cd32be579a6b2c7bd92f487326f5fda025a17  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/8e/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/8e/7a1dcd59c2e2f8c9685bf0e630c2e2d500027f  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/bf/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/bf/6d94292a7689dec6b3991e3e718a3b9449e8af  
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/bf/8b87e0a8d2aefbb22b44d02a13456ddf779590  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/56/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/56/1b864d5bac6f3394c7dc12e10d0d02a510fd16  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/75/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/75/37a76cca4a077df440746686b8a918c25d3826  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/92/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/92/aa507bbeae6788d82be804ec24cf1601aae6a8  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/b6/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/b6/9f26b3cee3637184d556a623814b540b0df127  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/20/
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/objects/20/e8167e3b03c16c8e92dfa35e5d3167c16dba87  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/FETCH_HEAD  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/logs/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/logs/refs/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/logs/refs/remotes/
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/logs/refs/remotes/origin/
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/logs/refs/remotes/origin/master  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/logs/refs/heads/
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/logs/refs/heads/master  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/logs/HEAD  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/.git/index  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/Dockerfile  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/Jenkinsfile  
    default:  extracting: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/README.md  
    default:   inflating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code/server.js  
    default:    creating: /var/lib/jenkins/workspace/cicd_dynamic_v3_pipeline_as_code@tmp/
    default:    creating: /var/lib/jenkins/.docker/
    default:   inflating: /var/lib/jenkins/.docker/.buildNodeID  
    default:    creating: /var/lib/jenkins/.groovy/
    default:    creating: /var/lib/jenkins/.groovy/grapes/
    default:    creating: /var/lib/jenkins/workflow-libs/
    default:   inflating: /var/lib/jenkins/config.xml.20190818.backup  
    default:  extracting: /var/lib/jenkins/org.jenkinsci.plugins.workflow.flow.FlowExecutionList.xml  
    default:   inflating: /var/lib/jenkins/config.xml.20190819.backup  
    default:   inflating: /var/lib/jenkins/queue.xml.bak  
    default:    creating: /var/lib/jenkins/caches/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/refs/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/refs/heads/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/refs/tags/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/refs/remotes/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/refs/remotes/origin/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/refs/remotes/origin/master  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/branches/
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/description  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/hooks/
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/hooks/applypatch-msg.sample  
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/hooks/commit-msg.sample  
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/hooks/post-update.sample  
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/hooks/pre-applypatch.sample  
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/hooks/pre-commit.sample  
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/hooks/pre-push.sample  
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/hooks/pre-rebase.sample  
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/hooks/prepare-commit-msg.sample  
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/hooks/update.sample  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/info/
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/info/exclude  
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/HEAD  
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/config  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/pack/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/info/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/7f/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/7f/f40a2d88161b99efd90e79e2caf5a2757bbd09  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/2f/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/2f/34f5a52760c27e6178ea424e3ae1e27e0445ad  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/12/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/12/fb2da73ab1b3e401d7b944b3732d6eb026bd32  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/b5/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/b5/ffa97c0d9dd637ddb5b5c90feff746208b908a  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/60/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/60/c56ca3f3daa0d19d5c359c24f6922e5091f45a  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/e6/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/49/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/49/2986ff316380b1986d7772b664410cc3065c42  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/bf/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/bf/dc113cef87a41574f8d7ca8074234f31dc6d5f  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/a1/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/a1/7039c3e49e7a03d091eccc73ecb1fe5192498e  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/31/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/31/41fa7691e733278cdfc16f49ebc18eddfd756a  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/45/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/45/e490df1c068d16eb55c06c443e19788f4a0151  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/9d/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/9d/aeafb9864cf43055ae93beb0afd6c7d144bfa4  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/7a/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/7a/92d1730677dc911a9a98b465c80568efcef446  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/d1/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/d1/3a5448f2f4f56e5bb0708fb5e2f1c417e08831  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/4d/
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/4d/7f0d8a5c679da4dc815a7d91af57fc39849683  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/8b/
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/8b/f2a1de43e0bbe10f812edbc999091fff08a849  
    default: 
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/39/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/39/c96b920a3ec094f5143837fe8046b1a5b92896  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/d2/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/d2/1de79d1d8ca1b5a9417b80abbda14901753ce1  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/a0/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/a0/e11093d182807ae80c18ca52e8634b35eeff4f  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/7b/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/7b/ae1d38a02b028a19aab91a63a57f63cd70abd0  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/f9/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/f9/3e3a1a1525fb5b91020da86e44810c87a2d7bc  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/4a/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/4a/3867048d2409c447fdede33efb8e5f50ea8a20  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/9e/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/9e/d170be79d436ebd67d2edfa8d2b047cf256920  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/16/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/16/ff985207c5051fba0d69c7fb6f3f1f5cc8a40c  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/69/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/69/6e516106c63e24a3a09ce9e1969291401392b6  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/35/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/35/51367b5cfefe1508042679cdec728d0ab5cc11  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/95/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/95/c3903ef21da3c180330812d445dfcfd24a929e  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/62/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/62/560add7af41d82c638d67c5d5e641b1e0e6553  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/92/
    default:  extracting: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/objects/92/956413b4f750492cf572ed575a523f5cbc20cd  
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/FETCH_HEAD  
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/logs/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/logs/refs/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/logs/refs/remotes/
    default:    creating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/logs/refs/remotes/origin/
    default:   inflating: /var/lib/jenkins/caches/git-c76dbc979d43cafd92f1aa6b824d29b2/.git/logs/refs/remotes/origin/master  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/refs/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/refs/heads/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/refs/tags/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/refs/remotes/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/refs/remotes/origin/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/refs/remotes/origin/master  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/branches/
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/description  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/hooks/
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/hooks/applypatch-msg.sample  
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/hooks/commit-msg.sample  
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/hooks/post-update.sample  
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/hooks/pre-applypatch.sample  
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/hooks/pre-commit.sample  
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/hooks/pre-push.sample  
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/hooks/pre-rebase.sample  
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/hooks/prepare-commit-msg.sample  
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/hooks/update.sample  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/info/
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/info/exclude  
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/HEAD  
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/config  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/pack/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/info/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/b1/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/b1/20d9b3ca7009feb9103aaa50f0a7dba6e71265  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/7d/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/7d/13c9b455b959082f76861b3a87171f87c3504e  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/53/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/53/e1144facaa9b3c641e5ed9a9a3c8cb26102e96  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/81/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/81/8cd32be579a6b2c7bd92f487326f5fda025a17  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/8e/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/8e/7a1dcd59c2e2f8c9685bf0e630c2e2d500027f  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/bf/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/bf/6d94292a7689dec6b3991e3e718a3b9449e8af  
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/bf/8b87e0a8d2aefbb22b44d02a13456ddf779590  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/56/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/56/1b864d5bac6f3394c7dc12e10d0d02a510fd16  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/75/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/75/37a76cca4a077df440746686b8a918c25d3826  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/92/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/92/aa507bbeae6788d82be804ec24cf1601aae6a8  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/b6/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/b6/9f26b3cee3637184d556a623814b540b0df127  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/20/
    default:  extracting: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/objects/20/e8167e3b03c16c8e92dfa35e5d3167c16dba87  
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/FETCH_HEAD  
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/logs/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/logs/refs/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/logs/refs/remotes/
    default:    creating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/logs/refs/remotes/origin/
    default:   inflating: /var/lib/jenkins/caches/git-fd1f3a6b94c08077929e575f6838c96a/.git/logs/refs/remotes/origin/master  
    default:   inflating: /var/lib/jenkins/queue.xml  
    default: Starting jenkins (via systemctl):  
    default: [  OK  ]
    default: ● jenkins.service - LSB: Jenkins Automation Server
    default:    Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
    default:    Active: active (running) since Thu 2019-08-22 17:10:50 UTC; 80ms ago
    default:      Docs: man:systemd-sysv-generator(8)
    default:   Process: 10517 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)
    default:    CGroup: /system.slice/jenkins.service
    default:            └─10541 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
    default: 
    default: Aug 22 17:10:50 auto systemd[1]: Starting LSB: Jenkins Automation Server...
    default: Aug 22 17:10:50 auto runuser[10522]: pam_unix(runuser:session): session opened for user jenkins by (uid=0)
    default: Aug 22 17:10:50 auto runuser[10522]: pam_unix(runuser:session): session closed for user jenkins
    default: Aug 22 17:10:50 auto jenkins[10517]: Starting Jenkins [  OK  ]
    default: Aug 22 17:10:50 auto systemd[1]: Started LSB: Jenkins Automation Server.
    default: Loaded plugins: fastestmirror
    default: No Match for argument: docker
    default: No Match for argument: docker-client
    default: No Match for argument: docker-client-latest
    default: No Match for argument: docker-common
    default: No Match for argument: docker-latest
    default: No Match for argument: docker-latest-logrotate
    default: No Match for argument: docker-logrotate
    default: No Match for argument: docker-engine
    default: No Packages marked for removal
    default: Loaded plugins: fastestmirror
    default: Loading mirror speeds from cached hostfile
    default:  * base: mirrors.tuna.tsinghua.edu.cn
    default:  * extras: mirror.jdcloud.com
    default:  * updates: mirror.jdcloud.com
    default: Package yum-utils-1.1.31-50.el7.noarch already installed and latest version
    default: Resolving Dependencies
    default: --> Running transaction check
    default: ---> Package device-mapper-persistent-data.x86_64 0:0.7.3-3.el7 will be installed
    default: --> Processing Dependency: libaio.so.1(LIBAIO_0.4)(64bit) for package: device-mapper-persistent-data-0.7.3-3.el7.x86_64
    default: --> Processing Dependency: libaio.so.1(LIBAIO_0.1)(64bit) for package: device-mapper-persistent-data-0.7.3-3.el7.x86_64
    default: --> Processing Dependency: libaio.so.1()(64bit) for package: device-mapper-persistent-data-0.7.3-3.el7.x86_64
    default: ---> Package lvm2.x86_64 7:2.02.180-10.el7_6.8 will be installed
    default: --> Processing Dependency: lvm2-libs = 7:2.02.180-10.el7_6.8 for package: 7:lvm2-2.02.180-10.el7_6.8.x86_64
    default: --> Processing Dependency: liblvm2app.so.2.2(Base)(64bit) for package: 7:lvm2-2.02.180-10.el7_6.8.x86_64
    default: --> Processing Dependency: libdevmapper-event.so.1.02(Base)(64bit) for package: 7:lvm2-2.02.180-10.el7_6.8.x86_64
    default: --> Processing Dependency: liblvm2app.so.2.2()(64bit) for package: 7:lvm2-2.02.180-10.el7_6.8.x86_64
    default: --> Processing Dependency: libdevmapper-event.so.1.02()(64bit) for package: 7:lvm2-2.02.180-10.el7_6.8.x86_64
    default: --> Running transaction check
    default: ---> Package device-mapper-event-libs.x86_64 7:1.02.149-10.el7_6.8 will be installed
    default: ---> Package libaio.x86_64 0:0.3.109-13.el7 will be installed
    default: ---> Package lvm2-libs.x86_64 7:2.02.180-10.el7_6.8 will be installed
    default: --> Processing Dependency: device-mapper-event = 7:1.02.149-10.el7_6.8 for package: 7:lvm2-libs-2.02.180-10.el7_6.8.x86_64
    default: --> Running transaction check
    default: ---> Package device-mapper-event.x86_64 7:1.02.149-10.el7_6.8 will be installed
    default: --> Processing Dependency: device-mapper = 7:1.02.149-10.el7_6.8 for package: 7:device-mapper-event-1.02.149-10.el7_6.8.x86_64
    default: --> Running transaction check
    default: ---> Package device-mapper.x86_64 7:1.02.149-10.el7_6.7 will be updated
    default: --> Processing Dependency: device-mapper = 7:1.02.149-10.el7_6.7 for package: 7:device-mapper-libs-1.02.149-10.el7_6.7.x86_64
    default: ---> Package device-mapper.x86_64 7:1.02.149-10.el7_6.8 will be an update
    default: --> Running transaction check
    default: ---> Package device-mapper-libs.x86_64 7:1.02.149-10.el7_6.7 will be updated
    default: ---> Package device-mapper-libs.x86_64 7:1.02.149-10.el7_6.8 will be an update
    default: --> Finished Dependency Resolution
    default: 
    default: Dependencies Resolved
    default: 
    default: ================================================================================
    default:  Package                        Arch    Version                  Repository
    default:                                                                            Size
    default: ================================================================================
    default: Installing:
    default:  device-mapper-persistent-data  x86_64  0.7.3-3.el7              base     405 k
    default:  lvm2                           x86_64  7:2.02.180-10.el7_6.8    updates  1.3 M
    default: Installing for dependencies:
    default:  device-mapper-event            x86_64  7:1.02.149-10.el7_6.8    updates  189 k
    default:  device-mapper-event-libs       x86_64  7:1.02.149-10.el7_6.8    updates  188 k
    default:  libaio                         x86_64  0.3.109-13.el7           base      24 k
    default:  lvm2-libs                      x86_64  7:2.02.180-10.el7_6.8    updates  1.1 M
    default: Updating for dependencies:
    default:  device-mapper                  x86_64  7:1.02.149-10.el7_6.8    updates  293 k
    default:  device-mapper-libs             x86_64  7:1.02.149-10.el7_6.8    updates  321 k
    default: 
    default: Transaction Summary
    default: ================================================================================
    default: Install  2 Packages (+4 Dependent packages)
    default: Upgrade             ( 2 Dependent packages)
    default: 
    default: Total download size: 3.8 M
    default: Downloading packages:
    default: Delta RPMs reduced 321 k of updates to 171 k (46% saved)
    default: --------------------------------------------------------------------------------
    default: Total                                              5.1 MB/s | 3.6 MB  00:00     
    default: Running transaction check
    default: Running transaction test
    default: Transaction test succeeded
    default: Running transaction
    default:   Updating   : 7:device-mapper-1.02.149-10.el7_6.8.x86_64                  1/10
    default:  
    default:   Updating   : 7:device-mapper-libs-1.02.149-10.el7_6.8.x86_64             2/10
    default:  
    default:   Installing : 7:device-mapper-event-libs-1.02.149-10.el7_6.8.x86_64       3/10
    default:  
    default:   Installing : libaio-0.3.109-13.el7.x86_64                                4/10
    default:  
    default:   Installing : device-mapper-persistent-data-0.7.3-3.el7.x86_64            5/10
    default:  
    default:   Installing : 7:device-mapper-event-1.02.149-10.el7_6.8.x86_64            6/10
    default:  
    default:   Installing : 7:lvm2-libs-2.02.180-10.el7_6.8.x86_64                      7/10
    default:  
    default:   Installing : 7:lvm2-2.02.180-10.el7_6.8.x86_64                           8/10
    default:  
    default:   Cleanup    : 7:device-mapper-1.02.149-10.el7_6.7.x86_64                  9/10
    default:  
    default:   Cleanup    : 7:device-mapper-libs-1.02.149-10.el7_6.7.x86_64            10/10
    default:  
    default:   Verifying  : device-mapper-persistent-data-0.7.3-3.el7.x86_64            1/10
    default:  
    default:   Verifying  : 7:device-mapper-event-libs-1.02.149-10.el7_6.8.x86_64       2/10
    default:  
    default:   Verifying  : 7:device-mapper-libs-1.02.149-10.el7_6.8.x86_64             3/10
    default:  
    default:   Verifying  : 7:lvm2-2.02.180-10.el7_6.8.x86_64                           4/10
    default:  
    default:   Verifying  : 7:lvm2-libs-2.02.180-10.el7_6.8.x86_64                      5/10
    default:  
    default:   Verifying  : libaio-0.3.109-13.el7.x86_64                                6/10
    default:  
    default:   Verifying  : 7:device-mapper-1.02.149-10.el7_6.8.x86_64                  7/10
    default:  
    default:   Verifying  : 7:device-mapper-event-1.02.149-10.el7_6.8.x86_64            8/10
    default:  
    default:   Verifying  : 7:device-mapper-libs-1.02.149-10.el7_6.7.x86_64             9/10 
    default:   Verifying  : 7:device-mapper-1.02.149-10.el7_6.7.x86_64                 10/10
    default:  
    default: 
    default: Installed:
    default:   device-mapper-persistent-data.x86_64 0:0.7.3-3.el7                            
    default:   lvm2.x86_64 7:2.02.180-10.el7_6.8                                             
    default: 
    default: Dependency Installed:
    default:   device-mapper-event.x86_64 7:1.02.149-10.el7_6.8                              
    default:   device-mapper-event-libs.x86_64 7:1.02.149-10.el7_6.8                         
    default:   libaio.x86_64 0:0.3.109-13.el7                                                
    default:   lvm2-libs.x86_64 7:2.02.180-10.el7_6.8                                        
    default: 
    default: Dependency Updated:
    default:   device-mapper.x86_64 7:1.02.149-10.el7_6.8                                    
    default:   device-mapper-libs.x86_64 7:1.02.149-10.el7_6.8                               
    default: 
    default: Complete!
    default: Loaded plugins: fastestmirror
    default: Examining docker-ce-19.03.1-3.el7.x86_64.rpm: 3:docker-ce-19.03.1-3.el7.x86_64
    default: Marking docker-ce-19.03.1-3.el7.x86_64.rpm to be installed
    default: Examining docker-ce-cli-19.03.1-3.el7.x86_64.rpm: 1:docker-ce-cli-19.03.1-3.el7.x86_64
    default: Marking docker-ce-cli-19.03.1-3.el7.x86_64.rpm to be installed
    default: Examining containerd.io-1.2.6-3.3.el7.x86_64.rpm: containerd.io-1.2.6-3.3.el7.x86_64
    default: Marking containerd.io-1.2.6-3.3.el7.x86_64.rpm to be installed
    default: Resolving Dependencies
    default: --> Running transaction check
    default: ---> Package containerd.io.x86_64 0:1.2.6-3.3.el7 will be installed
    default: --> Processing Dependency: container-selinux >= 2:2.74 for package: containerd.io-1.2.6-3.3.el7.x86_64
    default: Loading mirror speeds from cached hostfile
    default:  * base: mirrors.tuna.tsinghua.edu.cn
    default:  * extras: mirror.jdcloud.com
    default:  * updates: mirror.jdcloud.com
    default: ---> Package docker-ce.x86_64 3:19.03.1-3.el7 will be installed
    default: --> Processing Dependency: libcgroup for package: 3:docker-ce-19.03.1-3.el7.x86_64
    default: ---> Package docker-ce-cli.x86_64 1:19.03.1-3.el7 will be installed
    default: --> Running transaction check
    default: ---> Package container-selinux.noarch 2:2.107-1.el7_6 will be installed
    default: --> Processing Dependency: policycoreutils-python for package: 2:container-selinux-2.107-1.el7_6.noarch
    default: ---> Package libcgroup.x86_64 0:0.41-20.el7 will be installed
    default: --> Running transaction check
    default: ---> Package policycoreutils-python.x86_64 0:2.5-29.el7_6.1 will be installed
    default: --> Processing Dependency: setools-libs >= 3.3.8-4 for package: policycoreutils-python-2.5-29.el7_6.1.x86_64
    default: --> Processing Dependency: libsemanage-python >= 2.5-14 for package: policycoreutils-python-2.5-29.el7_6.1.x86_64
    default: --> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.5-29.el7_6.1.x86_64
    default: --> Processing Dependency: python-IPy for package: policycoreutils-python-2.5-29.el7_6.1.x86_64
    default: --> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.5-29.el7_6.1.x86_64
    default: --> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.5-29.el7_6.1.x86_64
    default: --> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.5-29.el7_6.1.x86_64
    default: --> Processing Dependency: checkpolicy for package: policycoreutils-python-2.5-29.el7_6.1.x86_64
    default: --> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.5-29.el7_6.1.x86_64
    default: --> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.5-29.el7_6.1.x86_64
    default: --> Running transaction check
    default: ---> Package audit-libs-python.x86_64 0:2.8.4-4.el7 will be installed
    default: ---> Package checkpolicy.x86_64 0:2.5-8.el7 will be installed
    default: ---> Package libsemanage-python.x86_64 0:2.5-14.el7 will be installed
    default: ---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
    default: ---> Package setools-libs.x86_64 0:3.3.8-4.el7 will be installed
    default: --> Finished Dependency Resolution
    default: 
    default: Dependencies Resolved
    default: 
    default: ================================================================================
    default:  Package                Arch   Version         Repository                  Size
    default: ================================================================================
    default: Installing:
    default:  containerd.io          x86_64 1.2.6-3.3.el7   /containerd.io-1.2.6-3.3.el7.x86_64
    default:                                                                            96 M
    default:  docker-ce              x86_64 3:19.03.1-3.el7 /docker-ce-19.03.1-3.el7.x86_64
    default:                                                                           104 M
    default:  docker-ce-cli          x86_64 1:19.03.1-3.el7 /docker-ce-cli-19.03.1-3.el7.x86_64
    default:                                                                           169 M
    default: Installing for dependencies:
    default:  audit-libs-python      x86_64 2.8.4-4.el7     base                        76 k
    default:  checkpolicy            x86_64 2.5-8.el7       base                       295 k
    default:  container-selinux      noarch 2:2.107-1.el7_6 extras                      39 k
    default:  libcgroup              x86_64 0.41-20.el7     base                        66 k
    default:  libsemanage-python     x86_64 2.5-14.el7      base                       113 k
    default:  policycoreutils-python x86_64 2.5-29.el7_6.1  updates                    456 k
    default:  python-IPy             noarch 0.75-6.el7      base                        32 k
    default:  setools-libs           x86_64 3.3.8-4.el7     base                       620 k
    default: 
    default: Transaction Summary
    default: ================================================================================
    default: Install  3 Packages (+8 Dependent packages)
    default: 
    default: Total size: 370 M
    default: Total download size: 1.7 M
    default: Installed size: 374 M
    default: Downloading packages:
    default: --------------------------------------------------------------------------------
    default: Total                                              2.1 MB/s | 1.7 MB  00:00     
    default: Running transaction check
    default: Running transaction test
    default: Transaction test succeeded
    default: Running transaction
    default:   Installing : libcgroup-0.41-20.el7.x86_64                                1/11
    default:  
    default:   Installing : setools-libs-3.3.8-4.el7.x86_64                             2/11
    default:  
    default:   Installing : 1:docker-ce-cli-19.03.1-3.el7.x86_64                        3/11
    default:  
    default:   Installing : python-IPy-0.75-6.el7.noarch                                4/11
    default:  
    default:   Installing : libsemanage-python-2.5-14.el7.x86_64                        5/11
    default:  
    default:   Installing : audit-libs-python-2.8.4-4.el7.x86_64                        6/11
    default:  
    default:   Installing : checkpolicy-2.5-8.el7.x86_64                                7/11
    default:  
    default:   Installing : policycoreutils-python-2.5-29.el7_6.1.x86_64                8/11
    default:  
    default:   Installing : 2:container-selinux-2.107-1.el7_6.noarch                    9/11
    default:  
    default:   Installing : containerd.io-1.2.6-3.3.el7.x86_64                         10/11
    default:  
    default:   Installing : 3:docker-ce-19.03.1-3.el7.x86_64                           11/11
    default:  
    default:   Verifying  : libcgroup-0.41-20.el7.x86_64                                1/11
    default:  
    default:   Verifying  : checkpolicy-2.5-8.el7.x86_64                                2/11
    default:  
    default:   Verifying  : policycoreutils-python-2.5-29.el7_6.1.x86_64                3/11
    default:  
    default:   Verifying  : audit-libs-python-2.8.4-4.el7.x86_64                        4/11
    default:  
    default:   Verifying  : libsemanage-python-2.5-14.el7.x86_64                        5/11
    default:  
    default:   Verifying  : 2:container-selinux-2.107-1.el7_6.noarch                    6/11
    default:  
    default:   Verifying  : python-IPy-0.75-6.el7.noarch                                7/11
    default:  
    default:   Verifying  : containerd.io-1.2.6-3.3.el7.x86_64                          8/11
    default:  
    default:   Verifying  : 1:docker-ce-cli-19.03.1-3.el7.x86_64                        9/11
    default:  
    default:   Verifying  : 3:docker-ce-19.03.1-3.el7.x86_64                           10/11
    default:  
    default:   Verifying  : setools-libs-3.3.8-4.el7.x86_64                            11/11
    default:  
    default: 
    default: Installed:
    default:   containerd.io.x86_64 0:1.2.6-3.3.el7     docker-ce.x86_64 3:19.03.1-3.el7    
    default:   docker-ce-cli.x86_64 1:19.03.1-3.el7    
    default: 
    default: Dependency Installed:
    default:   audit-libs-python.x86_64 0:2.8.4-4.el7                                        
    default:   checkpolicy.x86_64 0:2.5-8.el7                                                
    default:   container-selinux.noarch 2:2.107-1.el7_6                                      
    default:   libcgroup.x86_64 0:0.41-20.el7                                                
    default:   libsemanage-python.x86_64 0:2.5-14.el7                                        
    default:   policycoreutils-python.x86_64 0:2.5-29.el7_6.1                                
    default:   python-IPy.noarch 0:0.75-6.el7                                                
    default:   setools-libs.x86_64 0:3.3.8-4.el7                                             
    default: 
    default: Complete!
    default: Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
    default: ● docker.service - Docker Application Container Engine
    default:    Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
    default:    Active: active (running) since Thu 2019-08-22 17:11:31 UTC; 38ms ago
    default:      Docs: https://docs.docker.com
    default:  Main PID: 10898 (dockerd)
    default:     Tasks: 13
    default:    Memory: 42.0M
    default:    CGroup: /system.slice/docker.service
    default:            └─10898 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
    default: 
    default: Aug 22 17:11:31 auto dockerd[10898]: time="2019-08-22T17:11:31.297142956Z" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
    default: Aug 22 17:11:31 auto dockerd[10898]: time="2019-08-22T17:11:31.297187812Z" level=info msg="pickfirstBalancer: HandleSubConnStateChange: 0xc0006be9e0, CONNECTING" module=grpc
    default: Aug 22 17:11:31 auto dockerd[10898]: time="2019-08-22T17:11:31.297820073Z" level=info msg="pickfirstBalancer: HandleSubConnStateChange: 0xc0006be9e0, READY" module=grpc
    default: Aug 22 17:11:31 auto dockerd[10898]: time="2019-08-22T17:11:31.515597524Z" level=info msg="Loading containers: start."
    default: Aug 22 17:11:31 auto dockerd[10898]: time="2019-08-22T17:11:31.644557050Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
    default: Aug 22 17:11:31 auto dockerd[10898]: time="2019-08-22T17:11:31.702660839Z" level=info msg="Loading containers: done."
    default: Aug 22 17:11:31 auto dockerd[10898]: time="2019-08-22T17:11:31.744486527Z" level=info msg="Docker daemon" commit=74b1e89 graphdriver(s)=overlay2 version=19.03.1
    default: Aug 22 17:11:31 auto dockerd[10898]: time="2019-08-22T17:11:31.744997982Z" level=info msg="Daemon has completed initialization"
    default: Aug 22 17:11:31 auto dockerd[10898]: time="2019-08-22T17:11:31.762590991Z" level=info msg="API listen on /var/run/docker.sock"
    default: Aug 22 17:11:31 auto systemd[1]: Started Docker Application Container Engine.
    default: Unable to find image 'hello-world:latest' locally
    default: latest: Pulling from library/hello-world
    default: 1b930d010525: Pulling fs layer
    default: 1b930d010525: Download complete
    default: 1b930d010525: Pull complete
    default: Digest: sha256:451ce787d12369c5df2a32c85e5a03d52cbcef6eb3586dd03075f3034f10adcd
    default: Status: Downloaded newer image for hello-world:latest
    default: 
    default: Hello from Docker!
    default: This message shows that your installation appears to be working correctly.
    default: 
    default: To generate this message, Docker took the following steps:
    default:  1. The Docker client contacted the Docker daemon.
    default:  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    default:     (amd64)
    default:  3. The Docker daemon created a new container from that image which runs the
    default:     executable that produces the output you are currently reading.
    default:  4. The Docker daemon streamed that output to the Docker client, which sent it
    default:     to your terminal.
    default: 
    default: To try something more ambitious, you can run an Ubuntu container with:
    default:  $ docker run -it ubuntu bash
    default: 
    default: Share images, automate workflows, and more with a free Docker ID:
    default:  https://hub.docker.com/
    default: 
    default: For more examples and ideas, visit:
    default:  https://docs.docker.com/get-started/
    default: docker version >= 1.12
    default: {"registry-mirrors": ["http://f1361db2.m.daocloud.io"]}
    default: Success.
    default: You need to restart docker to take effect: sudo systemctl restart docker 
    default: ● docker.service - Docker Application Container Engine
    default:    Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
    default:    Active: active (running) since Thu 2019-08-22 17:11:59 UTC; 23ms ago
    default:      Docs: https://docs.docker.com
    default:  Main PID: 11286 (dockerd)
    default:     Tasks: 13
    default:    Memory: 43.9M
    default:    CGroup: /system.slice/docker.service
    default:            └─11286 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
    default: 
    default: Aug 22 17:11:58 auto dockerd[11286]: time="2019-08-22T17:11:58.947880648Z" level=info msg="pickfirstBalancer: HandleSubConnStateChange: 0xc000866620, CONNECTING" module=grpc
    default: Aug 22 17:11:58 auto dockerd[11286]: time="2019-08-22T17:11:58.948275382Z" level=info msg="pickfirstBalancer: HandleSubConnStateChange: 0xc000866620, READY" module=grpc
    default: Aug 22 17:11:58 auto dockerd[11286]: time="2019-08-22T17:11:58.964607528Z" level=info msg="[graphdriver] using prior storage driver: overlay2"
    default: Aug 22 17:11:58 auto dockerd[11286]: time="2019-08-22T17:11:58.971908586Z" level=info msg="Loading containers: start."
    default: Aug 22 17:11:59 auto dockerd[11286]: time="2019-08-22T17:11:59.092077195Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
    default: Aug 22 17:11:59 auto dockerd[11286]: time="2019-08-22T17:11:59.140494651Z" level=info msg="Loading containers: done."
    default: Aug 22 17:11:59 auto dockerd[11286]: time="2019-08-22T17:11:59.167767602Z" level=info msg="Docker daemon" commit=74b1e89 graphdriver(s)=overlay2 version=19.03.1
    default: Aug 22 17:11:59 auto dockerd[11286]: time="2019-08-22T17:11:59.167847429Z" level=info msg="Daemon has completed initialization"
    default: Aug 22 17:11:59 auto dockerd[11286]: time="2019-08-22T17:11:59.183866340Z" level=info msg="API listen on /var/run/docker.sock"
    default: Aug 22 17:11:59 auto systemd[1]: Started Docker Application Container Engine.
    default:   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
    default:                                  Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
 12 39.8M   12 4927k    0     0  4871k      0  0:00:08  0:00:01  0:00:07 4873k
 34 39.8M   34 13.6M    0     0  6972k      0  0:00:05  0:00:02  0:00:03 6974k
 57 39.8M   57 22.8M    0     0  7757k      0  0:00:05  0:00:03  0:00:02 7756k
 80 39.8M   80 32.2M    0     0  8231k      0  0:00:04  0:00:04 --:--:-- 8231k
100 39.8M  100 39.8M    0     0  8516k      0  0:00:04  0:00:04 --:--:-- 8540k
    default: * minikube v1.2.0 on linux (amd64)
    default: * using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
    default: * Creating none VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
    default: * Configuring environment for Kubernetes v1.15.0 on Docker 19.03.1
    default: * Downloading kubelet v1.15.0
    default: * Downloading kubeadm v1.15.0
    default: * Pulling images ...
    default: * Launching Kubernetes ... 
    default: * Configuring local host environment ...
    default: 
    default:   - https://github.com/kubernetes/minikube/blob/master/docs/vmdriver-none.md
    default: 
    default: 
    default:   - sudo mv /root/.kube /root/.minikube $HOME
    default:   - sudo chown -R $USER $HOME/.kube $HOME/.minikube
    default: 
    default: * This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
    default: * Verifying:
    default: ! The 'none' driver provides limited isolation and may reduce system security and reliability.
    default: ! For more information, see:
    default: ! kubectl and minikube configuration will be stored in /root
    default: ! To use kubectl or minikube commands as your own user, you may
    default: ! need to relocate them. For example, to overwrite your own settings:
    default:  apiserver
    default:  proxy
    default:  etcd
    default:  scheduler
    default:  controller
    default:  dns
    default: 
    default: * Done! kubectl is now configured to use "minikube"
    default: * heapster was successfully enabled
    default: 6.14.2: Pulling from library/node
    default: 3d77ce4481b1: Pulling fs layer
    default: 7d2f32934963: Pulling fs layer
    default: 0c5cf711b890: Pulling fs layer
    default: 9593dc852d6b: Pulling fs layer
    default: 4e3b8a1eb914: Pulling fs layer
    default: ddcf13cc1951: Pulling fs layer
    default: 2e460d114172: Pulling fs layer
    default: d94b1226fbf2: Pulling fs layer
    default: 4e3b8a1eb914: Waiting
    default: ddcf13cc1951: Waiting
    default: 2e460d114172: Waiting
    default: d94b1226fbf2: 
    default: Waiting
    default: 9593dc852d6b: Waiting
    default: 7d2f32934963: Verifying Checksum
    default: 7d2f32934963: Download complete
    default: 0c5cf711b890: Verifying Checksum
    default: 0c5cf711b890: Download complete
    default: 4e3b8a1eb914: Verifying Checksum
    default: 4e3b8a1eb914: Download complete
    default: ddcf13cc1951: Verifying Checksum
    default: ddcf13cc1951: Download complete
    default: 3d77ce4481b1: Verifying Checksum
    default: 3d77ce4481b1: Download complete
    default: d94b1226fbf2: Verifying Checksum
    default: d94b1226fbf2: Download complete
    default: 3d77ce4481b1: Pull complete
    default: 7d2f32934963: Pull complete
    default: 0c5cf711b890: Pull complete
    default: 9593dc852d6b: Verifying Checksum
    default: 9593dc852d6b: Download complete
    default: 9593dc852d6b: Pull complete
    default: 4e3b8a1eb914: Pull complete
    default: ddcf13cc1951: Pull complete
    default: 2e460d114172: Verifying Checksum
    default: 2e460d114172: Download complete
    default: 2e460d114172: Pull complete
    default: d94b1226fbf2: Pull complete
    default: Digest: sha256:85a878cb14deb5a5a59944e906d43ef400e182e9fcc81a6beb18907f52309261
    default: Status: Downloaded newer image for daocloud.io/library/node:6.14.2
    default: daocloud.io/library/node:6.14.2
    default: Using default tag: latest
    default: latest: Pulling from library/nginx
    default: 0a4690c5d889: Pulling fs layer
    default: 9719afee3eb7: Pulling fs layer
    default: 44446b456159: Pulling fs layer
    default: 44446b456159: Verifying Checksum
    default: 44446b456159: Download complete
    default: 9719afee3eb7: Verifying Checksum
    default: 9719afee3eb7: 
    default: Download complete
    default: 0a4690c5d889: Verifying Checksum
    default: 0a4690c5d889: Download complete
    default: 0a4690c5d889: Pull complete
    default: 9719afee3eb7: Pull complete
    default: 44446b456159: Pull complete
    default: Digest: sha256:f83b2ffd963ac911f9e638184c8d580cc1f3139d5c8c33c87c3fb90aebdebf76
    default: Status: Downloaded newer image for daocloud.io/library/nginx:latest
    default: daocloud.io/library/nginx:latest
    default: Using default tag: latest
    default: latest: Pulling from gogs/gogs
    default: [DEPRECATION NOTICE] registry v2 schema1 support will be removed in an upcoming release. Please contact admins of the docker.io registry NOW to avoid future disruption.
    default: 050382585609: 
    default: Pulling fs layer
    default: da970bb7d5e8: Pulling fs layer
    default: 3d985387bce7: Pulling fs layer
    default: cafc497b4de2: Pulling fs layer
    default: a4be98e46571: Pulling fs layer
    default: 97142f536ce9: Pulling fs layer
    default: e208b6376d69: Pulling fs layer
    default: 153cbf5ae1ce: Pulling fs layer
    default: f3008dcd5700: Pulling fs layer
    default: 0ebc0ebf1a9b: Pulling fs layer
    default: 97142f536ce9: Waiting
    default: e208b6376d69: Waiting
    default: cafc497b4de2: Waiting
    default: a4be98e46571: Waiting
    default: 153cbf5ae1ce: Waiting
    default: f3008dcd5700: Waiting
    default: 0ebc0ebf1a9b: Waiting
    default: da970bb7d5e8: Verifying Checksum
    default: da970bb7d5e8: Download complete
    default: cafc497b4de2: Verifying Checksum
    default: cafc497b4de2: Download complete
    default: a4be98e46571: Verifying Checksum
    default: a4be98e46571: Download complete
    default: 3d985387bce7: Verifying Checksum
    default: 3d985387bce7: Download complete
    default: e208b6376d69: Verifying Checksum
    default: e208b6376d69: Download complete
    default: 050382585609: Verifying Checksum
    default: 050382585609: Download complete
    default: 050382585609: Pull complete
    default: da970bb7d5e8: Pull complete
    default: 3d985387bce7: Pull complete
    default: cafc497b4de2: Pull complete
    default: a4be98e46571: Pull complete
    default: 153cbf5ae1ce: Verifying Checksum
    default: 153cbf5ae1ce: Download complete
    default: 0ebc0ebf1a9b: Download complete
    default: 97142f536ce9: Verifying Checksum
    default: 97142f536ce9: Download complete
    default: 97142f536ce9: Pull complete
    default: e208b6376d69: Pull complete
    default: 153cbf5ae1ce: Pull complete
    default: f3008dcd5700: Verifying Checksum
    default: f3008dcd5700: Download complete
    default: f3008dcd5700: Pull complete
    default: 0ebc0ebf1a9b: Pull complete
    default: Digest: sha256:0474162845fa99a80d58a9fc94ac35fcc72e3519fa9e96c1a97230c354182939
    default: Status: Downloaded newer image for gogs/gogs:latest
    default: docker.io/gogs/gogs:latest
    default: Sending build context to Docker daemon  300.5MB
    default: Step 1/5 : FROM daocloud.io/library/node:6.14.2
    default:  ---> 00165cd5d0c0
    default: Step 2/5 : EXPOSE 8080
    default:  ---> Running in eafbf9790ae4
    default: Removing intermediate container eafbf9790ae4
    default:  ---> b4c7211e7cdb
    default: Step 3/5 : ENV ENV_TAG INIT
    default:  ---> Running in 8d0ab1fd3205
    default: Removing intermediate container 8d0ab1fd3205
    default:  ---> fb4c8dcfcc95
    default: Step 4/5 : COPY server.js .
    default:  ---> 7d82243934ce
    default: Step 5/5 : CMD node server.js
    default:  ---> Running in 402c09b6e80f
    default: Removing intermediate container 402c09b6e80f
    default:  ---> a16efbc141a0
    default: Successfully built a16efbc141a0
    default: Successfully tagged registry.cn-hangzhou.aliyuncs.com/wilmos/nodejs:INIT_v1
    default: deployment.apps/nodejs created
    default: service/nodejs exposed
    default: deployment.apps/nodejs-stag created
    default: service/nodejs-stag exposed
    default: deployment.apps/nodejs-prod created
    default: service/nodejs-prod exposed
    default: nodejs dev/test env:
    default: http://10.1.0.165:30374
    default: nodejs stage env:
    default: http://10.1.0.165:31334
    default: nodejs prod env:
    default: http://10.1.0.165:30818
    default: Sending build context to Docker daemon  300.5MB
    default: Step 1/2 : FROM daocloud.io/library/nginx
    default:  ---> 98ebf73aba75
    default: Step 2/2 : COPY env.txt /usr/share/nginx/html/
    default:  ---> be9c775ae699
    default: Successfully built be9c775ae699
    default: Successfully tagged registry.cn-hangzhou.aliyuncs.com/wilmos/nginx:INIT_v1
    default: deployment.apps/nginx created
    default: service/nginx exposed
    default: deployment.apps/nginx-stag created
    default: service/nginx-stag exposed
    default: deployment.apps/nginx-prod created
    default: service/nginx-prod exposed
    default: nginx dev/test env:
    default: http://10.1.0.165:32547/env.txt
    default: nginx stage env:
    default: http://10.1.0.165:32404/env.txt
    default: nginx prod env:
    default: http://10.1.0.165:30219/env.txt
    default: Archive:  /tmp/package/gogs.zip
    default:    creating: /var/lib/gogs/
    default:    creating: /var/lib/gogs/gogs/
    default:    creating: /var/lib/gogs/gogs/data/
    default:   inflating: /var/lib/gogs/gogs/data/gogs.db  
    default:    creating: /var/lib/gogs/gogs/data/sessions/
    default:    creating: /var/lib/gogs/gogs/data/sessions/7/
    default:    creating: /var/lib/gogs/gogs/data/sessions/7/9/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/7/9/793b70c943d2deee  
    default:    creating: /var/lib/gogs/gogs/data/sessions/7/d/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/7/d/7d259eb271eaaf92  
    default:    creating: /var/lib/gogs/gogs/data/sessions/7/5/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/7/5/75367c0d6dea84aa  
    default:    creating: /var/lib/gogs/gogs/data/sessions/7/4/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/7/4/7476f37d2d40cad2  
    default:    creating: /var/lib/gogs/gogs/data/sessions/7/8/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/7/8/782b9fc51c914c88  
    default:    creating: /var/lib/gogs/gogs/data/sessions/2/
    default:    creating: /var/lib/gogs/gogs/data/sessions/2/2/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/2/2/22c26f1e8634167d  
    default:    creating: /var/lib/gogs/gogs/data/sessions/5/
    default:    creating: /var/lib/gogs/gogs/data/sessions/5/0/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/5/0/50adbc59e3e65463  
    default:    creating: /var/lib/gogs/gogs/data/sessions/5/b/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/5/b/5b63c396f4491873  
    default:    creating: /var/lib/gogs/gogs/data/sessions/5/3/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/5/3/5365014472cb7e1c  
    default:    creating: /var/lib/gogs/gogs/data/sessions/5/5/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/5/5/5587c15cee0e5899  
    default:    creating: /var/lib/gogs/gogs/data/sessions/1/
    default:    creating: /var/lib/gogs/gogs/data/sessions/1/2/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/1/2/1200091795e32501  
    default:    creating: /var/lib/gogs/gogs/data/sessions/1/1/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/1/1/11164f75f5cfdfb6  
    default:   inflating: /var/lib/gogs/gogs/data/sessions/1/1/111bb136960173c8  
    default:    creating: /var/lib/gogs/gogs/data/sessions/1/7/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/1/7/1783c2edec363475  
    default:    creating: /var/lib/gogs/gogs/data/sessions/1/b/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/1/b/1b28e05b03fb26b7  
    default:    creating: /var/lib/gogs/gogs/data/sessions/a/
    default:    creating: /var/lib/gogs/gogs/data/sessions/a/c/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/a/c/acd942089560660e  
    default:    creating: /var/lib/gogs/gogs/data/sessions/a/5/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/a/5/a54fb59c08651aee  
    default:    creating: /var/lib/gogs/gogs/data/sessions/a/6/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/a/6/a6fdfdba023342ab  
    default:    creating: /var/lib/gogs/gogs/data/sessions/0/
    default:    creating: /var/lib/gogs/gogs/data/sessions/0/5/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/0/5/05479de1db7bee3f  
    default:    creating: /var/lib/gogs/gogs/data/sessions/0/0/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/0/0/00e9184b08c15721  
    default:    creating: /var/lib/gogs/gogs/data/sessions/0/7/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/0/7/0757b6e3309d3ff3  
    default:    creating: /var/lib/gogs/gogs/data/sessions/0/1/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/0/1/01a5f041e0f538f1  
    default:   inflating: /var/lib/gogs/gogs/data/sessions/0/1/018352a6c7e7162b  
    default:    creating: /var/lib/gogs/gogs/data/sessions/3/
    default:    creating: /var/lib/gogs/gogs/data/sessions/3/4/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/3/4/3491f15ce96ef069  
    default:    creating: /var/lib/gogs/gogs/data/sessions/3/c/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/3/c/3ce5333f12f01400  
    default:    creating: /var/lib/gogs/gogs/data/sessions/3/a/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/3/a/3a4529acfa6ec22c  
    default:   inflating: /var/lib/gogs/gogs/data/sessions/3/a/3a34438c1071e583  
    default:    creating: /var/lib/gogs/gogs/data/sessions/3/5/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/3/5/35f6eec9d57b9865  
    default:    creating: /var/lib/gogs/gogs/data/sessions/f/
    default:    creating: /var/lib/gogs/gogs/data/sessions/f/5/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/f/5/f54e169ea2bbfd5e  
    default:    creating: /var/lib/gogs/gogs/data/sessions/f/a/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/f/a/fa406fccd3835b0d  
    default:    creating: /var/lib/gogs/gogs/data/sessions/4/
    default:    creating: /var/lib/gogs/gogs/data/sessions/4/1/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/4/1/414a4e894c1ef2f1  
    default:    creating: /var/lib/gogs/gogs/data/sessions/4/e/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/4/e/4eada1a0377c3e7a  
    default:    creating: /var/lib/gogs/gogs/data/sessions/4/f/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/4/f/4f601870f7f9bd95  
    default:    creating: /var/lib/gogs/gogs/data/sessions/4/d/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/4/d/4d9a06fef3254d1a  
    default:    creating: /var/lib/gogs/gogs/data/sessions/9/
    default:    creating: /var/lib/gogs/gogs/data/sessions/9/5/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/9/5/955d8fd4d5171fd4  
    default:    creating: /var/lib/gogs/gogs/data/sessions/9/0/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/9/0/90cc71c958fb50b8  
    default:    creating: /var/lib/gogs/gogs/data/sessions/9/f/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/9/f/9f82da4ef07ecd48  
    default:    creating: /var/lib/gogs/gogs/data/sessions/9/8/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/9/8/98cf602aa269f2da  
    default:    creating: /var/lib/gogs/gogs/data/sessions/9/d/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/9/d/9d0872b22532b64e  
    default:    creating: /var/lib/gogs/gogs/data/sessions/d/
    default:    creating: /var/lib/gogs/gogs/data/sessions/d/6/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/d/6/d6ae2e90ed6d0414  
    default:    creating: /var/lib/gogs/gogs/data/sessions/d/a/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/d/a/daba9d45f1949e18  
    default:    creating: /var/lib/gogs/gogs/data/sessions/d/e/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/d/e/de1fd92c7a62ed7c  
    default:    creating: /var/lib/gogs/gogs/data/sessions/d/1/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/d/1/d1e4831ec4a75685  
    default:    creating: /var/lib/gogs/gogs/data/sessions/6/
    default:    creating: /var/lib/gogs/gogs/data/sessions/6/6/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/6/6/663f4539f0232330  
    default:    creating: /var/lib/gogs/gogs/data/sessions/c/
    default:    creating: /var/lib/gogs/gogs/data/sessions/c/9/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/c/9/c9898542bf2e09ae  
    default:    creating: /var/lib/gogs/gogs/data/sessions/c/2/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/c/2/c217eb76466f59ce  
    default:    creating: /var/lib/gogs/gogs/data/sessions/c/4/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/c/4/c419788d75bd9329  
    default:    creating: /var/lib/gogs/gogs/data/sessions/c/0/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/c/0/c0025bffa0948f65  
    default:    creating: /var/lib/gogs/gogs/data/sessions/8/
    default:    creating: /var/lib/gogs/gogs/data/sessions/8/e/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/8/e/8e0b1dd61c87cd69  
    default:    creating: /var/lib/gogs/gogs/data/sessions/8/8/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/8/8/888222d3aed2253a  
    default:    creating: /var/lib/gogs/gogs/data/sessions/8/b/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/8/b/8b872d7ea3b3beb8  
    default:    creating: /var/lib/gogs/gogs/data/sessions/b/
    default:    creating: /var/lib/gogs/gogs/data/sessions/b/b/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/b/b/bbf7dfb491247d5d  
    default:    creating: /var/lib/gogs/gogs/data/sessions/b/6/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/b/6/b6f6dccb728fb4e9  
    default:    creating: /var/lib/gogs/gogs/data/sessions/b/f/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/b/f/bfa61487f73e04b2  
    default:    creating: /var/lib/gogs/gogs/data/sessions/e/
    default:    creating: /var/lib/gogs/gogs/data/sessions/e/1/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/e/1/e175583b780e16cf  
    default:    creating: /var/lib/gogs/gogs/data/sessions/e/0/
    default:   inflating: /var/lib/gogs/gogs/data/sessions/e/0/e04323dffe1f3062  
    default:    creating: /var/lib/gogs/gogs/conf/
    default:   inflating: /var/lib/gogs/gogs/conf/app.ini  
    default:    creating: /var/lib/gogs/gogs/log/
    default:   inflating: /var/lib/gogs/gogs/log/gogs.log  
    default:  extracting: /var/lib/gogs/gogs/log/xorm.log  
    default:    creating: /var/lib/gogs/gogs/log/hooks/
    default:  extracting: /var/lib/gogs/gogs/log/hooks/pre-receive.log  
    default:  extracting: /var/lib/gogs/gogs/log/hooks/update.log  
    default:  extracting: /var/lib/gogs/gogs/log/hooks/post-receive.log  
    default:    creating: /var/lib/gogs/git/
    default:    creating: /var/lib/gogs/git/.ssh/
    default:  extracting: /var/lib/gogs/git/.ssh/environment  
    default:    creating: /var/lib/gogs/git/gogs-repositories/
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/branches/
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/description  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/applypatch-msg.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/commit-msg.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/post-update.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/pre-applypatch.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/pre-commit.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/pre-push.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/pre-rebase.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/pre-receive.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/prepare-commit-msg.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/update.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/pre-receive  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/update  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/hooks/post-receive  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/info/
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/info/exclude  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/info/refs  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/refs/
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/refs/heads/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/refs/heads/master  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/refs/tags/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/HEAD  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/config  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/pack/
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/info/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/info/packs  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/2f/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/2f/34f5a52760c27e6178ea424e3ae1e27e0445ad  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/f9/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/f9/3e3a1a1525fb5b91020da86e44810c87a2d7bc  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/e6/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/7f/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/7f/f40a2d88161b99efd90e79e2caf5a2757bbd09  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/12/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/12/fb2da73ab1b3e401d7b944b3732d6eb026bd32  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/b5/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/b5/ffa97c0d9dd637ddb5b5c90feff746208b908a  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/60/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/60/c56ca3f3daa0d19d5c359c24f6922e5091f45a  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/49/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/49/2986ff316380b1986d7772b664410cc3065c42  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/bf/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/bf/dc113cef87a41574f8d7ca8074234f31dc6d5f  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/a1/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/a1/7039c3e49e7a03d091eccc73ecb1fe5192498e  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/31/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/31/41fa7691e733278cdfc16f49ebc18eddfd756a  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/45/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/45/e490df1c068d16eb55c06c443e19788f4a0151  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/9d/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/9d/aeafb9864cf43055ae93beb0afd6c7d144bfa4  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/7a/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/7a/92d1730677dc911a9a98b465c80568efcef446  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/d1/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/d1/3a5448f2f4f56e5bb0708fb5e2f1c417e08831  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/4d/
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/4d/7f0d8a5c679da4dc815a7d91af57fc39849683  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/8b/
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/8b/f2a1de43e0bbe10f812edbc999091fff08a849  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/39/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/39/c96b920a3ec094f5143837fe8046b1a5b92896  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/d2/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/d2/1de79d1d8ca1b5a9417b80abbda14901753ce1  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/a0/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/a0/e11093d182807ae80c18ca52e8634b35eeff4f  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/7b/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/7b/ae1d38a02b028a19aab91a63a57f63cd70abd0  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/9e/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/9e/d170be79d436ebd67d2edfa8d2b047cf256920  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/92/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/92/956413b4f750492cf572ed575a523f5cbc20cd  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/69/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/69/6e516106c63e24a3a09ce9e1969291401392b6  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/4a/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/4a/3867048d2409c447fdede33efb8e5f50ea8a20  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/16/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/16/ff985207c5051fba0d69c7fb6f3f1f5cc8a40c  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/35/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/35/51367b5cfefe1508042679cdec728d0ab5cc11  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/95/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/95/c3903ef21da3c180330812d445dfcfd24a929e  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/62/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/assets.git/objects/62/560add7af41d82c638d67c5d5e641b1e0e6553  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/branches/
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/description  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/applypatch-msg.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/commit-msg.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/post-update.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/pre-applypatch.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/pre-commit.sample  
    default: 
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/pre-push.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/pre-rebase.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/pre-receive.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/prepare-commit-msg.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/update.sample  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/pre-receive  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/update  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/hooks/post-receive  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/info/
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/info/exclude  
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/info/refs  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/refs/
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/refs/heads/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/refs/heads/master  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/refs/tags/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/HEAD  
    default:   inflating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/config  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/pack/
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/info/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/info/packs  
    default: 
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/7d/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/7d/13c9b455b959082f76861b3a87171f87c3504e  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/56/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/56/1b864d5bac6f3394c7dc12e10d0d02a510fd16  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/81/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/81/8cd32be579a6b2c7bd92f487326f5fda025a17  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/75/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/75/37a76cca4a077df440746686b8a918c25d3826  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/bf/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/bf/6d94292a7689dec6b3991e3e718a3b9449e8af  
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/bf/8b87e0a8d2aefbb22b44d02a13456ddf779590  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/b1/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/b1/20d9b3ca7009feb9103aaa50f0a7dba6e71265  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/53/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/53/e1144facaa9b3c641e5ed9a9a3c8cb26102e96  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/8e/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/8e/7a1dcd59c2e2f8c9685bf0e630c2e2d500027f  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/92/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/92/aa507bbeae6788d82be804ec24cf1601aae6a8  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/b6/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/b6/9f26b3cee3637184d556a623814b540b0df127  
    default:    creating: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/20/
    default:  extracting: /var/lib/gogs/git/gogs-repositories/root/dynamic_web.git/objects/20/e8167e3b03c16c8e92dfa35e5d3167c16dba87  
    default:   inflating: /var/lib/gogs/git/.gitconfig  
    default:    creating: /var/lib/gogs/ssh/
    default:   inflating: /var/lib/gogs/ssh/ssh_host_rsa_key  
    default:   inflating: /var/lib/gogs/ssh/ssh_host_rsa_key.pub  
    default:   inflating: /var/lib/gogs/ssh/ssh_host_dsa_key  
    default:   inflating: /var/lib/gogs/ssh/ssh_host_dsa_key.pub  
    default:   inflating: /var/lib/gogs/ssh/ssh_host_ecdsa_key  
    default:   inflating: /var/lib/gogs/ssh/ssh_host_ecdsa_key.pub  
    default:   inflating: /var/lib/gogs/ssh/ssh_host_ed25519_key  
    default:   inflating: /var/lib/gogs/ssh/ssh_host_ed25519_key.pub  
    default: d5f129fe2e9606f6784db59539346e3ee22416768ad37b4b14a0984cf9118db8
    default: env init finished!
==> default: Running provisioner: shell...
    default: Running: inline script
    default: * Stopping "minikube" in none ...
    default: * "minikube" stopped.
    default: * minikube v1.2.0 on linux (amd64)
    default: * using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
    default: * Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
    default: * Restarting existing none VM for "minikube" ...
    default: * Waiting for SSH access ...
    default: * Configuring environment for Kubernetes v1.15.0 on Docker 19.03.1
    default: * Relaunching Kubernetes v1.15.0 using kubeadm ... 
    default: * Configuring local host environment ...
    default: 
    default:   - https://github.com/kubernetes/minikube/blob/master/docs/vmdriver-none.md
    default: ! The 'none' driver provides limited isolation and may reduce system security and reliability.
    default: ! For more information, see:
    default: ! kubectl and minikube configuration will be stored in /root
    default: ! To use kubectl or minikube commands as your own user, you may
    default: 
    default:   - sudo mv /root/.kube /root/.minikube $HOME
    default:   - sudo chown -R $USER $HOME/.kube $HOME/.minikube
    default: 
    default: * This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
    default: ! need to relocate them. For example, to overwrite your own settings:
    default: * Verifying:
    default:  apiserver
    default:  proxy
    default:  etcd
    default:  scheduler
    default:  controller
    default:  dns
    default: 
    default: * Done! kubectl is now configured to use "minikube"
==> default: Running provisioner: shell...
    default: Running: inline script
    default: gogs
    default: gogs is up, follow this link to access the web:
    default: http://10.1.0.165:10080
    default: default password:
    default: root/root
==> default: Running provisioner: shell...
    default: Running: inline script
    default: env is up, follow this link to access the web:
    default: http://10.1.0.155:8080
    default: default password:
    default: admin/admin

real	11m55.325s
user	0m31.427s
sys	0m5.126s
nothing@pc:~/vagrant/cicd$ 
~~~

不得不说，从之前半个多小时，现在下降到12分钟不到，提速还是很明显的


---

# 总结


~~~
git clone https://github.com/wilmosfang/cicd.git
cd cicd
vagrant up
~~~


* TOC
{:toc}

---


[virtualbox]:https://www.virtualbox.org/
[minikube]:https://minikube.sigs.k8s.io/
[vagrant]:https://www.vagrantup.com/
[gogs]:https://gogs.io/


