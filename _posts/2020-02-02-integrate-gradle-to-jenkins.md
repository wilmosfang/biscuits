---
layout: post
title: "Integrate Gradle to Jenkins"
date: 2020-02-02 12:10:05
image: '/assets/img/'
description: 'Gradle 集成到 Jenkins'
main-class: jenkins
color: 
tags: 
 - tools
 - jenkins
 - gradle
 - java
 - gitlab
categories: 
 - jenkins
twitter_text: 'Integrate Gradle to Jenkins'
introduction: 'Integrate Gradle to Jenkins'
---

# 前言

接着前面的 workshop 这里分享一下 **[Gradle][gradle]** 与 Jenkins 的集成方法

> **Tip:** 当前版本 **Gradle Version: 6.1.1**, **Jenkins ver. 2.214** 

---

# 操作

## 系统环境

~~~
(base) [vagrant@workspace demo]$ hostnamectl 
   Static hostname: workspace
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 583200922eeb40808cf8c6e488ba09c4
           Boot ID: 7c1897f3e048466199677522c37e3abd
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
(base) [vagrant@workspace demo]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84160sec preferred_lft 84160sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:3a:8f:13 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.100/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe3a:8f13/64 scope link 
       valid_lft forever preferred_lft forever
(base) [vagrant@workspace demo]$ 
~~~

## Jenkins 中安装 gradle 插件

安装 gradle 插件

![gradle](/assets/img/jenkins/jenkins33.png)

## 配置 gradle 插件

[Manage Jenkins] > [Global Tool Configuration]

![gradle](/assets/img/jenkins/jenkins34.png)


## gitlab 中创建一个新的 repo

![gradle](/assets/img/gitlab/gitlab04.png)


## 初始化此 repo

~~~
(base) [vagrant@workspace demo]$ git config --global user.name "yyghdfz"
(base) [vagrant@workspace demo]$ git config --global user.email "yyghdfz@163.com"
(base) [vagrant@workspace demo]$ git init
Initialized empty Git repository in /home/vagrant/springboot/cli/demo/.git/
(base) [vagrant@workspace demo]$ git remote add origin git@10.0.0.12:yyghdfz/springboot_gradle.git
(base) [vagrant@workspace demo]$ git add .
(base) [vagrant@workspace demo]$ git commit -m "Initial commit"
[master (root-commit) 4e7d011] Initial commit
 12 files changed, 386 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 build.gradle
 create mode 100644 gradle/wrapper/gradle-wrapper.jar
 create mode 100644 gradle/wrapper/gradle-wrapper.properties
 create mode 100755 gradlew
 create mode 100644 gradlew.bat
 create mode 100644 settings.gradle
 create mode 100644 src/main/java/com/example/demo/DemoApplication.java
 create mode 100644 src/main/java/com/example/demo/Greeting.java
 create mode 100644 src/main/java/com/example/demo/GreetingController.java
 create mode 100644 src/main/resources/application.properties
 create mode 100644 src/test/java/com/example/demo/DemoApplicationTests.java
(base) [vagrant@workspace demo]$ git push -u origin master
Counting objects: 28, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (18/18), done.
Writing objects: 100% (28/28), 54.94 KiB | 0 bytes/s, done.
Total 28 (delta 0), reused 0 (delta 0)
To git@10.0.0.12:yyghdfz/springboot_gradle.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
(base) [vagrant@workspace demo]$ 
~~~

![gradle](/assets/img/gitlab/gitlab05.png)


## 配置一个 jenkins 项目

配置好 gitlab repo 的地址

![gradle](/assets/img/jenkins/jenkins35.png)

配置 webhook 的key

![gradle](/assets/img/jenkins/jenkins36.png)

选择之前配置的 Gradle 版本

![gradle](/assets/img/jenkins/jenkins37.png)

## 配置 gitlab 的 webhook

![gradle](/assets/img/gitlab/gitlab06.png)

![gradle](/assets/img/gitlab/gitlab07.png)

## 执行构建

以下为构建的 console 日志

