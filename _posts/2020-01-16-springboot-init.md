---
layout: post
title: "Spring Boot Initialization"
date: 2020-01-16 13:13:08
image: '/assets/img/'
description: 'Spring Boot 的初始化'
main-class: java
color: 
tags:
 - sdk
 - gradle
 - springboot
 - java
categories:
 - java
twitter_text: 'simple process of spring boot initialization'
introduction: 'initialization method of spring boot'
---

# 前言

**[Spring Boot][spring_boot]** 是一个目前很受欢迎的 Java web 框架

>Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".

能非常方便地构建一个 Java rest 应用

具备以下特性

~~~
可以构建一个简单的 Spring 应用
可以直接嵌入到 Tomcat, Jetty or Undertow 中，不用布署 war 文件
集成默认的依赖简化构建配置
可以方便地集成第三方库
提供了一些类似于健康检查，外置配置，度量指标一类的生产特性
~~~

作为运维，非常有必要对它的构建过程有一定了解

这里分享一下 **[Spring Boot][spring_boot]** 的构建方法

参考 **[Building a RESTful Web Service][rest_service]**

> **Tip:** 当前版本 **Spring Boot 2.2.3** 

## 依赖与兼容

### Java

* requires Java 8 and is compatible up to Java 13 (included)

### Spring

* Spring Framework 5.2.3.RELEASE or above

### 支持的构建工具

| Build Tool	 | Version                                                            |
|-------------|--------------------------------------------------------------------|
| Maven       | 3\.3\+                                                             |
| Gradle      | 5\.x and 6\.x \(4\.10 is also supported but in a deprecated form\) |

### 支持的 Servlet 容器

|    Name       | servlet Version |
|---------------|-----------------|
| Tomcat 9\.0   | 4\.0            |
| Jetty 9\.4    | 3\.1            |
| Undertow 2\.0 | 4\.0            |


参考 **[spring_boot_start][spring_boot_start]**

---

# 操作

## 系统环境

~~~
(base) [vagrant@workspace java]$ hostnamectl
   Static hostname: workspace
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 583200922eeb40808cf8c6e488ba09c4
           Boot ID: 27e3271027ad4b6090285804f665667e
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
(base) [vagrant@workspace java]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 86257sec preferred_lft 86257sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:3a:8f:13 brd ff:ff:ff:ff:ff:ff
    inet 10.0.1.100/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe3a:8f13/64 scope link
       valid_lft forever preferred_lft forever
(base) [vagrant@workspace java]$ java -version
openjdk version "11.0.5" 2019-10-15
OpenJDK Runtime Environment 18.9 (build 11.0.5+10)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.5+10, mixed mode)
(base) [vagrant@workspace java]$
~~~

## 安装 java

我系统中安装的是 Java 11, 与当前最新的 spring boot 相兼容

但我还是决定添加一个 Java 8 JDK 来进行这个实验

>**Tips:** JDK版本要求 JDK 1.8 or later

~~~
(base) [vagrant@workspace java]$ sdk ls java
================================================================================
Available Java Versions
================================================================================
 Vendor        | Use | Version      | Dist    | Status     | Identifier
