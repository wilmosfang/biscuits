---
layout: post
title: "SonarScanner for Maven"
date: 2020-01-13 14:24:14
image: '/assets/img/'
description: 'SonarScanner 集成到 Maven'
main-class: tools
color: 
tags: 
 - maven
 - tools
 - sonarqube
 - java
categories: 
 - sonarqube
twitter_text: 'sonarscanner for maven'
introduction: 'sonarscanner for maven'
---

# 前言

**[SonarQube][sonarqube]** 是一个很受欢迎的静态代码扫描软件

>Your teammate for Code Quality and Security
>SonarQube empowers all developers to write cleaner and safer code

能非常方便地集成到 CI/CD 环境中

这里分享一下 **[SonarQube][sonarqube]** 与 Maven 的集成方法

参考 **[SonarScanner for Maven][sonarscanner_for_maven]**

> **Tip:** 当前版本 **SonarQube Version: 8.1** Release on December 2019

---

# 操作

## 系统环境

~~~
[vagrant@workspace ~]$ hostnamectl  
   Static hostname: workspace
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 583200922eeb40808cf8c6e488ba09c4
           Boot ID: 42a763d1cd80442f8192dcfb686c86ac
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[vagrant@workspace ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 83718sec preferred_lft 83718sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:3a:8f:13 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.100/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe3a:8f13/64 scope link 
       valid_lft forever preferred_lft forever
[vagrant@workspace ~]$ 
~~~


## 软件版本

java 版本为 **11.0.5**

~~~
[vagrant@workspace ~]$ java -version
openjdk version "11.0.5" 2019-10-15
OpenJDK Runtime Environment 18.9 (build 11.0.5+10)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.5+10, mixed mode)
[vagrant@workspace ~]$
~~~

Maven 版本为 **3.6.3**

~~~
[vagrant@workspace ~]$ mvn -v
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /home/vagrant/.sdkman/candidates/maven/current
Java version: 11.0.5, vendor: Oracle Corporation, runtime: /home/vagrant/.sdkman/candidates/java/11.0.5-open
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-957.12.2.el7.x86_64", arch: "amd64", family: "unix"
[vagrant@workspace ~]$ 
~~~

SonarQube 的版本为 **Community EditionVersion 8.1 (build 31237)**

## SonarQube 的服务地址

服务地址为 **`http://10.0.0.13:8000/sonarqube`**

~~~
[sonar@sonarqube conf]$ pwd
/home/sonar/sonarqube-8.1.0.31237/conf
[sonar@sonarqube conf]$ grep -i 'sonar.web' sonar.properties  | grep -v "#"
sonar.web.host=0.0.0.0
sonar.web.context=/sonarqube
sonar.web.port=8000
[sonar@sonarqube conf]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 80531sec preferred_lft 80531sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:6b:b4:f8 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.13/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe6b:b4f8/64 scope link 
       valid_lft forever preferred_lft forever
[sonar@sonarqube conf]$ 
~~~


## 配置 Maven

~~~
[vagrant@workspace ~]$ which mvn 
~/.sdkman/candidates/maven/current/bin/mvn
[vagrant@workspace ~]$ cd ~/.sdkman/candidates/maven/current/
[vagrant@workspace current]$ ls
bin  boot  conf  lib  LICENSE  NOTICE  README.txt
[vagrant@workspace current]$ ll conf/
total 16
drwxr-xr-x. 2 vagrant vagrant    37 Nov  7 12:32 logging
-rw-r--r--. 1 vagrant vagrant 10965 Jan 13 14:06 settings.xml
-rw-r--r--. 1 vagrant vagrant  3747 Nov  7 12:32 toolchains.xml
[vagrant@workspace current]$ vim conf/settings.xml 
[vagrant@workspace current]$ 
~~~


### 配置 pluginGroup

添加一个 **`<pluginGroup>org.sonarsource.scanner.maven</pluginGroup>`**

~~~
 <!-- pluginGroups
   | This is a list of additional group identifiers that will be searched when resolving plugins by their prefix, i.e.
   | when invoking a command line like "mvn prefix:goal". Maven will automatically add the group identifiers
   | "org.apache.maven.plugins" and "org.codehaus.mojo" if these are not already contained in the list.
   |-->
  <pluginGroups>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
    <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
  </pluginGroups>
~~~

### 配置 profile