~~~
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/gradle_hello
using credential yyghdfz_for_jenkins
 > /bin/git rev-parse --is-inside-work-tree # timeout=10
Fetching changes from the remote Git repository
 > /bin/git config remote.origin.url http://10.0.0.12/yyghdfz/springboot_gradle.git # timeout=10
Fetching upstream changes from http://10.0.0.12/yyghdfz/springboot_gradle.git
 > /bin/git --version # timeout=10
using GIT_ASKPASS to set credentials jenkins user
 > /bin/git fetch --tags --progress http://10.0.0.12/yyghdfz/springboot_gradle.git +refs/heads/*:refs/remotes/origin/* # timeout=10
Seen branch in repository origin/master
Seen 1 remote branch
 > /bin/git show-ref --tags -d # timeout=10
Checking out Revision 4e7d011b8d9feb900b344af3d1ae21822f69a0bb (origin/master)
 > /bin/git config core.sparsecheckout # timeout=10
 > /bin/git checkout -f 4e7d011b8d9feb900b344af3d1ae21822f69a0bb # timeout=10
Commit message: "Initial commit"
 > /bin/git rev-list --no-walk 4e7d011b8d9feb900b344af3d1ae21822f69a0bb # timeout=10
[Gradle] - Launching build.
Unpacking https://services.gradle.org/distributions/gradle-6.1.1-bin.zip to /var/lib/jenkins/tools/hudson.plugins.gradle.GradleInstallation/Gradle_6.1.1 on Jenkins
[gradle_hello] $ /var/lib/jenkins/tools/hudson.plugins.gradle.GradleInstallation/Gradle_6.1.1/bin/gradle clean build

Welcome to Gradle 6.1.1!

Here are the highlights of this release:
 - Reusable dependency cache
 - Configurable compilation order between Groovy/Kotlin/Java/Scala
 - New sample projects in Gradle's documentation

For more details see https://docs.gradle.org/6.1.1/release-notes.html

Starting a Gradle Daemon (subsequent builds will be faster)
> Task :clean UP-TO-DATE
> Task :compileJava
> Task :processResources
> Task :classes
> Task :bootJar
> Task :jar SKIPPED
> Task :assemble
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses

> Task :test
2020-02-02 13:08:32.233  INFO 6029 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'

> Task :check
> Task :build

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.1.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 2m 38s
6 actionable tasks: 5 executed, 1 up-to-date
Build step 'Invoke Gradle script' changed build result to SUCCESS
Finished: SUCCESS
~~~

## 测试生成的 jar 包

~~~
[root@jenkins libs]# pwd
/var/lib/jenkins/workspace/gradle_hello/build/libs
[root@jenkins libs]# 
[root@jenkins libs]# java -jar demo-0.0.1-SNAPSHOT.jar --server.port=8000

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.3.RELEASE)

