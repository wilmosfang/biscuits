---
layout: post
title: "Gin JWT Gorm"
date: 2018-08-23 16:03:46
image: '/assets/img/'
description:  '使用 DB 来存放认证信息进行 web 认证'
main-class:  'go'
color: '#68d6e3'
tags:
 - go
 - gin
 - jwt
 - viper
 - gorm
categories: 
 - go
twitter_text: 'Web auth by DB'
introduction: 'Web auth by DB'
---

# 前言 #


**[Gin][gin]** 是一款用 Go(Golang) 编写的 web 框架

>Gin is a web framework written in Go (Golang). It features a martini-like API with much better performance, up to 40 times faster thanks to httprouter

因为 httprouter, 它提供了更高的性能

配置管理是一个 Web 应用非常关键的环节

这里使用 **[Viper][viper]** 来实现

什么是 **Viper** 的文档可以参考 **[package viper][viper_doc]** 

这里演示一下在 **[Gin][gin]** 中如何进行配置管理

同时接合 **Gorm** 和 **JWT** 一起来实现一个基于数据库的 web 登录认证功能

Gin 的 API 可以参考 **[API REFFERENCE][gin_api_doc]**

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
	//用来格式化输出一些内容
	"fmt"
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
	//The fantastic ORM library for Golang
	//一个非常好用的 go 语言 ORM 库
	"github.com/jinzhu/gorm"
	//加入postgres的库
	_ "github.com/jinzhu/gorm/dialects/postgres"
)

//定义一个全局的 db 指针用来进行认证，数据校验
var authDB *gorm.DB

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
	loginPath='/loginpath'
	authPath='/authpath'
	refreshPath='/refresh_token'
	testPath='/hello'
	db_host     = "192.168.56.160"
	db_port     = "5432"
	db_user     = "gorm"
	db_name   = "testdb"
	db_password = "123456"
	admin_name = 'dex'
	admin_pass = 'dex'
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
	v.SetDefault("db_host", "127.0.0.1")
	v.SetDefault("db_port", "5432")
	v.SetDefault("db_user", "postgresql")
	v.SetDefault("db_name", "testdb")
	v.SetDefault("db_password", "123456")
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

//定义一个 User 的结构体, 用来存放用户名和密码
type User struct {
	gorm.Model
	UserID   string `gorm:"type:varchar(30);UNIQUE;unique_index;not null"`
	Password string `gorm:"size:255"`
}