~~~
</profiles>
...
...
        <profile>
            <id>sonar</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <!-- Optional URL to server. Default value is http://localhost:9000 -->
                <sonar.host.url>
                  http://10.0.0.13:8000/sonarqube
                </sonar.host.url>
            </properties>
        </profile>
  </profiles>
~~~


## 进行代码扫描

解压一个 demo 项目

~~~
[vagrant@workspace ~]$ unzip x.zip 
Archive:  x.zip
   creating: maven-hello/
   creating: maven-hello/target/
  inflating: maven-hello/pom.xml     
  inflating: maven-hello/.classpath  
  inflating: maven-hello/.project    
   creating: maven-hello/src/
   creating: maven-hello/src/test/
   creating: maven-hello/src/test/java/
   creating: maven-hello/src/test/java/com/
   creating: maven-hello/src/test/java/com/itranswarp/
   creating: maven-hello/src/test/java/com/itranswarp/learnjava/
  inflating: maven-hello/src/test/java/com/itranswarp/learnjava/MainTest.java  
   creating: maven-hello/src/main/
   creating: maven-hello/src/main/java/
   creating: maven-hello/src/main/java/com/
   creating: maven-hello/src/main/java/com/itranswarp/
   creating: maven-hello/src/main/java/com/itranswarp/learnjava/
  inflating: maven-hello/src/main/java/com/itranswarp/learnjava/Main.java  
[vagrant@workspace ~]$  
[vagrant@workspace ~]$ ls
maven-hello  maven-plugin  maven-plugin.zip  x.zip
[vagrant@workspace ~]$ cd maven-hello/
[vagrant@workspace maven-hello]$ ls
pom.xml  src  target
[vagrant@workspace maven-hello]$ tree 
.
├── pom.xml
├── src
│   ├── main
│   │   └── java
│   │       └── com
│   │           └── itranswarp
│   │               └── learnjava
│   │                   └── Main.java
│   └── test
│       └── java
│           └── com
│               └── itranswarp
│                   └── learnjava
│                       └── MainTest.java
└── target

12 directories, 3 files
[vagrant@workspace maven-hello]$
~~~


进行检查