2020-02-02 13:14:44.833  INFO 6123 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication on jenkins with PID 6123 (/var/lib/jenkins/workspace/gradle_hello/build/libs/demo-0.0.1-SNAPSHOT.jar started by root in /var/lib/jenkins/workspace/gradle_hello/build/libs)
2020-02-02 13:14:44.836  INFO 6123 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
2020-02-02 13:14:46.352  INFO 6123 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8000 (http)
2020-02-02 13:14:46.365  INFO 6123 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-02-02 13:14:46.365  INFO 6123 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.30]
2020-02-02 13:14:46.455  INFO 6123 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-02-02 13:14:46.455  INFO 6123 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1512 ms
2020-02-02 13:14:46.715  INFO 6123 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-02-02 13:14:46.918  INFO 6123 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8000 (http) with context path ''
2020-02-02 13:14:46.922  INFO 6123 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 2.774 seconds (JVM running for 3.264)
2020-02-02 13:14:49.648  INFO 6123 --- [nio-8000-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2020-02-02 13:14:49.648  INFO 6123 --- [nio-8000-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2020-02-02 13:14:49.654  INFO 6123 --- [nio-8000-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 5 ms
...
...
...
~~~

## 进行访问测试

~~~
(base) [vagrant@workspace demo]$ curl http://10.0.0.11:8000/greeting?name=xxx
{"id":2,"content":"Hello, xxx!"}(base) [vagrant@workspace demo]$ 
(base) [vagrant@workspace demo]$
~~~

## 测试通过 webhook 自动触发

~~~
(base) [vagrant@workspace demo]$ ls
DemoApplication.java  GreetingController.java  Greeting.java
(base) [vagrant@workspace demo]$ cat GreetingController.java 
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
(base) [vagrant@workspace demo]$ vim GreetingController.java 
(base) [vagrant@workspace demo]$ cat GreetingController.java 
package com.example.demo;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

	private static final String template = "Hello,test %s!";
	private final AtomicLong counter = new AtomicLong();

	@GetMapping("/greeting")
	public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
		return new Greeting(counter.incrementAndGet(), String.format(template, name));
	}
}
(base) [vagrant@workspace demo]$ git add . 
(base) [vagrant@workspace demo]$ git commit -m 'add test'
[master 7b51bd6] add test
 1 file changed, 1 insertion(+), 1 deletion(-)
(base) [vagrant@workspace demo]$ git push 
warning: push.default is unset; its implicit value is changing in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the current behavior after the default changes, use:

  git config --global push.default matching

To squelch this message and adopt the new behavior now, use:

  git config --global push.default simple

See 'git help config' and search for 'push.default' for further information.
(the 'simple' mode was introduced in Git 1.7.11. Use the similar mode
'current' instead of 'simple' if you sometimes use older versions of Git)

Counting objects: 17, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 603 bytes | 0 bytes/s, done.
Total 9 (delta 3), reused 0 (delta 0)
To git@10.0.0.12:yyghdfz/springboot_gradle.git
   4e7d011..7b51bd6  master -> master
(base) [vagrant@workspace demo]$ 
~~~

这时 jenkins 的界面中可以看到，项目被自动触发了并且随后完成了构建

~~~
[root@jenkins libs]# java -jar demo-0.0.1-SNAPSHOT.jar --server.port=8001

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.3.RELEASE)

2020-02-02 14:01:46.525  INFO 6346 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication on jenkins with PID 6346 (/var/lib/jenkins/workspace/gradle_hello/build/libs/demo-0.0.1-SNAPSHOT.jar started by root in /var/lib/jenkins/workspace/gradle_hello/build/libs)
2020-02-02 14:01:46.527  INFO 6346 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
2020-02-02 14:01:47.706  INFO 6346 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8001 (http)
2020-02-02 14:01:47.724  INFO 6346 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-02-02 14:01:47.724  INFO 6346 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.30]
2020-02-02 14:01:47.802  INFO 6346 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-02-02 14:01:47.802  INFO 6346 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1176 ms
2020-02-02 14:01:47.988  INFO 6346 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-02-02 14:01:48.257  INFO 6346 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8001 (http) with context path ''
2020-02-02 14:01:48.259  INFO 6346 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 2.367 seconds (JVM running for 3.112)
2020-02-02 14:02:16.389  INFO 6346 --- [nio-8001-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2020-02-02 14:02:16.389  INFO 6346 --- [nio-8001-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2020-02-02 14:02:16.394  INFO 6346 --- [nio-8001-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 5 ms

...
...
...
~~~

测试访问，也如预期发生了变化

~~~
(base) [vagrant@workspace demo]$ curl http://10.0.0.11:8000/greeting?name=xxx
curl: (7) Failed to connect to 10.0.0.11 port 8000: Connection refused
(base) [vagrant@workspace demo]$ curl http://10.0.0.11:8001/greeting?name=xxx
{"id":1,"content":"Hello,test xxx!"}(base) [vagrant@workspace demo]$ 
~~~

到此 Gradle 与 Jenkins 的 CI 集成就演示完毕了

---

# 总结

上面的步骤描述了 **[Gradle][gradle]** 与 Jenkins 的集成过程

后面可以将上述过程在 Jenkins 里改造成 Pipeline 

* TOC
{:toc}

---

[gradle]:https://gradle.org/
