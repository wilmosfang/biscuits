---
layout: post
title: "Simple CI/CD with poll SCM of Jenkins"
date: 2018-01-20 14:19:06
image: '/assets/img/'
description: '使用Jenkins的pollSCM实现简单的CI/CD'
main-class: 'jenkins'
color: '#1077ad'
tags:
 - jenkins
 - ci
 - cd
categories:
 - jenkins
twitter_text: 'CI/CD with pollSCM of Jenkins'
introduction: 'Simple CI/CD method'
---


# 前言


**[Jenkins][jenkins]** 是一套自动化软件，结合不同的插件可以轻易实现 **CI/CD** 工作流

**[Jenkins][jenkins]** 与 **k8s** 还有 **Gitlab** 常常放在一起构建持续集成系统

下面分享一下 **[Jenkins][jenkins]** 构建 **CI/CD** 流的简单实现


> **Tip:** 当前版本 **Jenkins 2.89.3 LTS**

---

# 操作

## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: f91fb2eedcd345cf9cd7e6d80aab5115
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d1:5d:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 83782sec preferred_lft 83782sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:47:20:56 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.208/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]#
~~~


## 创建密钥对


**[HomePage]->[Credentials]->[System]->[Global credentials (unrestricted)]->[Add Credentials]**

![jenkins](/assets/img/jenkins/jenkins8.png)

这个密钥对的作用是用来登录目标服务器

代码最终要更新到此服务器中，WEB服务在此服务器中运行

**Username** 和 **Password** 必须手动指定，即为登录账号与密码

**Description** 可以不填，只是为了识别

**ID** 可以不填，会自动生成

## 添加SSH远程主机

**[HomePage]->[Manage Jenkins]->[Configure System]->[SSH remote hosts]->[Add]**

![jenkins](/assets/img/jenkins/jenkins9.png)

配置完成后点击 **[Check connection]** 进行连接测试

**Successfull connection**　表明可以正常联通

**Hostname** 指定远程主机 IP 或主机名，必须网络可达

**Port** 指定远程的 SSH 端口

**Credentials** 选择上一步中设定的密钥对

其它保持默认，这样就配置好了一个远程主机


## 创建项目

![jenkins](/assets/img/jenkins/jenkins10.png)

**[HomePage]->[New Item]->[Freestyle Project]->[OK]**

**Enter an item name** 下输入项目名


## 配置SCM

**SCM** 是 **Source Code Management** 的缩写

![jenkins](/assets/img/jenkins/jenkins11.png)

选择 **Git** (因为我的项目在GitHub上)

然后指定正确的 **Repository URL** 和　**Branch Specifier (blank for 'any')** 分支 (因为我的Web只发布于 gh-pages,所以我只需要让其检查此分支的变化就可以了)

## 配置触发器

**Build Triggers**

![jenkins](/assets/img/jenkins/jenkins12.png)

这里为了简便，就使用了 **Poll SCM**

**`H/2  * * * *`** 代表每两分钟检查一次

编辑框下面会提示下一次执行检查的时间

### Poll SCM 与 Build periodically 区别

**Build periodically** 也会要求输入调动周期

那 **Poll SCM** 和它有什么区别呢

两者都会周期性地调动，但是 **Poll SCM** 只在检查到源码版本有变化的时候才会执行后面的 build 操作，而 **Build periodically** 是不论源码版本是否有变化都会执行后面的 build 操作

如果源代码在公网平台上（比如github），那这两者与其它触发机制有什么不同呢

这两者由于是主动发起的，所以可以没有公网IP而隐藏在 NAT 后面，只要有可以主动访问公网的权限就可以，而其它方式(比如 GitHub hook trigger for GITScm polling),就需要提供一个公网 IP 或从公网 IP 到此 Jenkins 服务端口的 DNAT 映射,无疑后者的环境要求要高一些，但是前者的开销要大一点，因为毕竟事件触发的响应式模型更加有效和节省系统资源

## 配置执行内容

**Build** 作为整个构建过程中最核心的一步，里面定义了所有要作的操作内容

这里选择 **Excuete shell scrip on remote host using ssh**

**SSH site** 中选择在系统配置里设定好的 host

**Command** 中定义脚本内容

由于我是使用的 jekyll 来构建 web 所以可以动态发布，并没额外的build步骤，这一步由jekyll代劳了，我只需要更新发布代码就可以了

~~~
cd  /home/git/git/biscuits/
git pull
echo `date` > /tmp/date
cat /tmp/date
~~~

上面两步是进入代码根目录，下拉最新代码到本地，下面两步是记录一个更新时间戳


## 提交变更触发发布

~~~

~~~



---

# 总结

因为 Jenkins 被打成了一个 war 包，并且所有的配置与代码都进行了解耦，所以升级过程特别简单

简而言之就是只用替换掉 war 包，其它都不变

* TOC
{:toc}


---

[jenkins]:https://jenkins.io/