~~~
[vagrant@workspace maven-hello]$ mvn clean verify sonar:sonar 
[INFO] Scanning for projects...
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-surefire-plugin/3.0.0-M3/maven-surefire-plugin-3.0.0-M3.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-surefire-plugin/3.0.0-M3/maven-surefire-plugin-3.0.0-M3.pom (7.2 kB at 1.6 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire/3.0.0-M3/surefire-3.0.0-M3.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire/3.0.0-M3/surefire-3.0.0-M3.pom (23 kB at 3.7 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-surefire-plugin/3.0.0-M3/maven-surefire-plugin-3.0.0-M3.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-surefire-plugin/3.0.0-M3/maven-surefire-plugin-3.0.0-M3.jar (40 kB at 7.7 kB/s)
[INFO] 
[INFO] ----------------< com.itranswarp.learnjava:maven-hello >----------------
[INFO] Building hello 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-engine/5.5.2/junit-jupiter-engine-5.5.2.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-engine/5.5.2/junit-jupiter-engine-5.5.2.pom (2.4 kB at 4.4 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.1.0/apiguardian-api-1.1.0.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.1.0/apiguardian-api-1.1.0.pom (1.2 kB at 797 B/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.5.2/junit-platform-engine-1.5.2.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.5.2/junit-platform-engine-1.5.2.pom (2.4 kB at 1.4 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.pom (1.7 kB at 1.1 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.5.2/junit-platform-commons-1.5.2.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.5.2/junit-platform-commons-1.5.2.pom (2.0 kB at 3.7 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-api/5.5.2/junit-jupiter-api-5.5.2.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-api/5.5.2/junit-jupiter-api-5.5.2.pom (2.4 kB at 4.4 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-engine/5.5.2/junit-jupiter-engine-5.5.2.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.5.2/junit-platform-commons-1.5.2.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.1.0/apiguardian-api-1.1.0.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.5.2/junit-platform-engine-1.5.2.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-engine/5.5.2/junit-jupiter-engine-5.5.2.jar (220 kB at 127 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-api/5.5.2/junit-jupiter-api-5.5.2.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.1.0/apiguardian-api-1.1.0.jar (2.4 kB at 1.1 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-api/5.5.2/junit-jupiter-api-5.5.2.jar (142 kB at 52 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.jar (7.7 kB at 1.9 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.5.2/junit-platform-commons-1.5.2.jar (92 kB at 10 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.5.2/junit-platform-engine-1.5.2.jar (164 kB at 9.7 kB/s)
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-hello ---
[INFO] Deleting /home/vagrant/maven-hello/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-hello ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/vagrant/maven-hello/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-hello ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/vagrant/maven-hello/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ maven-hello ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/vagrant/maven-hello/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ maven-hello ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/vagrant/maven-hello/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:3.0.0-M3:test (default-test) @ maven-hello ---
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/maven-surefire-common/3.0.0-M3/maven-surefire-common-3.0.0-M3.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/maven-surefire-common/3.0.0-M3/maven-surefire-common-3.0.0-M3.pom (9.7 kB at 5.9 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-api/3.0.0-M3/surefire-api-3.0.0-M3.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-api/3.0.0-M3/surefire-api-3.0.0-M3.pom (3.5 kB at 5.4 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-logger-api/3.0.0-M3/surefire-logger-api-3.0.0-M3.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-logger-api/3.0.0-M3/surefire-logger-api-3.0.0-M3.pom (2.0 kB at 4.2 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-booter/3.0.0-M3/surefire-booter-3.0.0-M3.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-booter/3.0.0-M3/surefire-booter-3.0.0-M3.pom (7.4 kB at 3.9 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-toolchain/3.0-alpha-2/maven-toolchain-3.0-alpha-2.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-toolchain/3.0-alpha-2/maven-toolchain-3.0-alpha-2.pom (2.2 kB at 2.1 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven/3.0-alpha-2/maven-3.0-alpha-2.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven/3.0-alpha-2/maven-3.0-alpha-2.pom (21 kB at 9.6 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-parent/10/maven-parent-10.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-parent/10/maven-parent-10.pom (32 kB at 9.2 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-compat/3.0/maven-compat-3.0.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-compat/3.0/maven-compat-3.0.pom (4.0 kB at 5.6 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-provider-api/1.0-beta-6/wagon-provider-api-1.0-beta-6.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-provider-api/1.0-beta-6/wagon-provider-api-1.0-beta-6.pom (1.8 kB at 1.3 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon/1.0-beta-6/wagon-1.0-beta-6.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon/1.0-beta-6/wagon-1.0-beta-6.pom (12 kB at 5.1 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/1.4.2/plexus-utils-1.4.2.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/1.4.2/plexus-utils-1.4.2.pom (2.0 kB at 1.3 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-java/1.0.1/plexus-java-1.0.1.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-java/1.0.1/plexus-java-1.0.1.pom (4.8 kB at 3.6 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-languages/1.0.1/plexus-languages-1.0.1.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-languages/1.0.1/plexus-languages-1.0.1.pom (4.2 kB at 2.8 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/com/thoughtworks/qdox/qdox/2.0-M9/qdox-2.0-M9.pom
Downloaded from central: https://repo.maven.apache.org/maven2/com/thoughtworks/qdox/qdox/2.0-M9/qdox-2.0-M9.pom (16 kB at 2.6 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/sonatype/oss/oss-parent/9/oss-parent-9.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/oss/oss-parent/9/oss-parent-9.pom (6.6 kB at 3.8 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/maven-surefire-common/3.0.0-M3/maven-surefire-common-3.0.0-M3.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-api/3.0.0-M3/surefire-api-3.0.0-M3.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-logger-api/3.0.0-M3/surefire-logger-api-3.0.0-M3.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-booter/3.0.0-M3/surefire-booter-3.0.0-M3.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-toolchain/3.0-alpha-2/maven-toolchain-3.0-alpha-2.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-booter/3.0.0-M3/surefire-booter-3.0.0-M3.jar (312 kB at 103 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/2.0.4/plexus-utils-2.0.4.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-logger-api/3.0.0-M3/surefire-logger-api-3.0.0-M3.jar (13 kB at 4.0 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-compat/3.0/maven-compat-3.0.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/2.0.4/plexus-utils-2.0.4.jar (222 kB at 34 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-provider-api/1.0-beta-6/wagon-provider-api-1.0-beta-6.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-toolchain/3.0-alpha-2/maven-toolchain-3.0-alpha-2.jar (36 kB at 5.2 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-java/1.0.1/plexus-java-1.0.1.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-provider-api/1.0-beta-6/wagon-provider-api-1.0-beta-6.jar (53 kB at 6.2 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm/7.0-beta/asm-7.0-beta.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-java/1.0.1/plexus-java-1.0.1.jar (51 kB at 5.0 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/com/thoughtworks/qdox/qdox/2.0-M9/qdox-2.0-M9.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm/7.0-beta/asm-7.0-beta.jar (114 kB at 10 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-api/3.0.0-M3/surefire-api-3.0.0-M3.jar (178 kB at 6.5 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/com/thoughtworks/qdox/qdox/2.0-M9/qdox-2.0-M9.jar (317 kB at 10 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-compat/3.0/maven-compat-3.0.jar (285 kB at 6.2 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/maven-surefire-common/3.0.0-M3/maven-surefire-common-3.0.0-M3.jar (627 kB at 6.0 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit-platform/3.0.0-M3/surefire-junit-platform-3.0.0-M3.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit-platform/3.0.0-M3/surefire-junit-platform-3.0.0-M3.jar (20 kB at 8.7 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit-platform/3.0.0-M3/surefire-junit-platform-3.0.0-M3.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit-platform/3.0.0-M3/surefire-junit-platform-3.0.0-M3.pom (5.5 kB at 1.4 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-providers/3.0.0-M3/surefire-providers-3.0.0-M3.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-providers/3.0.0-M3/surefire-providers-3.0.0-M3.pom (2.5 kB at 3.0 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/common-java5/3.0.0-M3/common-java5-3.0.0-M3.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/common-java5/3.0.0-M3/common-java5-3.0.0-M3.pom (3.1 kB at 2.2 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-launcher/1.3.1/junit-platform-launcher-1.3.1.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-launcher/1.3.1/junit-platform-launcher-1.3.1.pom (2.2 kB at 3.4 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.0.0/apiguardian-api-1.0.0.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.0.0/apiguardian-api-1.0.0.pom (1.2 kB at 2.5 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.3.1/junit-platform-engine-1.3.1.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.3.1/junit-platform-engine-1.3.1.pom (2.4 kB at 1.3 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.3.1/junit-platform-commons-1.3.1.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.3.1/junit-platform-commons-1.3.1.pom (2.0 kB at 2.3 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.1.1/opentest4j-1.1.1.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.1.1/opentest4j-1.1.1.pom (1.7 kB at 1.4 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/common-java5/3.0.0-M3/common-java5-3.0.0-M3.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-launcher/1.3.1/junit-platform-launcher-1.3.1.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.0.0/apiguardian-api-1.0.0.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.3.1/junit-platform-engine-1.3.1.jar
Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.3.1/junit-platform-commons-1.3.1.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.0.0/apiguardian-api-1.0.0.jar (2.2 kB at 3.8 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.1.1/opentest4j-1.1.1.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.1.1/opentest4j-1.1.1.jar (7.1 kB at 9.6 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.3.1/junit-platform-commons-1.3.1.jar (78 kB at 55 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/common-java5/3.0.0-M3/common-java5-3.0.0-M3.jar (32 kB at 9.4 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.3.1/junit-platform-engine-1.3.1.jar (135 kB at 10 kB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-launcher/1.3.1/junit-platform-launcher-1.3.1.jar (95 kB at 4.8 kB/s)
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.itranswarp.learnjava.MainTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.036 s - in com.itranswarp.learnjava.MainTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ maven-hello ---
[INFO] Building jar: /home/vagrant/maven-hello/target/maven-hello-1.0-SNAPSHOT.jar
[INFO] 
[INFO] ----------------< com.itranswarp.learnjava:maven-hello >----------------
[INFO] Building hello 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- sonar-maven-plugin:3.7.0.1746:sonar (default-cli) @ maven-hello ---
[INFO] User cache: /home/vagrant/.sonar/cache
[INFO] SonarQube version: 8.1.0
[INFO] Default locale: "en_US", source code encoding: "UTF-8"
[INFO] Load global settings
[INFO] Load global settings (done) | time=91ms
[INFO] Server id: 5ACACF06-AW-QG8czf4dfML9SHoJZ
[INFO] User cache: /home/vagrant/.sonar/cache
[INFO] Load/download plugins
[INFO] Load plugins index
[INFO] Load plugins index (done) | time=52ms
[INFO] Load/download plugins (done) | time=105ms
[INFO] Process project properties
[INFO] Process project properties (done) | time=9ms
[INFO] Execute project builders
[INFO] Execute project builders (done) | time=2ms
[INFO] Project key: com.itranswarp.learnjava:maven-hello
[INFO] Base dir: /home/vagrant/maven-hello
[INFO] Working dir: /home/vagrant/maven-hello/target/sonar
[INFO] Load project settings for component key: 'com.itranswarp.learnjava:maven-hello'
[INFO] Load quality profiles
[INFO] Load quality profiles (done) | time=53ms
[INFO] Load active rules
[INFO] Load active rules (done) | time=739ms
[WARNING] SCM provider autodetection failed. Please use "sonar.scm.provider" to define SCM of your project, or disable the SCM Sensor in the project settings.
[INFO] Indexing files...
[INFO] Project configuration:
[INFO] 3 files indexed
[INFO] Quality profile for java: Sonar way
[INFO] Quality profile for xml: Sonar way
[INFO] ------------- Run sensors on module hello
[INFO] Load metrics repository
[INFO] Load metrics repository (done) | time=28ms
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by net.sf.cglib.core.ReflectUtils$1 (file:/home/vagrant/.sonar/cache/a2b6a8802525720c8af2ca4fa85a2513/sonar-javascript-plugin.jar) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain)
WARNING: Please consider reporting this to the maintainers of net.sf.cglib.core.ReflectUtils$1
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
[INFO] Sensor JavaSquidSensor [java]
[INFO] Configured Java source version (sonar.java.source): 11
[INFO] JavaClasspath initialization
[INFO] JavaClasspath initialization (done) | time=9ms
[INFO] JavaTestClasspath initialization
[INFO] JavaTestClasspath initialization (done) | time=0ms
[INFO] Java Main Files AST scan
[INFO] 1 source files to be analyzed
[INFO] Load project repositories
[INFO] Load project repositories (done) | time=7ms
[INFO] Java Main Files AST scan (done) | time=485ms
[INFO] Java Test Files AST scan
[INFO] 1 source files to be analyzed
[INFO] 1/1 source files have been analyzed
[INFO] Java Test Files AST scan (done) | time=95ms
[INFO] Sensor JavaSquidSensor [java] (done) | time=1161ms
[INFO] Sensor JaCoCo XML Report Importer [jacoco]
[INFO] Sensor JaCoCo XML Report Importer [jacoco] (done) | time=2ms
[INFO] Sensor SurefireSensor [java]
[INFO] parsing [/home/vagrant/maven-hello/target/surefire-reports]
[INFO] 1/1 source files have been analyzed
[INFO] Sensor SurefireSensor [java] (done) | time=27ms
[INFO] Sensor JaCoCoSensor [java]
[INFO] Sensor JaCoCoSensor [java] (done) | time=0ms
[INFO] Sensor JavaXmlSensor [java]
[INFO] 1 source files to be analyzed
[INFO] Sensor JavaXmlSensor [java] (done) | time=129ms
[INFO] Sensor HTML [web]
[INFO] Sensor HTML [web] (done) | time=2ms
[INFO] Sensor XML Sensor [xml]
[INFO] 1 source files to be analyzed
[INFO] 1/1 source files have been analyzed
[INFO] Sensor XML Sensor [xml] (done) | time=116ms
[INFO] ------------- Run sensors on project
[INFO] Sensor Zero Coverage Sensor
[INFO] Sensor Zero Coverage Sensor (done) | time=6ms
[INFO] Sensor Java CPD Block Indexer
[INFO] 1/1 source files have been analyzed
[INFO] Sensor Java CPD Block Indexer (done) | time=16ms
[INFO] SCM Publisher No SCM system was detected. You can use the 'sonar.scm.provider' property to explicitly specify it.
[INFO] CPD Executor 1 file had no CPD blocks
[INFO] CPD Executor Calculating CPD for 0 files
[INFO] CPD Executor CPD calculation finished (done) | time=0ms
[INFO] Analysis report generated in 83ms, dir size=82 KB
[INFO] Analysis report compressed in 12ms, zip size=14 KB
[INFO] Analysis report uploaded in 188ms
[INFO] ANALYSIS SUCCESSFUL, you can browse http://10.0.0.13:8000/sonarqube/dashboard?id=com.itranswarp.learnjava%3Amaven-hello
[INFO] Note that you will be able to access the updated dashboard once the server has processed the submitted analysis report
[INFO] More about the report processing at http://10.0.0.13:8000/sonarqube/api/ce/task?id=AW-fiJW8InTTnVLhDjNd
[INFO] Analysis total time: 4.354 s
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  03:34 min
[INFO] Finished at: 2020-01-13T15:32:06Z
[INFO] ------------------------------------------------------------------------
[vagrant@workspace maven-hello]$ 
~~~

生成了目标文件

~~~
[vagrant@workspace maven-hello]$ ls
pom.xml  src  target
[vagrant@workspace maven-hello]$ tree 
.
├── pom.xml
├── src
│   ├── main
│   │   └── java
│   │       └── com
│   │           └── itranswarp
│   │               └── learnjava
│   │                   └── Main.java
│   └── test
│       └── java
│           └── com
│               └── itranswarp
│                   └── learnjava
│                       └── MainTest.java
└── target
    ├── classes
    │   └── com
    │       └── itranswarp
    │           └── learnjava
    │               └── Main.class
    ├── generated-sources
    │   └── annotations
    ├── generated-test-sources
    │   └── test-annotations
    ├── maven-archiver
    │   └── pom.properties
    ├── maven-hello-1.0-SNAPSHOT.jar
    ├── maven-status
    │   └── maven-compiler-plugin
    │       ├── compile
    │       │   └── default-compile
    │       │       ├── createdFiles.lst
    │       │       └── inputFiles.lst
    │       └── testCompile
    │           └── default-testCompile
    │               ├── createdFiles.lst
    │               └── inputFiles.lst
    ├── sonar
    │   └── report-task.txt
    ├── surefire-reports
    │   ├── com.itranswarp.learnjava.MainTest.txt
    │   └── TEST-com.itranswarp.learnjava.MainTest.xml
    └── test-classes
        └── com
            └── itranswarp
                └── learnjava
                    └── MainTest.class

33 directories, 14 files
[vagrant@workspace maven-hello]$ 
~~~

## 查看扫描结果

在 Project 中可以看到所有项目的汇总信息

![sonarqube](/assets/img/sonarqube/sonarqube05.png)

进入指定项目，查看详情

![sonarqube](/assets/img/sonarqube/sonarqube06.png)

![sonarqube](/assets/img/sonarqube/sonarqube07.png)

这里可以直接看到代码

![sonarqube](/assets/img/sonarqube/sonarqube08.png)

## 再次运行检查

~~~
[vagrant@workspace maven-hello]$ mvn sonar:sonar 
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------< com.itranswarp.learnjava:maven-hello >----------------
[INFO] Building hello 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- sonar-maven-plugin:3.7.0.1746:sonar (default-cli) @ maven-hello ---
[INFO] User cache: /home/vagrant/.sonar/cache
[INFO] SonarQube version: 8.1.0
[INFO] Default locale: "en_US", source code encoding: "UTF-8"
[INFO] Load global settings
[INFO] Load global settings (done) | time=83ms
[INFO] Server id: 5ACACF06-AW-QG8czf4dfML9SHoJZ
[INFO] User cache: /home/vagrant/.sonar/cache
[INFO] Load/download plugins
[INFO] Load plugins index
[INFO] Load plugins index (done) | time=57ms
[INFO] Load/download plugins (done) | time=133ms
[INFO] Process project properties
[INFO] Process project properties (done) | time=9ms
[INFO] Execute project builders
[INFO] Execute project builders (done) | time=6ms
[INFO] Project key: com.itranswarp.learnjava:maven-hello
[INFO] Base dir: /home/vagrant/maven-hello
[INFO] Working dir: /home/vagrant/maven-hello/target/sonar
[INFO] Load project settings for component key: 'com.itranswarp.learnjava:maven-hello'
[INFO] Load project settings for component key: 'com.itranswarp.learnjava:maven-hello' (done) | time=17ms
[INFO] Load quality profiles
[INFO] Load quality profiles (done) | time=48ms
[INFO] Load active rules
[INFO] Load active rules (done) | time=594ms
[WARNING] SCM provider autodetection failed. Please use "sonar.scm.provider" to define SCM of your project, or disable the SCM Sensor in the project settings.
[INFO] Indexing files...
[INFO] Project configuration:
[INFO] 3 files indexed
[INFO] Quality profile for java: Sonar way
[INFO] Quality profile for xml: Sonar way
[INFO] ------------- Run sensors on module hello
[INFO] Load metrics repository
[INFO] Load metrics repository (done) | time=17ms
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by net.sf.cglib.core.ReflectUtils$1 (file:/home/vagrant/.sonar/cache/a2b6a8802525720c8af2ca4fa85a2513/sonar-javascript-plugin.jar) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain)
WARNING: Please consider reporting this to the maintainers of net.sf.cglib.core.ReflectUtils$1
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
[INFO] Sensor JavaSquidSensor [java]
[INFO] Configured Java source version (sonar.java.source): 11
[INFO] JavaClasspath initialization
[INFO] JavaClasspath initialization (done) | time=9ms
[INFO] JavaTestClasspath initialization
[INFO] JavaTestClasspath initialization (done) | time=0ms
[INFO] Java Main Files AST scan
[INFO] 1 source files to be analyzed
[INFO] Load project repositories
[INFO] Load project repositories (done) | time=17ms
[INFO] Java Main Files AST scan (done) | time=468ms
[INFO] Java Test Files AST scan
[INFO] 1 source files to be analyzed
[INFO] 1/1 source files have been analyzed
[INFO] 1/1 source files have been analyzed
[INFO] Java Test Files AST scan (done) | time=100ms
[INFO] Sensor JavaSquidSensor [java] (done) | time=1078ms
[INFO] Sensor JaCoCo XML Report Importer [jacoco]
[INFO] Sensor JaCoCo XML Report Importer [jacoco] (done) | time=2ms
[INFO] Sensor SurefireSensor [java]
[INFO] parsing [/home/vagrant/maven-hello/target/surefire-reports]
[INFO] Sensor SurefireSensor [java] (done) | time=29ms
[INFO] Sensor JaCoCoSensor [java]
[INFO] Sensor JaCoCoSensor [java] (done) | time=1ms
[INFO] Sensor JavaXmlSensor [java]
[INFO] 1 source files to be analyzed
[INFO] Sensor JavaXmlSensor [java] (done) | time=133ms
[INFO] Sensor HTML [web]
[INFO] 1/1 source files have been analyzed
[INFO] Sensor HTML [web] (done) | time=3ms
[INFO] Sensor XML Sensor [xml]
[INFO] 1 source files to be analyzed
[INFO] Sensor XML Sensor [xml] (done) | time=106ms
[INFO] ------------- Run sensors on project
[INFO] Sensor Zero Coverage Sensor
[INFO] 1/1 source files have been analyzed
[INFO] Sensor Zero Coverage Sensor (done) | time=15ms
[INFO] Sensor Java CPD Block Indexer
[INFO] Sensor Java CPD Block Indexer (done) | time=9ms
[INFO] SCM Publisher No SCM system was detected. You can use the 'sonar.scm.provider' property to explicitly specify it.
[INFO] CPD Executor 1 file had no CPD blocks
[INFO] CPD Executor Calculating CPD for 0 files
[INFO] CPD Executor CPD calculation finished (done) | time=0ms
[INFO] Analysis report generated in 67ms, dir size=82 KB
[INFO] Analysis report compressed in 21ms, zip size=14 KB
[INFO] Analysis report uploaded in 22ms
[INFO] ANALYSIS SUCCESSFUL, you can browse http://10.0.0.13:8000/sonarqube/dashboard?id=com.itranswarp.learnjava%3Amaven-hello
[INFO] Note that you will be able to access the updated dashboard once the server has processed the submitted analysis report
[INFO] More about the report processing at http://10.0.0.13:8000/sonarqube/api/ce/task?id=AW-fpVu6InTTnVLhDjNy
[INFO] Analysis total time: 4.023 s
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  5.567 s
[INFO] Finished at: 2020-01-13T16:03:31Z
[INFO] ------------------------------------------------------------------------
[vagrant@workspace maven-hello]$ 
~~~

可以看到多次运行的历史记录

![sonarqube](/assets/img/sonarqube/sonarqube09.png)


到此 SonarQube 与 Maven 的集成就演示完毕了

---

# 总结

一个敏捷开发团队一定会需要一套静态代码扫描软件

**[SonarQube][sonarqube]** 就是这样的软件

上面的步骤描述了 **[SonarQube][sonarqube]** 与 Maven 的集成过程

后面接着讲如何与 SVM 集成与 Jenkins 集成 

* TOC
{:toc}


---

[sonarqube]:https://www.sonarqube.org/
[sonarscanner_for_maven]:https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-maven/