--------------------------------------------------------------------------------
 AdoptOpenJDK  |     | 13.0.1.j9    | adpt    |            | 13.0.1.j9-adpt
               |     | 13.0.1.hs    | adpt    |            | 13.0.1.hs-adpt
               |     | 12.0.2.j9    | adpt    |            | 12.0.2.j9-adpt
               |     | 12.0.2.hs    | adpt    |            | 12.0.2.hs-adpt
               |     | 11.0.5.j9    | adpt    |            | 11.0.5.j9-adpt
               |     | 11.0.5.hs    | adpt    |            | 11.0.5.hs-adpt
               |     | 8.0.232.j9   | adpt    |            | 8.0.232.j9-adpt
               |     | 8.0.232.hs   | adpt    |            | 8.0.232.hs-adpt
 Amazon        |     | 11.0.5       | amzn    |            | 11.0.5-amzn
               |     | 8.0.232      | amzn    |            | 8.0.232-amzn
 Azul Zulu     |     | 13.0.1       | zulu    |            | 13.0.1-zulu
               |     | 12.0.2       | zulu    |            | 12.0.2-zulu
               |     | 11.0.5       | zulu    |            | 11.0.5-zulu
               |     | 10.0.2       | zulu    |            | 10.0.2-zulu
               |     | 9.0.7        | zulu    |            | 9.0.7-zulu
               |     | 8.0.232      | zulu    |            | 8.0.232-zulu
               |     | 7.0.242      | zulu    |            | 7.0.242-zulu
               |     | 6.0.119      | zulu    |            | 6.0.119-zulu
 Azul ZuluFX   |     | 11.0.2       | zulufx  |            | 11.0.2-zulufx
               |     | 8.0.202      | zulufx  |            | 8.0.202-zulufx
 BellSoft      |     | 13.0.1       | librca  |            | 13.0.1-librca
               |     | 12.0.2       | librca  |            | 12.0.2-librca
               |     | 11.0.5       | librca  |            | 11.0.5-librca
               |     | 8.0.232      | librca  |            | 8.0.232-librca
 GraalVM       |     | 19.3.1.r11   | grl     |            | 19.3.1.r11-grl
               |     | 19.3.1.r8    | grl     |            | 19.3.1.r8-grl
               |     | 19.3.0.r11   | grl     |            | 19.3.0.r11-grl
               |     | 19.3.0.r8    | grl     |            | 19.3.0.r8-grl
               |     | 19.3.0.2.r11 | grl     |            | 19.3.0.2.r11-grl
               |     | 19.3.0.2.r8  | grl     |            | 19.3.0.2.r8-grl
               |     | 19.2.1       | grl     |            | 19.2.1-grl
               |     | 19.1.1       | grl     |            | 19.1.1-grl
               |     | 19.0.2       | grl     |            | 19.0.2-grl
               |     | 1.0.0        | grl     |            | 1.0.0-rc-16-grl
 Java.net      |     | 15.ea.6      | open    |            | 15.ea.6-open
               |     | 14.ea.32     | open    |            | 14.ea.32-open
               |     | 13.0.1       | open    |            | 13.0.1-open
               |     | 12.0.2       | open    |            | 12.0.2-open
               | >>> | 11.0.5       | open    | installed  | 11.0.5-open
               |     | 10.0.2       | open    |            | 10.0.2-open
               |     | 9.0.4        | open    |            | 9.0.4-open
               |     | 8.0.232      | open    |            | 8.0.232-open
 SAP           |     | 12.0.2       | sapmchn |            | 12.0.2-sapmchn
               |     | 11.0.4       | sapmchn |            | 11.0.4-sapmchn
================================================================================
Use the Identifier for installation:

    $ sdk install java 11.0.3.hs-adpt
================================================================================
(base) [vagrant@workspace java]$ sdk i java 8.0.232-open

Downloading: java 8.0.232-open

In progress...

####################################################################################################################################################### 100.0%

Repackaging Java 8.0.232-open...

Done repackaging...

Installing: java 8.0.232-open
Done installing!

Do you want java 8.0.232-open to be set as default? (Y/n): Y

Setting java 8.0.232-open as default.
(base) [vagrant@workspace java]$ java -version
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (build 1.8.0_232-b09)
OpenJDK 64-Bit Server VM (build 25.232-b09, mixed mode)
(base) [vagrant@workspace java]$
~~~


## 安装 gradle 

构建工具可以使用 Gradle 和 Maven 这里我选择使用 Gradle 6.1

>**Tips:** 构建工具版本 Gradle 4+ or Maven 3.2+

~~~
(base) [vagrant@workspace ~]$ sdk ls gradle
================================================================================
Available Gradle Versions
================================================================================
     6.1                 5.1                 3.5.1               2.2
     6.1-rc-3            5.0                 3.5                 2.1
     6.1-rc-2            4.10.3              3.4.1               2.0
     6.1-rc-1            4.10.2              3.4                 1.12
     6.0.1               4.10.1              3.3                 1.11
     6.0                 4.10                3.2.1               1.10
     6.0-rc-3            4.9                 3.2                 1.9
     6.0-rc-2            4.8.1               3.1                 1.8
     6.0-rc-1            4.8                 3.0                 1.7
     5.6.4               4.7                 2.14.1              1.6
     5.6.3               4.6                 2.14                1.5
     5.6.2               4.5.1               2.13                1.4
     5.6.1               4.5                 2.12                1.3
     5.6                 4.4.1               2.11                1.2
     5.5.1               4.4                 2.10                1.1
     5.5                 4.3.1               2.9                 1.0
     5.4.1               4.3                 2.8                 0.9.2
     5.4                 4.2.1               2.7                 0.9.1
     5.3.1               4.2                 2.6                 0.9
     5.3                 4.1                 2.5                 0.8
     5.2.1               4.0.2               2.4                 0.7
     5.2                 4.0.1               2.3
     5.1.1               4.0                 2.2.1

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================
(base) [vagrant@workspace ~]$ sdk i gradle 6.1

