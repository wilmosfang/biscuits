---
layout: post
title: "Config Gradle Pipeline in Jenkins"
date: 2020-02-03 11:50:11
image: '/assets/img/'
description: 'Jenkins 中配置 Gradle pipeline'
main-class: jenkins
color: 
tags: 
 - tools
 - jenkins
 - java
 - gitlab
 - gradle
categories: 
 - jenkins
twitter_text: 'Config Gradle Pipeline in Jenkins'
introduction: 'Config Gradle Pipeline in Jenkins'
---

# 前言

**[Gradle][gradle]** 是一个能更高效完成软件集成的工具

>Accelerate developer productivity
>From mobile apps to microservices, from small startups to big enterprises, Gradle helps teams build, automate and deliver better software, faster.

这里分享一下 **[Gradle][gradle]** 与 Jenkins Pipeline 的集成方法

>**Tip:** 当前的版本为 **Gradle Version: 6.1.1**, **Jenkins ver. 2.214**

---

# 操作

## 系统环境

~~~
[root@jenkins ~]# hostnamectl  
   Static hostname: jenkins
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 03699531e1dc431b849afd1e28341158
           Boot ID: e43ec96db1f3415bb46fc67f6befc632
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@jenkins ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 77163sec preferred_lft 77163sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:db:e0:72 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.11/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fedb:e072/64 scope link 
       valid_lft forever preferred_lft forever
[root@jenkins ~]#
~~~

## 安装与配置 Gradle 插件

安装 gradle 插件

![gradle](/assets/img/jenkins/jenkins33.png)

配置 gradle 插件

**`[Manage Jenkins] > [Global Tool Configuration]`**

![gradle](/assets/img/jenkins/jenkins34.png)

## 配置 pipeline 项目

配置触发构建

![gradle](/assets/img/jenkins/jenkins38.png)

生成触发构建的认证 token

![gradle](/assets/img/jenkins/jenkins39.png)

将 pipeline 的配置放到 SCM 中进行管理 (pipeline as code)

![gradle](/assets/img/jenkins/jenkins40.png)

## 配置 gitlab webhook 

这个配置信息来自于上面的 pipeline 项目

![gradle](/assets/img/gitlab/gitlab08.png)

## 编辑 Pipeline file 

~~~
(base) [vagrant@workspace demo]$ ls
build         gradle   gradlew.bat  Jenkinsfile      src
build.gradle  gradlew  HELP.md      settings.gradle
(base) [vagrant@workspace demo]$ cat Jenkinsfile 
//env config setting
env.GIT_URL="http://10.0.0.12/yyghdfz/springboot_gradle.git"
env.BID="${BUILD_ID}"

//stage
node{
    stage('checkout'){
        git url: "${env.GIT_URL}", credentialsId: "yyghdfz_for_jenkins"
        echo "${BUILD_ID} checkout"
    }
    stage('TEST CONFIRM') {
        input message: "Is it OK in test env?", ok: "Yes, I confirm"  
        echo "ok , got it ,let us go to stage env "
    }
    stage('build'){
        sh "${tool 'Gradle 6.1.1'}/bin/gradle --version"
        sh "${tool 'Gradle 6.1.1'}/bin/gradle clean build"
        echo "build No. ${env.BID}"
    }
    stage('STAGE CONFIRM') {
        input message: "Is it OK in stage env?", ok: "Yes, I confirm"  
        echo "ok , got it ,let us go to prod env "
    }
}
(base) [vagrant@workspace demo]$ 
~~~

>**Note:** 注意这里的用法

**`git url: "${env.GIT_URL}", credentialsId: "yyghdfz_for_jenkins"`**

指定访问 GIT URL 需要用到的认证信息, 需要用到 **credentialsId**, 避免账号密码明文传输, 被记到日志中产生安全隐患

~~~
credentialsId (optional)
Identifier of the credential used to access the remote git repository. Default is '<empty>'.

The credential must be a private key credential if the remote git repository is accessed with the ssh protocol. The credential must be a username / password credential if the remote git repository is accessed with http or https protocol.

Type: String
~~~

使用这种形式来以脚本风格执行 gradle 命令

**`sh "${tool 'Gradle 6.1.1'}/bin/gradle clean build"`**


