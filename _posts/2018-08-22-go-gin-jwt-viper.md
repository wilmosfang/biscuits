---
layout: post
title: "Gin Viper"
date: 2018-08-22 14:38:44
image: '/assets/img/'
description: '使用 Viper 来进行配置管理'
main-class:  'go'
color: '#68d6e3'
tags:
 - go
 - gin
 - jwt
 - viper
categories: 
 - go
twitter_text: 'Web Config management by Viper'
introduction: 'Web Config management by Viper'
---


# 前言 #


**[Gin][gin]** 是一款用 Go(Golang) 编写的 web 框架

>Gin is a web framework written in Go (Golang). It features a martini-like API with much better performance, up to 40 times faster thanks to httprouter

因为 httprouter, 它提供了更高的性能

配置管理是一个 Web 应用非常关键的环节

这里使用 **[Viper][viper]** 来实现

什么是 **Viper** 的文档可以参考 **[package viper][viper_doc]** 

这里演示一下在 **[Gin][gin]** 中如何进行配置管理

gin 的 API 可以参考 **[API REFFERENCE][gin_api_doc]**

> **Tip:** 当前的版本为 **Gin 1.2** 和 **Go 1.10** (但是实验环境下，Go没有使用最新的版本)

---

# 操作 #

## 系统环境 ##

~~~
[root@h160 ~]# hostnamectl 
   Static hostname: h160
         Icon name: computer-vm
           Chassis: vm
        Machine ID: d46f9440d4be429ea66b726977adf233
           Boot ID: a0fc6a1b4f124e39a27866d4df0701fa
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.2.3.el7.x86_64
      Architecture: x86-64