Downloading: gradle 6.1

In progress...

######################################################################### 100.0%

Installing: gradle 6.1
Done installing!


Setting gradle 6.1 as default.
(base) [vagrant@workspace ~]$
(base) [vagrant@workspace ~]$ gradle --help

Welcome to Gradle 6.1!

Here are the highlights of this release:
 - Dependency cache is relocatable
 - Configurable compilation order between Groovy, Java & Scala
 - New sample projects in Gradle's documentation

For more details see https://docs.gradle.org/6.1/release-notes.html


USAGE: gradle [option...] [task...]

-?, -h, --help                 Shows this help message.
-a, --no-rebuild               Do not rebuild project dependencies.
-b, --build-file               Specify the build file.
--build-cache                  Enables the Gradle build cache. Gradle will try to reuse outputs from previous builds.
-c, --settings-file            Specify the settings file.
--configure-on-demand          Configure necessary projects only. Gradle will attempt to reduce configuration time for large multi-project builds. [incubating]
--console                      Specifies which type of console output to generate. Values are 'plain', 'auto' (default), 'rich' or 'verbose'.
--continue                     Continue task execution after a task failure.
-D, --system-prop              Set system property of the JVM (e.g. -Dmyprop=myvalue).
-d, --debug                    Log in debug mode (includes normal stacktrace).
--daemon                       Uses the Gradle Daemon to run the build. Starts the Daemon if not running.
--foreground                   Starts the Gradle Daemon in the foreground.
-g, --gradle-user-home         Specifies the gradle user home directory.
-I, --init-script              Specify an initialization script.
-i, --info                     Set log level to info.
--include-build                Include the specified build in the composite.
-m, --dry-run                  Run the builds with all task actions disabled.
--max-workers                  Configure the number of concurrent workers Gradle is allowed to use.
--no-build-cache               Disables the Gradle build cache.
--no-configure-on-demand       Disables the use of configuration on demand. [incubating]
--no-daemon                    Do not use the Gradle daemon to run the build. Useful occasionally if you have configured Gradle to always run with the daemon by default.
--no-parallel                  Disables parallel execution to build projects.
--no-scan                      Disables the creation of a build scan. For more information about build scans, please visit https://gradle.com/build-scans.
--offline                      Execute the build without accessing network resources.
-P, --project-prop             Set project property for the build script (e.g. -Pmyprop=myvalue).
-p, --project-dir              Specifies the start directory for Gradle. Defaults to current directory.
--parallel                     Build projects in parallel. Gradle will attempt to determine the optimal number of executor threads to use.
--priority                     Specifies the scheduling priority for the Gradle daemon and all processes launched by it. Values are 'normal' (default) or 'low' [incubating]
--profile                      Profile build execution time and generates a report in the <build_dir>/reports/profile directory.
--project-cache-dir            Specify the project-specific cache directory. Defaults to .gradle in the root project directory.
-q, --quiet                    Log errors only.
--refresh-dependencies         Refresh the state of dependencies.
--rerun-tasks                  Ignore previously cached task results.
-S, --full-stacktrace          Print out the full (very verbose) stacktrace for all exceptions.
-s, --stacktrace               Print out the stacktrace for all exceptions.
--scan                         Creates a build scan. Gradle will emit a warning if the build scan plugin has not been applied. (https://gradle.com/build-scans)
--status                       Shows status of running and recently stopped Gradle Daemon(s).
--stop                         Stops the Gradle Daemon if it is running.
-t, --continuous               Enables continuous build. Gradle does not exit and will re-execute tasks when task file inputs change.
--update-locks                 Perform a partial update of the dependency lock, letting passed in module notations change version. [incubating]
-v, --version                  Print version info.
-w, --warn                     Set log level to warn.
--warning-mode                 Specifies which mode of warnings to generate. Values are 'all', 'fail', 'summary'(default) or 'none'
--write-locks                  Persists dependency resolution for locked configurations, ignoring existing locking information if it exists [incubating]
--write-verification-metadata  Generates checksums for dependencies used in the project (comma-separated list) [incubating]
-x, --exclude-task             Specify a task to be excluded from execution.

(base) [vagrant@workspace ~]$ gradle -v

------------------------------------------------------------
Gradle 6.1
------------------------------------------------------------

Build time:   2020-01-15 23:56:46 UTC
Revision:     539d277fdba571ebcc9617a34329c83d7d2b259e

Kotlin:       1.3.61
Groovy:       2.5.8
Ant:          Apache Ant(TM) version 1.10.7 compiled on September 1 2019
JVM:          11.0.5 (Oracle Corporation 11.0.5+10)
OS:           Linux 3.10.0-957.12.2.el7.x86_64 amd64

(base) [vagrant@workspace ~]$
~~~

## 安装 Spring CLI

~~~
(base) [vagrant@workspace ~]$ sdk ls springboot
================================================================================
Available Springboot Versions
================================================================================
     2.2.3.RELEASE       2.0.3.RELEASE       1.5.3.RELEASE       1.2.5.RELEASE
     2.2.2.RELEASE       2.0.2.RELEASE       1.5.2.RELEASE       1.2.4.RELEASE
     2.2.1.RELEASE       2.0.1.RELEASE       1.5.1.RELEASE       1.2.3.RELEASE
     2.2.0.RELEASE       2.0.0.RELEASE       1.4.7.RELEASE       1.2.2.RELEASE
     2.1.12.RELEASE      1.5.22.RELEASE      1.4.6.RELEASE       1.2.1.RELEASE
     2.1.11.RELEASE      1.5.21.RELEASE      1.4.5.RELEASE       1.2.0.RELEASE
     2.1.10.RELEASE      1.5.20.RELEASE      1.4.4.RELEASE       1.1.12.RELEASE
     2.1.9.RELEASE       1.5.19.RELEASE      1.4.3.RELEASE       1.1.11.RELEASE
     2.1.8.RELEASE       1.5.18.RELEASE      1.4.2.RELEASE       1.1.10.RELEASE
     2.1.7.RELEASE       1.5.17.RELEASE      1.4.1.RELEASE       1.1.9.RELEASE
     2.1.6.RELEASE       1.5.16.RELEASE      1.4.0.RELEASE       1.1.8.RELEASE
     2.1.5.RELEASE       1.5.15.RELEASE      1.3.8.RELEASE       1.1.7.RELEASE
     2.1.4.RELEASE       1.5.14.RELEASE      1.3.7.RELEASE       1.1.6.RELEASE
     2.1.3.RELEASE       1.5.13.RELEASE      1.3.6.RELEASE       1.1.5.RELEASE
     2.1.2.RELEASE       1.5.12.RELEASE      1.3.5.RELEASE       1.1.4.RELEASE
     2.1.1.RELEASE       1.5.11.RELEASE      1.3.4.RELEASE       1.1.3.RELEASE
     2.1.0.RELEASE       1.5.10.RELEASE      1.3.3.RELEASE       1.1.2.RELEASE
     2.0.9.RELEASE       1.5.9.RELEASE       1.3.2.RELEASE       1.1.1.RELEASE
     2.0.8.RELEASE       1.5.8.RELEASE       1.3.1.RELEASE       1.1.0.RELEASE
     2.0.7.RELEASE       1.5.7.RELEASE       1.3.0.RELEASE       1.0.2.RELEASE
     2.0.6.RELEASE       1.5.6.RELEASE       1.2.8.RELEASE       1.0.1.RELEASE
     2.0.5.RELEASE       1.5.5.RELEASE       1.2.7.RELEASE       1.0.0.RELEASE
     2.0.4.RELEASE       1.5.4.RELEASE       1.2.6.RELEASE

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================
(base) [vagrant@workspace ~]$ sdk i springboot 2.2.3.RELEASE

Found a previously downloaded springboot 2.2.3.RELEASE archive. Not downloading it again...

Installing: springboot 2.2.3.RELEASE
Done installing!


Setting springboot 2.2.3.RELEASE as default.
(base) [vagrant@workspace ~]$ spring --help
usage: spring [--help] [--version]
       <command> [<args>]

Available commands are:

  run [options] <files> [--] [args]
    Run a spring groovy script

  grab
    Download a spring groovy script's dependencies to ./repository

  jar [options] <jar-name> <files>
    Create a self-contained executable jar file from a Spring Groovy script

  war [options] <war-name> <files>
    Create a self-contained executable war file from a Spring Groovy script

  install [options] <coordinates>
    Install dependencies to the lib/ext directory

  uninstall [options] <coordinates>
    Uninstall dependencies from the lib/ext directory

  init [options] [location]
    Initialize a new project using Spring Initializr (start.spring.io)

  encodepassword [options] <password to encode>
    Encode a password for use with Spring Security

  shell
    Start a nested shell

Common options:

  --debug Verbose mode
    Print additional status information for the command you are running


See 'spring help <command>' for more information on a specific command.
(base) [vagrant@workspace ~]$
(base) [vagrant@workspace ~]$
(base) [vagrant@workspace ~]$ spring --version
Spring CLI v2.2.3.RELEASE
(base) [vagrant@workspace ~]$
~~~


## 构建一个初始化项目

有三种方式来构建一个初始化的项目

### 克隆的方式

直接克隆 **https://github.com/spring-guides/gs-rest-service.git** 就可以生成一个初始的 spring boot 项目

~~~
(base) [vagrant@workspace springboot]$ pwd
/home/vagrant/springboot
(base) [vagrant@workspace springboot]$ ls
(base) [vagrant@workspace springboot]$ git clone https://github.com/spring-guides/gs-rest-service.git
Cloning into 'gs-rest-service'...
remote: Enumerating objects: 65, done.
remote: Counting objects: 100% (65/65), done.
remote: Compressing objects: 100% (33/33), done.
remote: Total 1892 (delta 26), reused 53 (delta 18), pack-reused 1827
Receiving objects: 100% (1892/1892), 1.42 MiB | 13.00 KiB/s, done.
Resolving deltas: 100% (1000/1000), done.
(base) [vagrant@workspace springboot]$ ls
gs-rest-service
(base) [vagrant@workspace springboot]$ cd gs-rest-service/
(base) [vagrant@workspace gs-rest-service]$ ls
complete  CONTRIBUTING.adoc  images  initial  LICENSE.code.txt  LICENSE.writing.txt  push-to-pws  README.adoc  test
(base) [vagrant@workspace gs-rest-service]$ tree initial/
build.gradle     gradle/          gradlew          gradlew.bat      .mvn/            mvnw             mvnw.cmd         pom.xml          settings.gradle  src/
(base) [vagrant@workspace gs-rest-service]$ tree initial/
initial/
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── mvnw
├── mvnw.cmd
├── pom.xml
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── restservice
    │   │               └── RestServiceApplication.java
    │   └── resources
    │       └── application.properties
    └── test
        └── java
            └── com
                └── example
                    └── restservice
                        └── RestServiceApplicationTests.java

14 directories, 12 files
(base) [vagrant@workspace gs-rest-service]$
~~~

>**Tip:** 这里其实包含了两个项目

一个初始版一个完成版

~~~
(base) [vagrant@workspace gs-rest-service]$ ll
total 24
drwxrwxr-x. 5 vagrant vagrant   183 Jan 18 07:39 complete
-rw-rw-r--. 1 vagrant vagrant   133 Jan 18 07:39 CONTRIBUTING.adoc
drwxrwxr-x. 2 vagrant vagrant    28 Jan 18 07:39 images
drwxrwxr-x. 5 vagrant vagrant   163 Jan 18 07:39 initial
-rw-rw-r--. 1 vagrant vagrant   722 Jan 18 07:39 LICENSE.code.txt
-rw-rw-r--. 1 vagrant vagrant   106 Jan 18 07:39 LICENSE.writing.txt
drwxrwxr-x. 2 vagrant vagrant    24 Jan 18 07:39 push-to-pws
-rw-rw-r--. 1 vagrant vagrant 10354 Jan 18 07:39 README.adoc
drwxrwxr-x. 2 vagrant vagrant    20 Jan 18 07:39 test
(base) [vagrant@workspace gs-rest-service]$ tree
.
├── complete
│   ├── build.gradle
│   ├── gradle
│   │   └── wrapper
│   │       ├── gradle-wrapper.jar
│   │       └── gradle-wrapper.properties
│   ├── gradlew
│   ├── gradlew.bat
│   ├── manifest.yml
│   ├── mvnw
│   ├── mvnw.cmd
│   ├── pom.xml
│   ├── settings.gradle
│   └── src
│       ├── main
│       │   └── java
│       │       └── com
│       │           └── example
│       │               └── restservice
│       │                   ├── GreetingController.java
│       │                   ├── Greeting.java
│       │                   └── RestServiceApplication.java
│       └── test
│           └── java
│               └── com
│                   └── example
│                       └── restservice
│                           └── GreetingControllerTests.java
├── CONTRIBUTING.adoc
├── images
│   └── initializr.png
├── initial
│   ├── build.gradle
│   ├── gradle
│   │   └── wrapper
│   │       ├── gradle-wrapper.jar
│   │       └── gradle-wrapper.properties
│   ├── gradlew
│   ├── gradlew.bat
│   ├── mvnw
│   ├── mvnw.cmd
│   ├── pom.xml
│   ├── settings.gradle
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── com
│       │   │       └── example
│       │   │           └── restservice
│       │   │               └── RestServiceApplication.java
│       │   └── resources
│       │       └── application.properties
│       └── test
│           └── java
│               └── com
│                   └── example
│                       └── restservice
│                           └── RestServiceApplicationTests.java
├── LICENSE.code.txt
├── LICENSE.writing.txt
├── push-to-pws
│   └── button.yml
├── README.adoc
└── test
    └── run.sh

32 directories, 33 files
(base) [vagrant@workspace gs-rest-service]$
~~~

### Spring Initializr

通过 **[Spring Initializr][spring_initializr]** 可以快速地生成一个初始项目

![sonarqube](/assets/img/springboot/springboot01.png)

这里可以非常直观地进选择和配置

~~~
(base) [vagrant@workspace demo]$ ls
demo.zip
(base) [vagrant@workspace demo]$ unzip demo.zip
Archive:  demo.zip
   creating: demo/
  inflating: demo/settings.gradle
  inflating: demo/gradlew.bat
  inflating: demo/gradlew
   creating: demo/gradle/
   creating: demo/gradle/wrapper/
  inflating: demo/gradle/wrapper/gradle-wrapper.jar
  inflating: demo/gradle/wrapper/gradle-wrapper.properties
  inflating: demo/build.gradle
   creating: demo/src/
   creating: demo/src/main/
   creating: demo/src/main/java/
   creating: demo/src/main/java/com/
   creating: demo/src/main/java/com/example/
   creating: demo/src/main/java/com/example/demo/
  inflating: demo/src/main/java/com/example/demo/DemoApplication.java
   creating: demo/src/main/resources/
  inflating: demo/src/main/resources/application.properties
   creating: demo/src/test/
   creating: demo/src/test/java/
   creating: demo/src/test/java/com/
   creating: demo/src/test/java/com/example/
   creating: demo/src/test/java/com/example/demo/
  inflating: demo/src/test/java/com/example/demo/DemoApplicationTests.java
  inflating: demo/HELP.md
  inflating: demo/.gitignore
(base) [vagrant@workspace demo]$ ls
demo  demo.zip
(base) [vagrant@workspace demo]$ tree  demo
demo
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── HELP.md
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               └── DemoApplication.java
    │   └── resources
    │       └── application.properties
    └── test
        └── java
            └── com
                └── example
                    └── demo
                        └── DemoApplicationTests.java

14 directories, 10 files
(base) [vagrant@workspace demo]$
~~~

### Spring CLI

spring 的 init 命令可以很快捷地生成一个 spring boot 框架

通过 help 可以看到 init 的用法

~~~
(base) [vagrant@workspace springboot]$ cd cli/
(base) [vagrant@workspace cli]$ ls
(base) [vagrant@workspace cli]$ spring help init
spring init - Initialize a new project using Spring Initializr (start.spring.io)

usage: spring init [options] [location]

Option                       Description
------                       -----------
-a, --artifactId <String>    Project coordinates; infer archive name (for
                               example 'test')
-b, --boot-version <String>  Spring Boot version (for example '1.2.0.RELEASE')
--build <String>             Build system to use (for example 'maven' or
                               'gradle') (default: maven)
-d, --dependencies <String>  Comma-separated list of dependency identifiers to
                               include in the generated project
--description <String>       Project description
-f, --force                  Force overwrite of existing files
--format <String>            Format of the generated content (for example
                               'build' for a build file, 'project' for a
                               project archive) (default: project)
-g, --groupId <String>       Project coordinates (for example 'org.test')
-j, --java-version <String>  Language level (for example '1.8')
-l, --language <String>      Programming language  (for example 'java')
--list                       List the capabilities of the service. Use it to
                               discover the dependencies and the types that are
                               available
-n, --name <String>          Project name; infer application name
-p, --packaging <String>     Project packaging (for example 'jar')
--package-name <String>      Package name
-t, --type <String>          Project type. Not normally needed if you use --
                               build and/or --format. Check the capabilities of
                               the service (--list) for more details
--target <String>            URL of the service to use (default: https://start.
                               spring.io)
-v, --version <String>       Project version (for example '0.0.1-SNAPSHOT')
-x, --extract                Extract the project archive. Inferred if a
                               location is specified without an extension

examples:

    To list all the capabilities of the service:
        $ spring init --list

    To creates a default project:
        $ spring init

    To create a web my-app.zip:
        $ spring init -d=web my-app.zip

    To create a web/data-jpa gradle project unpacked:
        $ spring init -d=web,jpa --build=gradle my-dir


(base) [vagrant@workspace cli]$ spring init -dweb --build gradle -p jar -j 1.8 -l java demo
Using service at https://start.spring.io
Project extracted to '/home/vagrant/springboot/cli/demo'
(base) [vagrant@workspace cli]$ ls
demo
(base) [vagrant@workspace cli]$ tree demo/
demo/
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── HELP.md
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               └── DemoApplication.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── com
                └── example
                    └── demo
                        └── DemoApplicationTests.java

16 directories, 10 files
(base) [vagrant@workspace cli]$
~~~

## 编辑代码加入逻辑

~~~
(base) [vagrant@workspace demo]$ pwd
/home/vagrant/springboot/cli/demo
(base) [vagrant@workspace demo]$ ll
total 24
-rw-rw-r--. 1 vagrant vagrant  514 Jan 18 08:11 build.gradle
drwxrwxr-x. 3 vagrant vagrant   21 Jan 18 08:11 gradle
-rwxrwxr-x. 1 vagrant vagrant 5296 Jan 18 08:11 gradlew
-rw-rw-r--. 1 vagrant vagrant 2260 Jan 18 08:11 gradlew.bat
-rw-rw-r--. 1 vagrant vagrant  939 Jan 18 08:11 HELP.md
-rw-rw-r--. 1 vagrant vagrant   26 Jan 18 08:11 settings.gradle
drwxrwxr-x. 4 vagrant vagrant   30 Jan 18 08:11 src
(base) [vagrant@workspace demo]$ tree
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── HELP.md
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               └── DemoApplication.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── com
                └── example
                    └── demo
                        └── DemoApplicationTests.java

16 directories, 10 files
(base) [vagrant@workspace demo]$ cat build.gradle
plugins {
	id 'org.springframework.boot' version '2.2.3.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
	useJUnitPlatform()
}
(base) [vagrant@workspace demo]$ cat src/main/java/com/example/demo/DemoApplication.java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
(base) [vagrant@workspace demo]$
~~~

编辑一下代码

添加 **`testCompile 'com.jayway.jsonpath:json-path'`** 库

用于接收和传递 JSON 的信息

~~~
(base) [vagrant@workspace demo]$ vim build.gradle
(base) [vagrant@workspace demo]$ cat build.gradle
plugins {
	id 'org.springframework.boot' version '2.2.3.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testCompile 'com.jayway.jsonpath:json-path'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
	useJUnitPlatform()
}
(base) [vagrant@workspace demo]$
~~~

创建一个资源标示类，用于反馈响应结果

再创建一个 Controller 用于对请求进行方法的路由

~~~
(base) [vagrant@workspace demo]$ vim src/main/java/com/example/demo/Greeting.java
(base) [vagrant@workspace demo]$ cat src/main/java/com/example/demo/Greeting.java
package com.example.demo;

public class Greeting {

	private final long id;
	private final String content;

	public Greeting(long id, String content) {
		this.id = id;
		this.content = content;
	}

	public long getId() {
		return id;
	}

	public String getContent() {
		return content;
	}
}
(base) [vagrant@workspace demo]$ vim src/main/java/com/example/demo/GreetingController.java
(base) [vagrant@workspace demo]$ cat src/main/java/com/example/demo/GreetingController.java
package com.example.demo;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

	private static final String template = "Hello, %s!";
	private final AtomicLong counter = new AtomicLong();

	@GetMapping("/greeting")
	public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
		return new Greeting(counter.incrementAndGet(), String.format(template, name));
	}
}
(base) [vagrant@workspace demo]$
(base) [vagrant@workspace demo]$ tree src/main/java/com/example/demo/
src/main/java/com/example/demo/
├── DemoApplication.java
├── GreetingController.java
└── Greeting.java
~~~


