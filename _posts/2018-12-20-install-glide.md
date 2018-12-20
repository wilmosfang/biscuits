---
layout: post
title: "Install Glide"
date: 2018-12-20 16:56:26
image: '/assets/img/'
description:  '安装 Glide'
main-class: 'tools'
color: 
tags: 
  - tools
  - glide
categories:
  - tools
twitter_text: 'simple process of Glide installation'
introduction: 'installation of Glide'
---

## 前言

**[Glide][glide]** 是一个 Go 的包管理工具

>Package Management for Go

这里分享一下 **[Glide][glide]** 的安装方法

参考 **[Glide 官方文档][glide]**


> **Tip:** 当前的版本为 **v0.13.2**

---

# 操作


## 环境

~~~
[vagrant@go ~]$ hostnamectl 
   Static hostname: go
         Icon name: computer-vm
           Chassis: vm
        Machine ID: b6191cc6682d4a308ad289f7f26c4849
           Boot ID: 892566cc1ce94522881e2b28028bede6
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.1.3.el7.x86_64
      Architecture: x86-64
[vagrant@go ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:84:81:d5 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84047sec preferred_lft 84047sec
    inet6 fe80::5054:ff:fe84:81d5/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:4f:40:e0 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.101/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe4f:40e0/64 scope link 
       valid_lft forever preferred_lft forever
[vagrant@go ~]$ 
~~~


## 安装软件

获取最新版本

>Get the latest release of Glide. The script puts it with your Go binaries ($GOPATH/bin or $GOBIN).

~~~
[root@go ~]# curl https://glide.sh/get | sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4833  100  4833    0     0   2671      0  0:00:01  0:00:01 --:--:--  2671
ARCH=amd64
OS=linux
Using curl as download tool
Getting https://glide.sh/version
TAG=v0.13.2
GLIDE_DIST=glide-v0.13.2-linux-amd64.tar.gz
Downloading https://github.com/Masterminds/glide/releases/download/v0.13.2/glide-v0.13.2-linux-amd64.tar.gz
glide version v0.13.2 installed successfully
[root@go ~]# 
~~~


初始化和创建配置文档

~~~
glide init
edit glide.yaml
~~~



## 解决依赖

~~~
[root@go ~]# cd /go/golang_learning/src/gw/
[root@go gw]# pwd
/go/golang_learning/src/gw
[root@go gw]# ls
conf        glide.lock  handler  main.go  README.md
controller  glide.yaml  log      model
[root@go gw]# pwd
/go/golang_learning/src/gw
[root@go gw]# ls
conf        glide.lock  handler  main.go  README.md
controller  glide.yaml  log      model
[root@go gw]# cat glide.yaml 
package: gw
import:
- package: github.com/appleboy/gin-jwt
  version: ~2.5.0
- package: github.com/casbin/casbin
  version: ~1.6.0
- package: github.com/casbin/gorm-adapter
- package: github.com/gin-gonic/gin
  version: ~1.3.0
- package: github.com/jinzhu/gorm
  version: ~1.9.1
  subpackages:
  - dialects/postgres
- package: github.com/spf13/viper
  version: ~1.1.0
[root@go gw]#
[root@go gw]# glide update 
[INFO]	Downloading dependencies. Please wait...
[INFO]	--> Fetching updates for github.com/appleboy/gin-jwt
[INFO]	--> Fetching updates for github.com/casbin/casbin
[INFO]	--> Fetching updates for github.com/casbin/gorm-adapter
[INFO]	--> Fetching updates for github.com/gin-gonic/gin
[INFO]	--> Fetching updates for github.com/jinzhu/gorm
[INFO]	--> Fetching updates for github.com/spf13/viper
[INFO]	--> Detected semantic version. Setting version for github.com/appleboy/gin-jwt to v2.5.0
[INFO]	--> Detected semantic version. Setting version for github.com/casbin/casbin to v1.6.0
[INFO]	--> Detected semantic version. Setting version for github.com/gin-gonic/gin to v1.3.0
[INFO]	--> Detected semantic version. Setting version for github.com/jinzhu/gorm to v1.9.2
[INFO]	--> Detected semantic version. Setting version for github.com/spf13/viper to v1.1.1
[INFO]	Resolving imports
[INFO]	--> Fetching updates for gopkg.in/dgrijalva/jwt-go.v3
[INFO]	--> Fetching updates for github.com/Knetic/govaluate
[INFO]	--> Fetching updates for github.com/lib/pq
[INFO]	--> Fetching updates for github.com/gin-contrib/sse
[INFO]	--> Fetching updates for github.com/mattn/go-isatty
[INFO]	--> Fetching updates for github.com/jinzhu/inflection
[INFO]	--> Fetching updates for github.com/fsnotify/fsnotify
[INFO]	--> Fetching updates for github.com/hashicorp/hcl
[INFO]	--> Fetching updates for github.com/magiconair/properties
[INFO]	--> Fetching updates for github.com/mitchellh/mapstructure
[INFO]	--> Fetching updates for github.com/pelletier/go-toml
[INFO]	--> Fetching updates for github.com/spf13/afero
[INFO]	--> Fetching updates for github.com/spf13/cast
[INFO]	--> Fetching updates for github.com/spf13/jwalterweatherman
[INFO]	--> Fetching updates for github.com/spf13/pflag
[INFO]	--> Fetching updates for gopkg.in/yaml.v2
[INFO]	--> Fetching updates for github.com/pkg/errors
[INFO]	--> Fetching updates for github.com/golang/protobuf
[INFO]	--> Fetching updates for github.com/ugorji/go
[INFO]	--> Fetching updates for gopkg.in/go-playground/validator.v8
[INFO]	--> Fetching updates for github.com/json-iterator/go
[INFO]	--> Fetching updates for golang.org/x/sys
[INFO]	--> Fetching updates for golang.org/x/text
[INFO]	--> Fetching updates for github.com/modern-go/concurrent
[INFO]	--> Fetching updates for github.com/modern-go/reflect2
[INFO]	Downloading dependencies. Please wait...
[INFO]	Setting references for remaining imports
[INFO]	Exporting resolved dependencies...
[INFO]	--> Exporting github.com/appleboy/gin-jwt
[INFO]	--> Exporting github.com/casbin/casbin
[INFO]	--> Exporting github.com/casbin/gorm-adapter
[INFO]	--> Exporting github.com/gin-gonic/gin
[INFO]	--> Exporting github.com/jinzhu/gorm
[INFO]	--> Exporting github.com/spf13/viper
[INFO]	--> Exporting github.com/Knetic/govaluate
[INFO]	--> Exporting github.com/lib/pq
[INFO]	--> Exporting github.com/mattn/go-isatty
[INFO]	--> Exporting github.com/jinzhu/inflection
[INFO]	--> Exporting github.com/fsnotify/fsnotify
[INFO]	--> Exporting github.com/hashicorp/hcl
[INFO]	--> Exporting github.com/magiconair/properties
[INFO]	--> Exporting github.com/mitchellh/mapstructure
[INFO]	--> Exporting github.com/pelletier/go-toml
[INFO]	--> Exporting github.com/spf13/afero
[INFO]	--> Exporting github.com/spf13/cast
[INFO]	--> Exporting github.com/spf13/jwalterweatherman
[INFO]	--> Exporting github.com/spf13/pflag
[INFO]	--> Exporting github.com/pkg/errors
[INFO]	--> Exporting github.com/golang/protobuf
[INFO]	--> Exporting github.com/ugorji/go
[INFO]	--> Exporting github.com/gin-contrib/sse
[INFO]	--> Exporting github.com/json-iterator/go
[INFO]	--> Exporting github.com/modern-go/concurrent
[INFO]	--> Exporting github.com/modern-go/reflect2
[INFO]	--> Exporting gopkg.in/dgrijalva/jwt-go.v3
[INFO]	--> Exporting gopkg.in/yaml.v2
[INFO]	--> Exporting golang.org/x/sys
[INFO]	--> Exporting golang.org/x/text
[INFO]	--> Exporting gopkg.in/go-playground/validator.v8
[INFO]	Replacing existing vendor dependencies
[INFO]	Project relies on 31 dependencies.
[root@go gw]# echo $?
0
[root@go gw]# 
~~~

## 安装依赖

~~~
[root@go gw]# glide install
[INFO]	Downloading dependencies. Please wait...
[INFO]	--> Found desired version locally github.com/appleboy/gin-jwt 18c777042be3ecc5cb3bf79cd24cbd6b28683cdc!
[INFO]	--> Found desired version locally github.com/casbin/casbin 695c374a6617e9cdc930e8666307a772577cbd70!
[INFO]	--> Found desired version locally github.com/casbin/gorm-adapter 3c6ecd6f277a6513aedee06cb97b030267d4782e!
[INFO]	--> Found desired version locally github.com/fsnotify/fsnotify ccc981bf80385c528a65fbfdd49bf2d8da22aa23!
[INFO]	--> Found desired version locally github.com/gin-contrib/sse 22d885f9ecc78bf4ee5d72b937e4bbcdc58e8cae!
[INFO]	--> Found desired version locally github.com/gin-gonic/gin b869fe1415e4b9eb52f247441830d502aece2d4d!
[INFO]	--> Found desired version locally github.com/golang/protobuf 1d3f30b51784bec5aad268e59fd3c2fc1c2fe73f!
[INFO]	--> Found desired version locally github.com/hashicorp/hcl 65a6292f0157eff210d03ed1bf6c59b190b8b906!
[INFO]	--> Found desired version locally github.com/jinzhu/gorm 472c70caa40267cb89fd8facb07fe6454b578626!
[INFO]	--> Found desired version locally github.com/jinzhu/inflection 04140366298a54a039076d798123ffa108fff46c!
[INFO]	--> Found desired version locally github.com/json-iterator/go d05f387f50c0c406889b62cb5ec51c76a64d89ee!
[INFO]	--> Found desired version locally github.com/Knetic/govaluate 9aa49832a739dcd78a5542ff189fb82c3e423116!
[INFO]	--> Found desired version locally github.com/lib/pq 9eb73efc1fcc404148b56765b0d3f61d9a5ef8ee!
[INFO]	--> Found desired version locally github.com/magiconair/properties 7c38529aac7222391b7a2661365177a97e21b998!
[INFO]	--> Found desired version locally github.com/mattn/go-isatty 3fb116b820352b7f0c281308a4d6250c22d94e27!
[INFO]	--> Found desired version locally github.com/mitchellh/mapstructure 3536a929edddb9a5b34bd6861dc4a9647cb459fe!
[INFO]	--> Found desired version locally github.com/modern-go/concurrent bacd9c7ef1dd9b15be4a9909b8ac7a4e313eec94!
[INFO]	--> Found desired version locally github.com/modern-go/reflect2 94122c33edd36123c84d5368cfb2b69df93a0ec8!
[INFO]	--> Found desired version locally github.com/pelletier/go-toml 27c6b39a135b7dc87a14afb068809132fb7a9a8f!
[INFO]	--> Found desired version locally github.com/pkg/errors 059132a15dd08d6704c67711dae0cf35ab991756!
[INFO]	--> Found desired version locally github.com/spf13/afero a5d6946387efe7d64d09dcba68cdd523dc1273a3!
[INFO]	--> Found desired version locally github.com/spf13/cast 8c9545af88b134710ab1cd196795e7f2388358d7!
[INFO]	--> Found desired version locally github.com/spf13/jwalterweatherman 94f6ae3ed3bceceafa716478c5fbf8d29ca601a1!
[INFO]	--> Found desired version locally github.com/spf13/pflag 916c5bf2d89aff6fd3e10e7811337218dfa81cb5!
[INFO]	--> Found desired version locally github.com/spf13/viper f69c91abd383f6e61b17df94c1020943eefe0349!
[INFO]	--> Found desired version locally github.com/ugorji/go 772ced7fd4c2f6322c07537a9a93b68d74551fa6!
[INFO]	--> Found desired version locally golang.org/x/sys 074acd46bca67915925527c07849494d115e7c43!
[INFO]	--> Found desired version locally golang.org/x/text 17bcc049122f272a32787ba38073ee47433023e9!
[INFO]	--> Found desired version locally gopkg.in/dgrijalva/jwt-go.v3 06ea1031745cb8b3dab3f6a236daf2b0aa468b7e!
[INFO]	--> Found desired version locally gopkg.in/go-playground/validator.v8 5f1438d3fca68893a817e4a66806cea46a9e4ebf!
[INFO]	--> Found desired version locally gopkg.in/yaml.v2 51d6538a90f86fe93ac480b35f37b2be17fef232!
[INFO]	Setting references.
[INFO]	--> Setting version for github.com/appleboy/gin-jwt to 18c777042be3ecc5cb3bf79cd24cbd6b28683cdc.
[INFO]	--> Setting version for github.com/casbin/casbin to 695c374a6617e9cdc930e8666307a772577cbd70.
[INFO]	--> Setting version for github.com/casbin/gorm-adapter to 3c6ecd6f277a6513aedee06cb97b030267d4782e.
[INFO]	--> Setting version for github.com/fsnotify/fsnotify to ccc981bf80385c528a65fbfdd49bf2d8da22aa23.
[INFO]	--> Setting version for github.com/gin-contrib/sse to 22d885f9ecc78bf4ee5d72b937e4bbcdc58e8cae.
[INFO]	--> Setting version for github.com/gin-gonic/gin to b869fe1415e4b9eb52f247441830d502aece2d4d.
[INFO]	--> Setting version for github.com/golang/protobuf to 1d3f30b51784bec5aad268e59fd3c2fc1c2fe73f.
[INFO]	--> Setting version for github.com/hashicorp/hcl to 65a6292f0157eff210d03ed1bf6c59b190b8b906.
[INFO]	--> Setting version for github.com/jinzhu/gorm to 472c70caa40267cb89fd8facb07fe6454b578626.
[INFO]	--> Setting version for github.com/jinzhu/inflection to 04140366298a54a039076d798123ffa108fff46c.
[INFO]	--> Setting version for github.com/json-iterator/go to d05f387f50c0c406889b62cb5ec51c76a64d89ee.
[INFO]	--> Setting version for github.com/Knetic/govaluate to 9aa49832a739dcd78a5542ff189fb82c3e423116.
[INFO]	--> Setting version for github.com/lib/pq to 9eb73efc1fcc404148b56765b0d3f61d9a5ef8ee.
[INFO]	--> Setting version for github.com/magiconair/properties to 7c38529aac7222391b7a2661365177a97e21b998.
[INFO]	--> Setting version for github.com/mattn/go-isatty to 3fb116b820352b7f0c281308a4d6250c22d94e27.
[INFO]	--> Setting version for github.com/mitchellh/mapstructure to 3536a929edddb9a5b34bd6861dc4a9647cb459fe.
[INFO]	--> Setting version for github.com/modern-go/concurrent to bacd9c7ef1dd9b15be4a9909b8ac7a4e313eec94.
[INFO]	--> Setting version for github.com/pelletier/go-toml to 27c6b39a135b7dc87a14afb068809132fb7a9a8f.
[INFO]	--> Setting version for github.com/pkg/errors to 059132a15dd08d6704c67711dae0cf35ab991756.
[INFO]	--> Setting version for github.com/modern-go/reflect2 to 94122c33edd36123c84d5368cfb2b69df93a0ec8.
[INFO]	--> Setting version for github.com/spf13/afero to a5d6946387efe7d64d09dcba68cdd523dc1273a3.
[INFO]	--> Setting version for github.com/spf13/cast to 8c9545af88b134710ab1cd196795e7f2388358d7.
[INFO]	--> Setting version for github.com/spf13/viper to f69c91abd383f6e61b17df94c1020943eefe0349.
[INFO]	--> Setting version for github.com/spf13/jwalterweatherman to 94f6ae3ed3bceceafa716478c5fbf8d29ca601a1.
[INFO]	--> Setting version for github.com/spf13/pflag to 916c5bf2d89aff6fd3e10e7811337218dfa81cb5.
[INFO]	--> Setting version for github.com/ugorji/go to 772ced7fd4c2f6322c07537a9a93b68d74551fa6.
[INFO]	--> Setting version for golang.org/x/text to 17bcc049122f272a32787ba38073ee47433023e9.
[INFO]	--> Setting version for golang.org/x/sys to 074acd46bca67915925527c07849494d115e7c43.
[INFO]	--> Setting version for gopkg.in/dgrijalva/jwt-go.v3 to 06ea1031745cb8b3dab3f6a236daf2b0aa468b7e.
[INFO]	--> Setting version for gopkg.in/go-playground/validator.v8 to 5f1438d3fca68893a817e4a66806cea46a9e4ebf.
[INFO]	--> Setting version for gopkg.in/yaml.v2 to 51d6538a90f86fe93ac480b35f37b2be17fef232.
[INFO]	Exporting resolved dependencies...
[INFO]	--> Exporting github.com/appleboy/gin-jwt
[INFO]	--> Exporting github.com/casbin/casbin
[INFO]	--> Exporting github.com/casbin/gorm-adapter
[INFO]	--> Exporting github.com/fsnotify/fsnotify
[INFO]	--> Exporting github.com/gin-contrib/sse
[INFO]	--> Exporting github.com/gin-gonic/gin
[INFO]	--> Exporting github.com/golang/protobuf
[INFO]	--> Exporting github.com/hashicorp/hcl
[INFO]	--> Exporting github.com/jinzhu/gorm
[INFO]	--> Exporting github.com/jinzhu/inflection
[INFO]	--> Exporting github.com/json-iterator/go
[INFO]	--> Exporting github.com/Knetic/govaluate
[INFO]	--> Exporting github.com/lib/pq
[INFO]	--> Exporting github.com/magiconair/properties
[INFO]	--> Exporting github.com/mattn/go-isatty
[INFO]	--> Exporting github.com/mitchellh/mapstructure
[INFO]	--> Exporting github.com/modern-go/concurrent
[INFO]	--> Exporting github.com/modern-go/reflect2
[INFO]	--> Exporting github.com/pelletier/go-toml
[INFO]	--> Exporting github.com/pkg/errors
[INFO]	--> Exporting github.com/spf13/afero
[INFO]	--> Exporting github.com/spf13/cast
[INFO]	--> Exporting github.com/spf13/jwalterweatherman
[INFO]	--> Exporting github.com/spf13/pflag
[INFO]	--> Exporting github.com/spf13/viper
[INFO]	--> Exporting github.com/ugorji/go
[INFO]	--> Exporting gopkg.in/dgrijalva/jwt-go.v3
[INFO]	--> Exporting golang.org/x/text
[INFO]	--> Exporting golang.org/x/sys
[INFO]	--> Exporting gopkg.in/yaml.v2
[INFO]	--> Exporting gopkg.in/go-playground/validator.v8
[INFO]	Replacing existing vendor dependencies
[root@go gw]# echo $?
0
[root@go gw]# 
~~~

## 获取额外依赖包

~~~
glide get github.com/foo/bar
glide get github.com/foo/bar#^1.2.3
~~~

## 运行检测

~~~
[root@go gw]# go run main.go 
Successfully connected!
Model:
r.r: sub, obj, act
p.p: sub, obj, act
e.e: some(where (p_eft == allow))
m.m: ((r_sub == p_sub) || g(r_sub,p_sub))  && ((r_obj == p_obj) || g(r_obj,p_obj)) && ((r_act == p_act) || g(r_act,p_act))
g.g: _, _
Policy:
p: sub, obj, act: []
g: _, _: []
Role links for: g

Policy:
p: sub, obj, act: []
g: _, _: []
Role links for: g

[GIN-debug] [WARNING] Now Gin requires Go 1.6 or later and Go 1.7 will be required soon.

[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /login                    --> gw/vendor/github.com/appleboy/gin-jwt.(*GinJWTMiddleware).LoginHandler-fm (3 handlers)
[GIN-debug] GET    /auth/hello               --> gw/handler.H_hello (4 handlers)
[GIN-debug] GET    /auth/refresh_token       --> gw/vendor/github.com/appleboy/gin-jwt.(*GinJWTMiddleware).RefreshHandler-fm (4 handlers)
[GIN-debug] GET    /auth/ping                --> gw/handler.H_ping (4 handlers)
[GIN-debug] Listening and serving HTTP on :9999
...
...
...
~~~

---

# 总结

由于 **[Glide][glide]**  准备好了相应的 脚本，所以整个过程就是一个执行脚本的过程

关于  **[Glide][glide]** 的高级用法，后面有需要，再进行展开

* TOC
{:toc}

---

[glide]:https://glide.sh/