[root@h160 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84363sec preferred_lft 84363sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:48:f4:2c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.160/24 brd 192.168.56.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe48:f42c/64 scope link 
       valid_lft forever preferred_lft forever
[root@h160 ~]# go version
go version go1.9.4 linux/amd64
[root@h160 ~]#
~~~

## 进入 GOPATH ##

~~~
[vagrant@h160 ~]$ echo $GOPATH
/vagrant/go
[vagrant@h160 ~]$ cd $GOPATH
[vagrant@h160 go]$ pwd
/vagrant/go
[vagrant@h160 go]$ 
~~~

## 代码注解 ##

~~~go
package main //指明此包为 main 包

import (
	//Package bytes implements functions for the manipulation of byte slices. It is analogous to the facilities of the strings package
	//这个包用来提供对 byte slices 的操作，有点类似于 string 的作用
	"bytes"
	//Package time provides functionality for measuring and displaying time
	//The calendrical calculations always assume a Gregorian calendar, with no leap seconds
	//用来提供一些计算和显示时间的函数
	"time"
	//JWT Middleware for Gin framework
	//一个 Gin 的 JWT 中间件
	"github.com/appleboy/gin-jwt"
	//Package gin implements a HTTP web framework called gin
	//这个包用来实现一个 HTTP 的 web 框架
	"github.com/gin-gonic/gin"
	//Viper is a complete configuration solution for Go applications including 12-Factor apps. It is designed to work within an application, and can handle all types of configuration needs and formats
	//viper 是一个非常强大的用来进行配置管理的包
	"github.com/spf13/viper"
)

//定义了一个配置函数，用来设定初始配置
//这个函数接收一个 Viper 的指针，然后对这个 Viper 结构进行配置
func confViper(v *viper.Viper) {
	//func SetConfigType(in string)
	//SetConfigType sets the type of the configuration returned by the remote source, e.g. "json".
	//这里我使用 toml 的格式来填充配置
	v.SetConfigType("toml")

	//定义一个 byte 的数组，用来存储配置
	//这种方式是直接把配置写到内存中
	//在测试环境下和配置比较少的情况下，可以直接使用这种方式来快速实现
	var tomlConf = []byte(`
	#port='9999'
	#realm='zone name just for test'
	key='secret key salt'
	tokenLookup='header: Authorization, query: token, cookie: jwt'
	tokenHeadName='Bearer'
	#loginPath='/login'
	authPath='/auth'
	refreshPath='/refresh_token'
	testPath='/hello'
	`)
	//func ReadConfig(in io.Reader) error
	//ReadConfig will read a configuration file, setting existing keys to nil if the key does not exist in the file.
	//用来从上面的byte数组中读取配置内容
	v.ReadConfig(bytes.NewBuffer(tomlConf))

	//配置默认值，如果配置内容中没有指定，就使用以下值来作为配置值
	v.SetDefault("port", "8000")
	v.SetDefault("realm", "testzone")
	v.SetDefault("key", "secret")
	v.SetDefault("tokenLookup", "header: Authorization, query: token, cookie: jwt")
	v.SetDefault("tokenHeadName", "Bearer")
	v.SetDefault("loginPath", "/login")
	v.SetDefault("authPath", "/auth")
	v.SetDefault("refreshPath", "/refresh_token")
	v.SetDefault("testPath", "/hello")
}

//定义一函数用来处理请求
func helloHandler(c *gin.Context) {
	//func ExtractClaims(c *gin.Context) jwt.MapClaims
	//ExtractClaims help to extract the JWT claims
	//用来将 Context 中的数据解析出来赋值给 claims
	//其实是解析出来了 JWT_PAYLOAD 里的内容
	/*
		func ExtractClaims(c *gin.Context) jwt.MapClaims {
		claims, exists := c.Get("JWT_PAYLOAD")
		if !exists {
			return make(jwt.MapClaims)
		}

		return claims.(jwt.MapClaims)
		}
	*/
	claims := jwt.ExtractClaims(c)
	//func (c *Context) JSON(code int, obj interface{})
	//JSON serializes the given struct as JSON into the response body. It also sets the Content-Type as "application/json".
	//将内容序列化成 JSON 格式，然后放到响应 body 里，同时将 Content-Type 置为 "application/json"
	c.JSON(200, gin.H{
		"userID": claims["id"],
		"text":   "Hello World",
	})
}

//定义一个 User 的结构体
type User struct {
	UserID    string //有三个字段都是 string 类型
	FirstName string
	LastName  string
}

//定义一个回调函数，用来决断用户id和密码是否有效
func authCallback(userId string, password string, c *gin.Context) (interface{}, bool) {
	//这里的 admin 和 test 都以硬代码的方式写进来用来作判断
	//生产环境下使用数据库来存储账号信息，进行检验和判断
	if (userId == "admin" && password == "admin") || (userId == "test" && password == "test") {
		return &User{
			UserID:    userId,
			LastName:  "test Last Name",
			FirstName: "test First Name",
		}, true
	}
	return nil, false
}

//定义一个回调函数，用来决断用户在认证成功的前提下，是否有权限对资源进行访问
func authPrivCallback(user interface{}, c *gin.Context) bool {
	if v, ok := user.(string); ok && v == "admin" {
		return true
	}
	return false
}

//定义一个函数用来处理，认证不成功的情况
func unAuthFunc(c *gin.Context, code int, message string) {
	c.JSON(code, gin.H{
		"code":    code,
		"message": message,
	})
}

func addMidd(v_realm, v_key, v_tokenLookup, v_tokenHeadName string) *jwt.GinJWTMiddleware {
	return &jwt.GinJWTMiddleware{
		//Realm name to display to the user. Required.
		//必要项，显示给用户看的域
		Realm: v_realm,
		//Secret key used for signing. Required.
		//用来进行签名的密钥，就是加盐用的
		Key: []byte(v_key),
		//Duration that a jwt token is valid. Optional, defaults to one hour
		//JWT 的有效时间，默认为一小时
		Timeout: time.Hour,
		// This field allows clients to refresh their token until MaxRefresh has passed.
		// Note that clients can refresh their token in the last moment of MaxRefresh.
		// This means that the maximum validity timespan for a token is MaxRefresh + Timeout.
		// Optional, defaults to 0 meaning not refreshable.
		//最长的刷新时间，用来给客户端自己刷新 token 用的
		MaxRefresh: time.Hour,
		// Callback function that should perform the authentication of the user based on userID and
		// password. Must return true on success, false on failure. Required.
		// Option return user data, if so, user data will be stored in Claim Array.
		//必要项, 这个函数用来判断 User 信息是否合法，如果合法就反馈 true，否则就是 false, 认证的逻辑就在这里
		Authenticator: authCallback,
		// Callback function that should perform the authorization of the authenticated user. Called
		// only after an authentication success. Must return true on success, false on failure.
		// Optional, default to success
		//可选项，用来在 Authenticator 认证成功的基础上进一步的检验用户是否有权限，默认为 success
		Authorizator: authPrivCallback,
		// User can define own Unauthorized func.
		//可以用来息定义如果认证不成功的的处理函数
		Unauthorized: unAuthFunc,
		// TokenLookup is a string in the form of "<source>:<name>" that is used
		// to extract token from the request.
		// Optional. Default value "header:Authorization".
		// Possible values:
		// - "header:<name>"
		// - "query:<name>"
		// - "cookie:<name>"
		//这个变量定义了从请求中解析 token 的格式
		TokenLookup: v_tokenLookup,
		// TokenLookup: "query:token",
		// TokenLookup: "cookie:token",

		// TokenHeadName is a string in the header. Default value is "Bearer"
		//TokenHeadName 是一个头部信息中的字符串
		TokenHeadName: v_tokenHeadName,

		// TimeFunc provides the current time. You can override it to use another time value. This is useful for testing or if your server uses a different time zone than your tokens.
		//这个指定了提供当前时间的函数，也可以自定义
		TimeFunc: time.Now,
	}
}

//main 包必要一个 main 函数，作为起点
func main() {
	//func New() *Viper
	//New returns an initialized Viper instance.
	//用来生成一个新的 viper
	v := viper.New()
	//对 viper 进行配置
	confViper(v)
	//func Default() *Engine
	//Default returns an Engine instance with the Logger and Recovery middleware already attached.
	//用来返回一个已经加载了Logger and Recovery中间件的引擎
	r := gin.Default()

	// the jwt middleware
	authMiddleware := addMidd(v.GetString("realm"), v.GetString("key"), v.GetString("tokenLookup"), v.GetString("tokenHeadName"))

	//func (mw *GinJWTMiddleware) LoginHandler(c *gin.Context)
	//LoginHandler can be used by clients to get a jwt token. Payload needs to be json in the form of {"username": "USERNAME", "password": "PASSWORD"}. Reply will be of the form {"token": "TOKEN"}.
	//将 /login 交给 authMiddleware.LoginHandler 函数来处理
	r.POST(v.GetString("loginPath"), authMiddleware.LoginHandler)

	//func (group *RouterGroup) Group(relativePath string, handlers ...HandlerFunc) *RouterGroup
	//Group creates a new router group. You should add all the routes that have common middlwares or the same path prefix. For example, all the routes that use a common middlware for authorization could be grouped
	//创建一个组 auth
	auth := r.Group(v.GetString("authPath"))
	//func (mw *GinJWTMiddleware) MiddlewareFunc() gin.HandlerFunc
	//MiddlewareFunc makes GinJWTMiddleware implement the Middleware interface.
	//auth 组中使用 MiddlewareFunc 中间件
	auth.Use(authMiddleware.MiddlewareFunc())
	{
		//如果是 /auth 组下的 /hello 就交给 helloHandler 来处理
		auth.GET(v.GetString("testPath"), helloHandler)
		//func (mw *GinJWTMiddleware) RefreshHandler(c *gin.Context)
		//RefreshHandler can be used to refresh a token. The token still needs to be valid on refresh. Shall be put under an endpoint that is using the GinJWTMiddleware. Reply will be of the form {"token": "TOKEN"}.
		//如果是 /auth 组下的 /refresh_token 就交给 RefreshHandler 来处理
		auth.GET(v.GetString("refreshPath"), authMiddleware.RefreshHandler)
	}

	r.Run(":" + v.GetString("port")) //在 0.0.0.0:配置端口 上启监听
}
~~~


## 编译执行

~~~bash
[vagrant@h160 go]$ go run gin_jwt.go 
[GIN-debug] [WARNING] Now Gin requires Go 1.6 or later and Go 1.7 will be required soon.

[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /login                    --> github.com/appleboy/gin-jwt.(*GinJWTMiddleware).LoginHandler-fm (3 handlers)
[GIN-debug] GET    /auth/hello               --> main.helloHandler (4 handlers)
[GIN-debug] GET    /auth/refresh_token       --> github.com/appleboy/gin-jwt.(*GinJWTMiddleware).RefreshHandler-fm (4 handlers)
[GIN-debug] Listening and serving HTTP on :8000
...
...
...
~~~


客户的请求为


分配使用 test/test 和 admin/admin 来获取一下 token


~~~bash
[vagrant@h105 ~]$ curl -v -X POST 'http://192.168.56.160:8000/login'  -H 'Content-Type: application/json'   -d '{"username": "test","password": "test"}'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> POST /login HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Content-Length: 39
> 
* upload completely sent off: 39 out of 39 bytes
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Date: Wed, 22 Aug 2018 14:34:31 GMT
< Content-Length: 211
< 
* Connection #0 to host 192.168.56.160 left intact
{"code":200,"expire":"2018-08-22T23:34:31+08:00","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzQ5NTIwNzEsImlkIjoidGVzdCIsIm9yaWdfaWF0IjoxNTM0OTQ4NDcxfQ.KkSrS9B78y5hMRJNaZ9mZuW4cTW2ZTUBIQY1c1R-75o"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ curl -v -X POST 'http://192.168.56.160:8000/login'  -H 'Content-Type: application/json'   -d '{"username": "admin","password": "admin"}'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> POST /login HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Content-Length: 41
> 
* upload completely sent off: 41 out of 41 bytes
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Date: Wed, 22 Aug 2018 14:34:53 GMT
< Content-Length: 212
< 
* Connection #0 to host 192.168.56.160 left intact
{"code":200,"expire":"2018-08-22T23:34:53+08:00","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzQ5NTIwOTMsImlkIjoiYWRtaW4iLCJvcmlnX2lhdCI6MTUzNDk0ODQ5M30.9cDZXa9DiUdPR_4acW65iJWBt-MJI0GCtveSVTxYNjE"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ curl -v -X GET 'http://192.168.56.160:8000/auth/hello' -H 'Content-Type: application/json'  -H 'Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzQ5NTIwNzEsImlkIjoidGVzdCIsIm9yaWdfaWF0IjoxNTM0OTQ4NDcxfQ.KkSrS9B78y5hMRJNaZ9mZuW4cTW2ZTUBIQY1c1R-75o'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> GET /auth/hello HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzQ5NTIwNzEsImlkIjoidGVzdCIsIm9yaWdfaWF0IjoxNTM0OTQ4NDcxfQ.KkSrS9B78y5hMRJNaZ9mZuW4cTW2ZTUBIQY1c1R-75o
> 
< HTTP/1.1 403 Forbidden
< Content-Type: application/json; charset=utf-8
< Www-Authenticate: JWT realm=testzone
< Date: Wed, 22 Aug 2018 14:35:48 GMT
< Content-Length: 74
< 
* Connection #0 to host 192.168.56.160 left intact
{"code":403,"message":"you don't have permission to access this resource"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ curl -v -X GET 'http://192.168.56.160:8000/auth/hello' -H 'Content-Type: application/json'  -H 'Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzQ5NTIwOTMsImlkIjoiYWRtaW4iLCJvcmlnX2lhdCI6MTUzNDk0ODQ5M30.9cDZXa9DiUdPR_4acW65iJWBt-MJI0GCtveSVTxYNjE'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> GET /auth/hello HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzQ5NTIwOTMsImlkIjoiYWRtaW4iLCJvcmlnX2lhdCI6MTUzNDk0ODQ5M30.9cDZXa9DiUdPR_4acW65iJWBt-MJI0GCtveSVTxYNjE
> 
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Date: Wed, 22 Aug 2018 14:36:24 GMT
< Content-Length: 39
< 
* Connection #0 to host 192.168.56.160 left intact
{"text":"Hello World","userID":"admin"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
~~~

从输出来看，符合预期逻辑


查看服务端的输出

~~~
[vagrant@h160 go]$ go run gin_jwt.go 
[GIN-debug] [WARNING] Now Gin requires Go 1.6 or later and Go 1.7 will be required soon.

[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /login                    --> github.com/appleboy/gin-jwt.(*GinJWTMiddleware).LoginHandler-fm (3 handlers)
[GIN-debug] GET    /auth/hello               --> main.helloHandler (4 handlers)
[GIN-debug] GET    /auth/refresh_token       --> github.com/appleboy/gin-jwt.(*GinJWTMiddleware).RefreshHandler-fm (4 handlers)
[GIN-debug] Listening and serving HTTP on :8000
[GIN] 2018/08/22 - 22:34:31 | 200 |     396.742µs |  192.168.56.105 | POST     /login
[GIN] 2018/08/22 - 22:34:53 | 200 |     120.852µs |  192.168.56.105 | POST     /login
[GIN] 2018/08/22 - 22:35:48 | 403 |     242.787µs |  192.168.56.105 | GET      /auth/hello
[GIN] 2018/08/22 - 22:36:24 | 200 |     193.351µs |  192.168.56.105 | GET      /auth/hello
...
...
...
~~~



---

# 总结 #

使用 **`https://github.com/appleboy/gin-jwt`** 来实现身份认证

充分利用了 **`JWT`** 的认证机制

相对于单纯 cookie 和 session 的方式提供了更高的安全性和灵活信息

服务端也可以设计成无状态的，方便地进行横向扩展

使用 **`github.com/spf13/viper`** 来实现配置管理

让系统的可维护性增加了

* TOC
{:toc}

---

[gin]:https://github.com/gin-gonic/gin
[gin_api_doc]:https://godoc.org/github.com/gin-gonic/gin
[viper]:https://github.com/spf13/viper
[viper_doc]:https://godoc.org/github.com/spf13/viper