## 进行构建

~~~
(base) [vagrant@workspace demo]$ ls
build.gradle  gradle  gradlew  gradlew.bat  HELP.md  settings.gradle  src
(base) [vagrant@workspace demo]$ gradle build
Starting a Gradle Daemon (subsequent builds will be faster)

> Task :test
2020-01-18 15:26:02.952  INFO 14171 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 35m 25s
5 actionable tasks: 5 executed
(base) [vagrant@workspace demo]$
~~~

下面为构建结果

~~~
(base) [vagrant@workspace demo]$ tree build
build
├── classes
│   └── java
│       ├── main
│       │   └── com
│       │       └── example
│       │           └── demo
│       │               ├── DemoApplication.class
│       │               ├── Greeting.class
│       │               └── GreetingController.class
│       └── test
│           └── com
│               └── example
│                   └── demo
│                       └── DemoApplicationTests.class
├── generated
│   └── sources
│       └── annotationProcessor
│           └── java
│               ├── main
│               └── test
├── libs
│   └── demo-0.0.1-SNAPSHOT.jar
├── reports
│   └── tests
│       └── test
│           ├── classes
│           │   └── com.example.demo.DemoApplicationTests.html
│           ├── css
│           │   ├── base-style.css
│           │   └── style.css
│           ├── index.html
│           ├── js
│           │   └── report.js
│           └── packages
│               └── com.example.demo.html
├── resources
│   └── main
│       ├── application.properties
│       ├── static
│       └── templates
├── test-results
│   └── test
│       ├── binary
│       │   ├── output.bin
│       │   ├── output.bin.idx
│       │   └── results.bin
│       └── TEST-com.example.demo.DemoApplicationTests.xml
└── tmp
    ├── bootJar
    │   └── MANIFEST.MF
    ├── compileJava
    └── compileTestJava