//定义一个回调函数，用来决断用户id和密码是否有效
func authCallback(userId string, password string, c *gin.Context) (interface{}, bool) {
	//这里的通过从数据库中查询来判断是否为现存用户，生产环境下一般都会使用数据库来存储账号信息，进行检验和判断
	user := User{}
	//如果这条记录存在
	if !authDB.Where("user_id = ?", userId).Find(&user).RecordNotFound() {
		//定义一个临时的结构对象
		queryRes := User{}
		//将 user_id 为认证信息中的 密码找出来(目前密码是明文的，这个其实不安全，可以通过加盐哈希将结果进行对比的方式以提高安全等级，这里只作原理演示，就不搞那么复杂了)
		//找到后放到前面定义的临时结构变量里
		authDB.Where("user_id = ?", userId).Find(&queryRes)
		//对比，如果密码也相同，就代表认证成功了
		if queryRes.Password == password {
			//反馈相关信息和 true 的值，代表成功
			return &User{
				UserID: userId,
			}, true
		}
	}
	//否则返回失败
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
	pg_conn_info := fmt.Sprintf("host=%s port=%s user=%s dbname=%s password=%s sslmode=disable", v.GetString("db_host"), v.GetString("db_port"), v.GetString("db_user"), v.GetString("db_name"), v.GetString("db_password"))
	db, err := gorm.Open("postgres", pg_conn_info)
	if err != nil {
		panic(err) //如果出错，就直接打印出错信息，并且退出
	} else {
		fmt.Println("Successfully connected!") //如果没有出错，就打印成功连接的信息
		authDB = db                            //连接成功的情况下将认证的数据库进行赋值
	}
	//func (s *DB) Close() error
	//Close close current db connection. If database connection is not an io.Closer, returns an error.
	//panic 之后并不是直接退出，而是先去执行 defer 的内容
	//关闭当前的 db 连接
	defer db.Close() //如果出错，先将 db 关掉
	//func (s *DB) AutoMigrate(values ...interface{}) *DB
	//AutoMigrate run auto migration for given models, will only add missing fields, won't delete/change current data
	//AutoMigrate 会自动将给定的模型进行迁移，只会添加缺失的字段，并不会删除或者修改当前的字段
	db.AutoMigrate(&User{})
	//创建一个结构变量
	user := User{}
	//如果 db 中没有这条记录，就创建，如果有就忽略掉
	if db.Where("user_id = ?", v.GetString("admin_name")).Find(&user).RecordNotFound() {
		user := User{UserID: v.GetString("admin_name"), Password: v.GetString("admin_pass")}
		db.Create(&user)
	}
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

数据库的状态

~~~bash
testdb=> \d
Did not find any relations.
testdb=> 
~~~


## 编译执行

~~~bash
[vagrant@h160 go]$ go run gin_jwt.go 
Successfully connected!
[GIN-debug] [WARNING] Now Gin requires Go 1.6 or later and Go 1.7 will be required soon.

[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /loginpath                --> github.com/appleboy/gin-jwt.(*GinJWTMiddleware).LoginHandler-fm (3 handlers)
[GIN-debug] GET    /authpath/hello           --> main.helloHandler (4 handlers)
[GIN-debug] GET    /authpath/refresh_token   --> github.com/appleboy/gin-jwt.(*GinJWTMiddleware).RefreshHandler-fm (4 handlers)
[GIN-debug] Listening and serving HTTP on :8000
...
...
...
~~~

可见配置生效了，默认值也生效了


再看看数据库的状态

~~~bash
testdb=> \d
Did not find any relations.
testdb=> \d
            List of relations
 Schema |     Name     |   Type   | Owner 
--------|--------------|----------|-------
 public | users        | table    | gorm
 public | users_id_seq | sequence | gorm
(2 rows)

testdb=> \d users
                                       Table "public.users"
   Column   |           Type           | Collation | Nullable |              Default              
------------|--------------------------|-----------|----------|-----------------------------------
 id         | integer                  |           | not null | nextval('users_id_seq'::regclass)
 created_at | timestamp with time zone |           |          | 
 updated_at | timestamp with time zone |           |          | 
 deleted_at | timestamp with time zone |           |          | 
 user_id    | character varying(30)    |           | not null | 
 password   | character varying(255)   |           |          | 
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
    "uix_users_user_id" UNIQUE, btree (user_id)
    "users_user_id_key" UNIQUE CONSTRAINT, btree (user_id)
    "idx_users_deleted_at" btree (deleted_at)

testdb=> select * from users;
 id |          created_at           |          updated_at           | deleted_at | user_id | password 
----|-------------------------------|-------------------------------|------------|---------|----------
  1 | 2018-08-24 00:13:38.031988+08 | 2018-08-24 00:13:38.031988+08 |            | dex     | dex
(1 row)

testdb=> 
~~~

数据表与一条管理员的记录已经生成了，符合预期逻辑


进行三次登录

~~~bash
[vagrant@h105 ~]$ curl -v -X POST 'http://192.168.56.160:8000/loginpath'  -H 'Content-Type: application/json'   -d '{"username": "def","password": "def"}'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> POST /loginpath HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Content-Length: 37
> 
* upload completely sent off: 37 out of 37 bytes
< HTTP/1.1 401 Unauthorized
< Content-Type: application/json; charset=utf-8
< Www-Authenticate: JWT realm=testzone
< Date: Thu, 23 Aug 2018 16:18:35 GMT
< Content-Length: 55
< 
* Connection #0 to host 192.168.56.160 left intact
{"code":401,"message":"incorrect Username or Password"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ curl -v -X POST 'http://192.168.56.160:8000/loginpath'  -H 'Content-Type: application/json'   -d '{"username": "dex","password": "dex"}'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> POST /loginpath HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Content-Length: 37
> 
* upload completely sent off: 37 out of 37 bytes
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Date: Thu, 23 Aug 2018 16:18:59 GMT
< Content-Length: 209
< 
* Connection #0 to host 192.168.56.160 left intact
{"code":200,"expire":"2018-08-24T01:18:59+08:00","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzUwNDQ3MzksImlkIjoiZGV4Iiwib3JpZ19pYXQiOjE1MzUwNDExMzl9.ExgnJ6IdKdjgjgGKgynWQLcPwy0SMuOpIrlO548RsYU"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ curl -v -X POST 'http://192.168.56.160:8000/loginpath'  -H 'Content-Type: application/json'   -d '{"username": "dex","password": "dexx"}'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> POST /loginpath HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Content-Length: 38
> 
* upload completely sent off: 38 out of 38 bytes
< HTTP/1.1 401 Unauthorized
< Content-Type: application/json; charset=utf-8
< Www-Authenticate: JWT realm=testzone
< Date: Thu, 23 Aug 2018 16:21:17 GMT
< Content-Length: 55
< 
* Connection #0 to host 192.168.56.160 left intact
{"code":401,"message":"incorrect Username or Password"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
~~~

进行了如预期的认证，一个成功，一个失败


创建一个新用户

~~~bash
testdb=> select * from users;
 id |          created_at           |          updated_at           | deleted_at | user_id | password 
----|-------------------------------|-------------------------------|------------|---------|----------
  1 | 2018-08-24 00:13:38.031988+08 | 2018-08-24 00:13:38.031988+08 |            | dex     | dex
(1 row)

testdb=> insert into users(id,created_at,updated_at,user_id,password) values ( 10000,'2018-08-23 21:42:12.44424+08','2018-08-23 21:42:12.44424+08','admin','admin');
INSERT 0 1
testdb=> 
testdb=> select * from users;
  id   |          created_at           |          updated_at           | deleted_at | user_id | password 
-------|-------------------------------|-------------------------------|------------|---------|----------
     1 | 2018-08-24 00:13:38.031988+08 | 2018-08-24 00:13:38.031988+08 |            | dex     | dex
 10000 | 2018-08-23 21:42:12.44424+08  | 2018-08-23 21:42:12.44424+08  |            | admin   | admin
(2 rows)

testdb=>
~~~


再次进行访问

~~~bash
[vagrant@h105 ~]$ curl -v -X POST 'http://192.168.56.160:8000/loginpath'  -H 'Content-Type: application/json'   -d '{"username": "admin","password": "admin"}'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> POST /loginpath HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Content-Length: 41
> 
* upload completely sent off: 41 out of 41 bytes
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Date: Thu, 23 Aug 2018 16:25:20 GMT
< Content-Length: 212
< 
* Connection #0 to host 192.168.56.160 left intact
{"code":200,"expire":"2018-08-24T01:25:20+08:00","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzUwNDUxMjAsImlkIjoiYWRtaW4iLCJvcmlnX2lhdCI6MTUzNTA0MTUyMH0.6J0Gf3go2M0EGU_x41_khdGG6ozSDK3o-6mQ-TdF0xo"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ curl -v -X POST 'http://192.168.56.160:8000/loginpath'  -H 'Content-Type: application/json'   -d '{"username": "admin","password": "adminx"}'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> POST /loginpath HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Content-Length: 42
> 
* upload completely sent off: 42 out of 42 bytes
< HTTP/1.1 401 Unauthorized
< Content-Type: application/json; charset=utf-8
< Www-Authenticate: JWT realm=testzone
< Date: Thu, 23 Aug 2018 16:25:23 GMT
< Content-Length: 55
< 
* Connection #0 to host 192.168.56.160 left intact
{"code":401,"message":"incorrect Username or Password"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ curl -v -X POST 'http://192.168.56.160:8000/loginpath'  -H 'Content-Type: application/json'   -d '{"username": "admin","password": "admin"}'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> POST /loginpath HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Content-Length: 41
> 
* upload completely sent off: 41 out of 41 bytes
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Date: Thu, 23 Aug 2018 16:25:26 GMT
< Content-Length: 212
< 
* Connection #0 to host 192.168.56.160 left intact
{"code":200,"expire":"2018-08-24T01:25:26+08:00","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzUwNDUxMjYsImlkIjoiYWRtaW4iLCJvcmlnX2lhdCI6MTUzNTA0MTUyNn0.RASqJxP-17ZkxotjsucJHXmz6orMMeODygYPxEWIRHE"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
~~~

尝试访问受限资源

~~~bash
[vagrant@h105 ~]$ curl -v -X GET 'http://192.168.56.160:8000/authpath/hello' -H 'Content-Type: application/json'  -H 'Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzUwNDUxMjYsImlkIjoiYWRtaW4iLCJvcmlnX2lhdCI6MTUzNTA0MTUyNn0.RASqJxP-17ZkxotjsucJHXmz6orMMeODygYPxEWIRHE'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> GET /authpath/hello HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzUwNDUxMjYsImlkIjoiYWRtaW4iLCJvcmlnX2lhdCI6MTUzNTA0MTUyNn0.RASqJxP-17ZkxotjsucJHXmz6orMMeODygYPxEWIRHE
> 
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Date: Thu, 23 Aug 2018 16:28:21 GMT
< Content-Length: 39
< 
* Connection #0 to host 192.168.56.160 left intact
{"text":"Hello World","userID":"admin"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
[vagrant@h105 ~]$
[vagrant@h105 ~]$ curl -v -X GET 'http://192.168.56.160:8000/authpath/hello' -H 'Content-Type: application/json'  -H 'Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzUwNDQ3MzksImlkIjoiZGV4Iiwib3JpZ19pYXQiOjE1MzUwNDExMzl9.ExgnJ6IdKdjgjgGKgynWQLcPwy0SMuOpIrlO548RsYU'
* About to connect() to 192.168.56.160 port 8000 (#0)
*   Trying 192.168.56.160...
* Connected to 192.168.56.160 (192.168.56.160) port 8000 (#0)
> GET /authpath/hello HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.56.160:8000
> Accept: */*
> Content-Type: application/json
> Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzUwNDQ3MzksImlkIjoiZGV4Iiwib3JpZ19pYXQiOjE1MzUwNDExMzl9.ExgnJ6IdKdjgjgGKgynWQLcPwy0SMuOpIrlO548RsYU
> 
< HTTP/1.1 403 Forbidden
< Content-Type: application/json; charset=utf-8
< Www-Authenticate: JWT realm=testzone
< Date: Thu, 23 Aug 2018 16:29:13 GMT
< Content-Length: 74
< 
* Connection #0 to host 192.168.56.160 left intact
{"code":403,"message":"you don't have permission to access this resource"}[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
[vagrant@h105 ~]$ 
~~~


查看服务端的输出

~~~bash
[vagrant@h160 go]$ go run gin_jwt.go 
Successfully connected!
[GIN-debug] [WARNING] Now Gin requires Go 1.6 or later and Go 1.7 will be required soon.

[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /loginpath                --> github.com/appleboy/gin-jwt.(*GinJWTMiddleware).LoginHandler-fm (3 handlers)
[GIN-debug] GET    /authpath/hello           --> main.helloHandler (4 handlers)
[GIN-debug] GET    /authpath/refresh_token   --> github.com/appleboy/gin-jwt.(*GinJWTMiddleware).RefreshHandler-fm (4 handlers)
[GIN-debug] Listening and serving HTTP on :8000
[GIN] 2018/08/24 - 00:18:35 | 401 |     865.894µs |  192.168.56.105 | POST     /loginpath
[GIN] 2018/08/24 - 00:18:59 | 200 |    1.400773ms |  192.168.56.105 | POST     /loginpath
[GIN] 2018/08/24 - 00:21:17 | 401 |    1.304586ms |  192.168.56.105 | POST     /loginpath
[GIN] 2018/08/24 - 00:25:20 | 200 |    1.397148ms |  192.168.56.105 | POST     /loginpath
[GIN] 2018/08/24 - 00:25:23 | 401 |    1.487195ms |  192.168.56.105 | POST     /loginpath
[GIN] 2018/08/24 - 00:25:26 | 200 |    1.165295ms |  192.168.56.105 | POST     /loginpath
[GIN] 2018/08/24 - 00:28:21 | 200 |     119.231µs |  192.168.56.105 | GET      /authpath/hello
[GIN] 2018/08/24 - 00:29:13 | 403 |      154.74µs |  192.168.56.105 | GET      /authpath/hello
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

让系统的可维护性增加

这里使用 **Gorm** 来存账户和密码信息，实现了最基础的认证功能 

* TOC
{:toc}

---

[gin]:https://github.com/gin-gonic/gin
[gin_api_doc]:https://godoc.org/github.com/gin-gonic/gin
[viper]:https://github.com/spf13/viper
[viper_doc]:https://godoc.org/github.com/spf13/viper