声明式Pipeline

~~~
pipeline {
    agent any
    tools {
        gradle "Gradle 6.1.1"
    }
    stages {
        stage('Gradle') {
            steps {
                sh 'gradle -v'
            }
        }
    }
}
~~~


脚本式Pipeline

~~~
node {
    stage('Gradle') {
        sh "${tool 'Gradle 6.1.1'}/bin/gradle -v"
    }
}
~~~

## 进行构建测试

直接运行此任务，会失败，console 日志如下

~~~
Started by user admin
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/gradle_pipeline
[Pipeline] {
[Pipeline] stage
[Pipeline] { (checkout)
[Pipeline] git
using credential yyghdfz_for_jenkins
 > /bin/git rev-parse --is-inside-work-tree # timeout=10
Fetching changes from the remote Git repository
 > /bin/git config remote.origin.url http://10.0.0.12/yyghdfz/springboot_gradle.git # timeout=10
Fetching upstream changes from http://10.0.0.12/yyghdfz/springboot_gradle.git
 > /bin/git --version # timeout=10
using GIT_ASKPASS to set credentials jenkins user
 > /bin/git fetch --tags --progress http://10.0.0.12/yyghdfz/springboot_gradle.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > /bin/git rev-parse refs/remotes/origin/master^{commit} # timeout=10
 > /bin/git rev-parse refs/remotes/origin/origin/master^{commit} # timeout=10
Checking out Revision b8ac0cfb98abc544f9d1ed5543c9359d44412921 (refs/remotes/origin/master)
 > /bin/git config core.sparsecheckout # timeout=10
 > /bin/git checkout -f b8ac0cfb98abc544f9d1ed5543c9359d44412921 # timeout=10
 > /bin/git branch -a -v --no-abbrev # timeout=10
 > /bin/git branch -D master # timeout=10
 > /bin/git checkout -b master b8ac0cfb98abc544f9d1ed5543c9359d44412921 # timeout=10
Commit message: "2"
 > /bin/git rev-list --no-walk b8ac0cfb98abc544f9d1ed5543c9359d44412921 # timeout=10
[Pipeline] echo
11 checkout
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (STAGE CONFIRM)
[Pipeline] input
Is it OK in stage env?
Yes, I confirm or Abort
Approved by admin
[Pipeline] echo
ok , got it ,let us go to prod env 
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (build)
[Pipeline] tool
[Pipeline] sh
+ /var/lib/jenkins/tools/hudson.plugins.gradle.GradleInstallation/Gradle_6.1.1/bin/gradle --version

------------------------------------------------------------
Gradle 6.1.1
------------------------------------------------------------

Build time:   2020-01-24 22:30:24 UTC
Revision:     a8c3750babb99d1894378073499d6716a1a1fa5d

Kotlin:       1.3.61
Groovy:       2.5.8
Ant:          Apache Ant(TM) version 1.10.7 compiled on September 1 2019
JVM:          1.8.0_232 (Oracle Corporation 25.232-b09)
OS:           Linux 3.10.0-957.12.2.el7.x86_64 amd64

[Pipeline] tool
[Pipeline] sh
+ /var/lib/jenkins/tools/hudson.plugins.gradle.GradleInstallation/Gradle_6.1.1/bin/gradle clean build
Starting a Gradle Daemon (subsequent builds will be faster)
> Task :clean UP-TO-DATE
> Task :compileJava FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileJava'.
> Could not find tools.jar. Please check that /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64/jre contains a valid JDK installation.

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.1.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD FAILED in 7s
2 actionable tasks: 1 executed, 1 up-to-date
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
ERROR: script returned exit code 1
Finished: FAILURE
~~~

提示找不到 **tools.jar**

~~~
[Pipeline] tool
[Pipeline] sh
+ /var/lib/jenkins/tools/hudson.plugins.gradle.GradleInstallation/Gradle_6.1.1/bin/gradle clean build
Starting a Gradle Daemon (subsequent builds will be faster)
> Task :clean UP-TO-DATE
> Task :compileJava FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileJava'.
> Could not find tools.jar. Please check that /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64/jre contains a valid JDK installation.

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.1.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD FAILED in 7s
~~~

直接使用 jenkins 的身份执行，提示相同

~~~
-bash-4.2$ /var/lib/jenkins/tools/hudson.plugins.gradle.GradleInstallation/Gradle_6.1.1/bin/gradle clean build
Starting a Gradle Daemon (subsequent builds will be faster)
> Task :compileJava FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileJava'.
> Could not find tools.jar. Please check that /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64/jre contains a valid JDK installation.

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.1.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD FAILED in 6s
2 actionable tasks: 2 executed
-bash-4.2$
~~~

### 修复 gradle 报错

解决此问题的办法是安装上 **`java-1.8.0-openjdk-devel.x86_64`**

~~~
[root@jenkins ~]# rpm -qa | grep openjdk
java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64
java-1.8.0-openjdk-headless-1.8.0.232.b09-0.el7_7.x86_64
[root@jenkins ~]# yum list all | grep java-1.8.0-openjdk
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
java-1.8.0-openjdk.x86_64                   1:1.8.0.232.b09-0.el7_7    @updates 
java-1.8.0-openjdk-headless.x86_64          1:1.8.0.232.b09-0.el7_7    @updates 
java-1.8.0-openjdk.i686                     1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-accessibility.i686       1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-accessibility.x86_64     1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-accessibility-debug.i686 1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-accessibility-debug.x86_64
java-1.8.0-openjdk-debug.i686               1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-debug.x86_64             1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-demo.i686                1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-demo.x86_64              1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-demo-debug.i686          1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-demo-debug.x86_64        1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-devel.i686               1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-devel.x86_64             1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-devel-debug.i686         1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-devel-debug.x86_64       1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-headless.i686            1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-headless-debug.i686      1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-headless-debug.x86_64    1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-javadoc.noarch           1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-javadoc-debug.noarch     1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-javadoc-zip.noarch       1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-javadoc-zip-debug.noarch 1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-src.i686                 1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-src.x86_64               1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-src-debug.i686           1:1.8.0.232.b09-0.el7_7    updates  
java-1.8.0-openjdk-src-debug.x86_64         1:1.8.0.232.b09-0.el7_7    updates  
[root@jenkins ~]# 
[root@jenkins ~]# yum install -y java-1.8.0-openjdk-devel.x86_64
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.163.com
 * extras: mirrors.163.com
 * updates: mirrors.163.com
Resolving Dependencies
--> Running transaction check
---> Package java-1.8.0-openjdk-devel.x86_64 1:1.8.0.242.b08-0.el7_7 will be installed
--> Processing Dependency: java-1.8.0-openjdk(x86-64) = 1:1.8.0.242.b08-0.el7_7 for package: 1:java-1.8.0-openjdk-devel-1.8.0.242.b08-0.el7_7.x86_64
--> Running transaction check
---> Package java-1.8.0-openjdk.x86_64 1:1.8.0.232.b09-0.el7_7 will be updated
---> Package java-1.8.0-openjdk.x86_64 1:1.8.0.242.b08-0.el7_7 will be an update
--> Processing Dependency: java-1.8.0-openjdk-headless(x86-64) = 1:1.8.0.242.b08-0.el7_7 for package: 1:java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64
--> Running transaction check
---> Package java-1.8.0-openjdk-headless.x86_64 1:1.8.0.232.b09-0.el7_7 will be updated
---> Package java-1.8.0-openjdk-headless.x86_64 1:1.8.0.242.b08-0.el7_7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================================================================================================================
 Package                                              Arch                            Version                                            Repository                        Size
================================================================================================================================================================================
Installing:
 java-1.8.0-openjdk-devel                             x86_64                          1:1.8.0.242.b08-0.el7_7                            updates                          9.8 M
Updating for dependencies:
 java-1.8.0-openjdk                                   x86_64                          1:1.8.0.242.b08-0.el7_7                            updates                          293 k
 java-1.8.0-openjdk-headless                          x86_64                          1:1.8.0.242.b08-0.el7_7                            updates                           32 M

Transaction Summary
================================================================================================================================================================================
Install  1 Package
Upgrade             ( 2 Dependent packages)

Total download size: 42 M
Downloading packages:
No Presto metadata available for updates
(1/3): java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64.rpm                                                                                               | 293 kB  00:00:00     
(2/3): java-1.8.0-openjdk-devel-1.8.0.242.b08-0.el7_7.x86_64.rpm                                                                                         | 9.8 MB  00:00:02     
(3/3): java-1.8.0-openjdk-headless-1.8.0.242.b08-0.el7_7.x86_64.rpm                                                                                      |  32 MB  00:00:16     
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                           2.4 MB/s |  42 MB  00:00:17     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 1:java-1.8.0-openjdk-headless-1.8.0.242.b08-0.el7_7.x86_64                                                                                                   1/5 
warning: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/jre/lib/security/java.security created as /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/jre/lib/security/java.security.rpmnew
restored /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/jre/lib/security/java.security.rpmnew to /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/jre/lib/security/java.security
  Updating   : 1:java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64                                                                                                            2/5 
  Installing : 1:java-1.8.0-openjdk-devel-1.8.0.242.b08-0.el7_7.x86_64                                                                                                      3/5 
  Cleanup    : 1:java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64                                                                                                            4/5 
  Cleanup    : 1:java-1.8.0-openjdk-headless-1.8.0.232.b09-0.el7_7.x86_64                                                                                                   5/5 
  Verifying  : 1:java-1.8.0-openjdk-headless-1.8.0.242.b08-0.el7_7.x86_64                                                                                                   1/5 
  Verifying  : 1:java-1.8.0-openjdk-devel-1.8.0.242.b08-0.el7_7.x86_64                                                                                                      2/5 
  Verifying  : 1:java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64                                                                                                            3/5 
  Verifying  : 1:java-1.8.0-openjdk-headless-1.8.0.232.b09-0.el7_7.x86_64                                                                                                   4/5 
  Verifying  : 1:java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64                                                                                                            5/5 

Installed:
  java-1.8.0-openjdk-devel.x86_64 1:1.8.0.242.b08-0.el7_7                                                                                                                       

Dependency Updated:
  java-1.8.0-openjdk.x86_64 1:1.8.0.242.b08-0.el7_7                                  java-1.8.0-openjdk-headless.x86_64 1:1.8.0.242.b08-0.el7_7                                 

Complete!
[root@jenkins ~]# 
~~~


再以 jenkins 的身份进行构建

~~~
-bash-4.2$ cd /var/lib/jenkins/workspace/
-bash-4.2$ ls
gradle_hello      gradle_pipeline      hello_project
gradle_hello@tmp  gradle_pipeline@tmp  hello_project@tmp
-bash-4.2$ cd gradle_pipeline
-bash-4.2$ ls
build  build.gradle  gradle  gradlew  gradlew.bat  settings.gradle  src
-bash-4.2$ /var/lib/jenkins/tools/hudson.plugins.gradle.GradleInstallation/Gradle_6.1.1/bin/gradle clean build
Starting a Gradle Daemon, 1 incompatible Daemon could not be reused, use --status for details

> Task :test
2020-02-03 11:04:36.823  INFO 7491 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.1.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 15s
6 actionable tasks: 6 executed
-bash-4.2$ ls
build  build.gradle  gradle  gradlew  gradlew.bat  settings.gradle  src
-bash-4.2$ ./gradlew clean build
Starting a Gradle Daemon, 1 incompatible Daemon could not be reused, use --status for details

> Task :test
2020-02-03 11:05:34.957  INFO 7597 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.0.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 41s
6 actionable tasks: 6 executed
-bash-4.2$ 
~~~

## 提交测试

~~~
(base) [vagrant@workspace demo]$ ls
build         gradle   gradlew.bat  Jenkinsfile      src
build.gradle  gradlew  HELP.md      settings.gradle
(base) [vagrant@workspace demo]$ vim Jenkinsfile 
(base) [vagrant@workspace demo]$ git add . ; git commit -m "test"; git push 
[master 3388bd5] test
 1 file changed, 1 insertion(+), 1 deletion(-)
warning: push.default is unset; its implicit value is changing in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the current behavior after the default changes, use:

  git config --global push.default matching

To squelch this message and adopt the new behavior now, use:

  git config --global push.default simple

See 'git help config' and search for 'push.default' for further information.
(the 'simple' mode was introduced in Git 1.7.11. Use the similar mode
'current' instead of 'simple' if you sometimes use older versions of Git)

Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 275 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
To git@10.0.0.12:yyghdfz/springboot_gradle.git
   1dfd539..3388bd5  master -> master
(base) [vagrant@workspace demo]$ 
~~~

![gradle](/assets/img/jenkins/jenkins41.png)

![gradle](/assets/img/jenkins/jenkins42.png)

![gradle](/assets/img/jenkins/jenkins43.png)

![gradle](/assets/img/jenkins/jenkins44.png)


再查看日志

~~~
Started by GitLab push by yyghdfz
Obtained Jenkinsfile from git http://10.0.0.12/yyghdfz/springboot_gradle.git
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/gradle_pipeline
[Pipeline] {
[Pipeline] stage
[Pipeline] { (checkout)
[Pipeline] git
using credential yyghdfz_for_jenkins
 > /bin/git rev-parse --is-inside-work-tree # timeout=10
Fetching changes from the remote Git repository
 > /bin/git config remote.origin.url http://10.0.0.12/yyghdfz/springboot_gradle.git # timeout=10
Fetching upstream changes from http://10.0.0.12/yyghdfz/springboot_gradle.git
 > /bin/git --version # timeout=10
using GIT_ASKPASS to set credentials jenkins user
 > /bin/git fetch --tags --progress http://10.0.0.12/yyghdfz/springboot_gradle.git +refs/heads/*:refs/remotes/origin/* # timeout=10
skipping resolution of commit remotes/origin/master, since it originates from another repository
 > /bin/git rev-parse refs/remotes/origin/master^{commit} # timeout=10
 > /bin/git rev-parse refs/remotes/origin/origin/master^{commit} # timeout=10
Checking out Revision 3388bd5d2915cf5549e91ad3f28288df1a018dfc (refs/remotes/origin/master)
 > /bin/git config core.sparsecheckout # timeout=10
 > /bin/git checkout -f 3388bd5d2915cf5549e91ad3f28288df1a018dfc # timeout=10
 > /bin/git branch -a -v --no-abbrev # timeout=10
 > /bin/git branch -D master # timeout=10
 > /bin/git checkout -b master 3388bd5d2915cf5549e91ad3f28288df1a018dfc # timeout=10
Commit message: "test"
 > /bin/git rev-list --no-walk 1dfd539c6cdb33aa0f730bffbdd13f0674a3746e # timeout=10
[Pipeline] echo
22 checkout
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (TEST CONFIRM)
[Pipeline] input
Is it OK in test env?
Yes, I confirm or Abort
Approved by admin
[Pipeline] echo
ok , got it ,let us go to stage env 
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (build)
[Pipeline] tool
[Pipeline] sh
+ /var/lib/jenkins/tools/hudson.plugins.gradle.GradleInstallation/Gradle_6.1.1/bin/gradle --version

------------------------------------------------------------
Gradle 6.1.1
------------------------------------------------------------

Build time:   2020-01-24 22:30:24 UTC
Revision:     a8c3750babb99d1894378073499d6716a1a1fa5d

Kotlin:       1.3.61
Groovy:       2.5.8
Ant:          Apache Ant(TM) version 1.10.7 compiled on September 1 2019
JVM:          1.8.0_242 (Oracle Corporation 25.242-b08)
OS:           Linux 3.10.0-957.12.2.el7.x86_64 amd64

[Pipeline] tool
[Pipeline] sh
+ /var/lib/jenkins/tools/hudson.plugins.gradle.GradleInstallation/Gradle_6.1.1/bin/gradle clean build
> Task :clean
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
2020-02-03 14:17:20.633  INFO 9721 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'

> Task :check
> Task :build

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.1.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 15s
6 actionable tasks: 6 executed
[Pipeline] echo
build No. 22
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (STAGE CONFIRM)
[Pipeline] input
Is it OK in stage env?
Yes, I confirm or Abort
Approved by admin
[Pipeline] echo
ok , got it ,let us go to prod env 
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
~~~

到此 Gradle 与 Jenkins Pipeline 的集成就演示完毕了

---

# 总结

上面的步骤描述了 **[Gradle][gradle]** 与 Jenkins Pipeline 的集成过程

后面再将上述结果打包成 Docker image

* TOC
{:toc}


---

[gradle]:https://gradle.org/