35 directories, 17 files
(base) [vagrant@workspace demo]$
~~~


## 运行

直接执行 **`build/libs/demo-0.0.1-SNAPSHOT.jar`**

~~~
(base) [vagrant@workspace demo]$ java -jar build/libs/demo-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.3.RELEASE)

2020-01-18 15:32:45.007  INFO 14329 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication on workspace with PID 14329 (/home/vagrant/springboot/cli/demo/build/libs/demo-0.0.1-SNAPSHOT.jar started by vagrant in /home/vagrant/springboot/cli/demo)
2020-01-18 15:32:45.010  INFO 14329 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
2020-01-18 15:32:45.979  INFO 14329 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-01-18 15:32:45.988  INFO 14329 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-01-18 15:32:45.989  INFO 14329 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.30]
2020-01-18 15:32:46.046  INFO 14329 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-01-18 15:32:46.046  INFO 14329 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 920 ms
2020-01-18 15:32:46.247  INFO 14329 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-01-18 15:32:46.463  INFO 14329 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-01-18 15:32:46.465  INFO 14329 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 2.025 seconds (JVM running for 2.409)
2020-01-18 15:33:15.388  INFO 14329 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2020-01-18 15:33:15.388  INFO 14329 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2020-01-18 15:33:15.393  INFO 14329 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 5 ms
...
...
...
~~~


## 进行访问

~~~
(base) [vagrant@workspace ~]$ curl http://localhost:8080/greeting
{"id":2,"content":"Hello, World!"}(base) [vagrant@workspace ~]$
(base) [vagrant@workspace ~]$
(base) [vagrant@workspace ~]$ curl http://localhost:8080/greeting?name=xxx
{"id":3,"content":"Hello, xxx!"}(base) [vagrant@workspace ~]$
(base) [vagrant@workspace ~]$
(base) [vagrant@workspace ~]$
~~~

到此一个简单的 springboot 的 web server 已经构建完成


---

# 总结

有了 spring cli 一类的脚手架工具，构建一个简单的 spring boot 应用还是很容易的

* TOC
{:toc}


---


[spring_boot]:https://spring.io/projects/spring-boot
[spring_boot_start]:https://docs.spring.io/spring-boot/docs/2.2.3.BUILD-SNAPSHOT/reference/html/getting-started.html#getting-started
[rest_service]:https://spring.io/guides/gs/rest-service/
[spring_initializr]:https://start.spring.io/




