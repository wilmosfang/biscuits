---
layout: post
title: "Auto Init Docker env by Vagrant"
date: 2019-09-10 15:37:42
image: '/assets/img/'
description: '自动初始化 Docker 环境'
main-class: 'code'
color: '#01f615' 
tags:
 - snippet
 - vagrant
 - linux
categories:
 - code
twitter_text: 'Auto Init Docker env by Vagrant'
introduction: 'Auto Init Docker env by Vagrant'
---

# 前言

开启一个 **CODE** 系列，用来收集各种好用的小片段

这一篇用来展示通过 Vagrant 和 VirtualBox 自动初始化一个可用的 Docker 环境

> **Tip:** 展示中使用的各种软件版本可能不是最新，但我都会尽量提前交代

---

# 操作

## 依赖

* Vagrant 2.2.9

~~~
➜  blog vagrant --version
Vagrant 2.2.9
➜  blog
~~~

* VirtualBox 6.0.16

~~~
VirtualBox Graphical User Interface
Version 6.0.16 r135674 (Qt5.6.3)
~~~

## OS 环境

~~~
➜  blog sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.4
BuildVersion:	19E287
➜  blog
~~~





这样 **WordPress** 就安装成功了

---

# 总结

WordPress 是一个经典的 LAMP 应用 

所以在 Linux Apache Mysql PHP 等环境准备好了的情况下只需要将相应的软件包解压并放在合适的位置就可以了

不得不说当前的 wordpress 版本颜值极高，插件支持，媒体库的托拽，编辑窗口的易用，内容版本差异的管理，评论支持，主题更换，界面的自定义都有着十分友好的接口

一个开源内容发布系统，真的很喜欢

* TOC
{:toc}

---

[wordpress]:https://wordpress.org/
[wordpress_dep]:https://wordpress.org/about/requirements/
[wordpress_install]:https://codex.wordpress.org/Installing_WordPress
[mariadb_repo]:https://downloads.mariadb.org/mariadb/repositories/#distro=CentOS&distro_release=centos7-amd64--centos7&mirror=liteserver&version=10.4
