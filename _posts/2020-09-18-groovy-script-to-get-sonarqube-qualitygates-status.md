---
layout: post
title: "Get Sonarqube Gualitygate status by Groovy"
date: 2020-09-18 20:14:42
image: '/assets/img/'
description: '通过 Groovy 脚本自动获取sonarqube质量门禁结果'
main-class: 'code'
color: '#fbbd04' 
tags:
 - snippet
 - gradle
 - tools
 - jenkins
 - devops
 - sonarqube
 - code
 - groovy
categories:
 - code
twitter_text: "Get Sonarqube Gualitygate status by Groovy"
introduction: "Get Sonarqube Gualitygate status by Groovy"
---

# 前言

开启一个 **[CODE][code]** 系列，用来收集各种好用的小片段

这一篇用来展示如何通过 Groovy 脚本自动获取 sonarqube 质量门禁的结果

> **Tip:** 此次使用的 gradle 版本为 **Gradle 5.2.1**

---

# 操作


## OS 环境

~~~
[root@jenkins tdd_in_jenkins]# hostnamectl
   Static hostname: jenkins
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 03699531e1dc431b849afd1e28341158
           Boot ID: 25fe44e55747483dbe82bf6488f7fbba
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@jenkins tdd_in_jenkins]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84679sec preferred_lft 84679sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:db:e0:72 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.11/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fedb:e072/64 scope link
       valid_lft forever preferred_lft forever
[root@jenkins tdd_in_jenkins]#
~~~


## 依赖

Gradle 

~~~
[root@jenkins tdd_in_jenkins]# gradle --version

------------------------------------------------------------
Gradle 5.2.1
------------------------------------------------------------

Build time:   2019-02-08 19:00:10 UTC
Revision:     f02764e074c32ee8851a4e1877dd1fea8ffb7183

Kotlin DSL:   1.1.3
Kotlin:       1.3.20
Groovy:       2.5.4
Ant:          Apache Ant(TM) version 1.9.13 compiled on July 10 2018
JVM:          1.8.0_242 (Oracle Corporation 25.242-b08)
OS:           Linux 3.10.0-957.12.2.el7.x86_64 amd64

[root@jenkins tdd_in_jenkins]#
~~~


## 代码

~~~
[root@jenkins tdd_in_jenkins]# cat settings.gradle
rootProject.name = 'tdd-gilded-rose'
[root@jenkins tdd_in_jenkins]# cat build.gradle
plugins {
    id 'java'
    id "org.sonarqube" version "2.8"
}

group 'cc.xpbootcamp'
version '1.0-SNAPSHOT'

apply plugin: 'java'
apply plugin: 'idea'
apply plugin: 'jacoco'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

def JUNIT5_API_VERSION = '5.3.0'
def JUNIT5_PLATFORM_VERSION = '1.3.0'

ext.libs = [
        junit4: [
                'junit:junit:4.12',
                'org.mockito:mockito-core:2.19.0'
        ],
        junit5: [
                "org.junit.jupiter:junit-jupiter-api:${JUNIT5_API_VERSION}",
                "org.junit.jupiter:junit-jupiter-engine:${JUNIT5_API_VERSION}",
                "org.junit.vintage:junit-vintage-engine:${JUNIT5_API_VERSION}",
                "org.junit.jupiter:junit-jupiter-params:${JUNIT5_API_VERSION}",
                "org.junit.platform:junit-platform-launcher:${JUNIT5_PLATFORM_VERSION}",
                "org.junit.platform:junit-platform-runner:${JUNIT5_PLATFORM_VERSION}",
                'org.mockito:mockito-junit-jupiter:2.17.0'
        ]
]

test {
    useJUnitPlatform()
    testLogging {
        events "passed", "skipped", "failed"
        afterSuite { desc, result ->
            if (!desc.parent) {
                def output = " Result: ${result.resultType} " +
                        "(${result.testCount} Tests, " +
                        "${result.successfulTestCount} Successes, " +
                        "${result.failedTestCount} Failures, " +
                        "${result.skippedTestCount} Skipped) "
                println('\n' + ('-' * output.length()) + '\n' + output + '\n' + ('-' * output.length()))
            }
        }
    }
}

dependencies {
    testCompile libs.junit5
    testCompile 'org.assertj:assertj-core:3.10.0'
    testCompile 'org.hamcrest:hamcrest-core:2.1'
}

jacocoTestReport {
    reports {
        xml.enabled true
        html.enabled true
    }
}

check.dependsOn jacocoTestReport

def getHttpDate(String url){
    def connection = new URL(url).openConnection()
    def jsonSluper = new groovy.json.JsonSlurper()
    connection.setRequestMethod('GET')
    connection.connect()
    def response = connection.content.text
    return jsonSluper.parseText(response)
}

project.task('checkSonarqubeQualitygate',group:'Verification',description:'Get quality gate status from SonarQube').doLast {
    def COUNT = 3
    def INTERVAL = 10
    def REPORT_PATH = '.scannerwork/report-task.txt'
//    def REPORT_PATH = '/tmp/report-task.txt'
    def project_status_api = 'api/qualitygates/project_status'
    def task_status_url = ''
    def server_url = ''

    File report = new File(REPORT_PATH)
    report.eachLine {
        println it
        if (it.contains('ceTaskUrl')) task_status_url = it - 'ceTaskUrl='
        if (it.contains('serverUrl')) server_url = it - 'serverUrl='
    }

    for(int i = 1; i <= COUNT; i++){
        def task_status = getHttpDate(task_status_url).task.status
        if ( task_status == 'SUCCESS'){
            break
        }
        else
        {
            sleep(INTERVAL*1000)
            println "retry "+i
        }
    }

    def analysisId = getHttpDate(task_status_url).task.analysisId
    def project_status_url = server_url+'/'+project_status_api+'?analysisId='+analysisId
    def project_status = getHttpDate(project_status_url).projectStatus.status

    if ( project_status == 'OK'){
        println "project_status is: " + project_status
    }
    else
    {
        throw new Exception("project_status is: " + project_status);
    }
}

//checkSonarqubeQualitygate.enabled = false
//checkSonarqubeQualitygate.dependsOn taskX
//checkSonarqubeQualitygate.shouldRunAfter taskX
//checkSonarqubeQualitygate.mustRunAfter taskX
[root@jenkins tdd_in_jenkins]# cat sonar-project.properties
sonar.projectKey=my_spike456
[root@jenkins tdd_in_jenkins]# cat Jenkinsfile.groovy
env.GIT_URL="http://10.0.0.222:3000/abc123/tdd.git"
env.BID="${BUILD_ID}"

node{
    stage('checkout'){
	    git url: "${env.GIT_URL}", credentialsId: "abc123_for_jenkins"
        echo "${BUILD_ID} checkout"
    }
    stage('build'){
            sh "${tool 'Gradle 5.2.1'}/bin/gradle --version"
            sh "${tool 'Gradle 5.2.1'}/bin/gradle clean build"
            echo "build No. ${env.BID}"
        }

    stage('SonarQube analysis') {
       def sonarqubeScannerHome = tool name: 'sonarQubeScanner'
        withSonarQubeEnv('sonarqube') {
            sh "${sonarqubeScannerHome}/bin/sonar-scanner -X -Dsonar.host.url=http://10.0.0.222:9000/ -Dproject.settings=sonar-project.properties  -Dsonar.java.binaries=**/classes "
//            sh "${sonarqubeScannerHome}/bin/sonar-scanner -Dsonar.host.url=http://10.0.0.13:8000/sonarqube -Dproject.settings=sonar-project.properties  -Dsonar.java.binaries=**/classes"
//            sh "${tool 'Gradle 5.2.1'}/bin/gradle sonarqube -Dsonar.host.url=http://10.0.0.222:9000/  -Dsonar.verbose=true"
        }
   }

    stage("Quality Gate"){
        sh "${tool 'Gradle 5.2.1'}/bin/gradle checkSonarqubeQualitygate"
    }

}
[root@jenkins tdd_in_jenkins]#
~~~



## 执行

这里我先使用 report **`/tmp/report-task.txt`** 替换实际生产中的 **`.scannerwork/report-task.txt`** 进行一个本地的测试

~~~
[root@jenkins tdd_in_jenkins]# gradle checkSonarqubeQualitygate

> Task :checkSonarqubeQualitygate
projectKey=my_spike123
serverUrl=http://10.0.0.222:9000
serverVersion=8.4.2.36762
dashboardUrl=http://10.0.0.222:9000/dashboard?id=my_spike123
ceTaskId=AXSfLlNNsKLipkeAEbI8
ceTaskUrl=http://10.0.0.222:9000/api/ce/task?id=AXSfLlNNsKLipkeAEbI8
project_status is: OK

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
[root@jenkins tdd_in_jenkins]#
~~~

> **Tip:** 可以定制重试次数 **`COUNT`**, 重试间隔 **`INTERVAL`**, 指定报告路径 **`REPORT_PATH`**
 
参考的逻辑如下：

**https://docs.sonarqube.org/display/SONARQUBE53/Breaking+the+CI+Build**

1. call the analysis's ceTaskUrl and examine the "status" value:

~~~
a. PENDING or IN_PROGRESS - check again later
b. FAILED or CANCELED - break the build?
c. SUCCESS - move forward
~~~

2. call Quality Gate web service for status


* Workflow of WS API Calls

~~~
1. Search in <work_dir>/report-task.txt the values of the CE Task URL (ceTaskUrl) and CE Task Id (ceTaskId)
2. Call /api/ce/task?id=XXX where XXX is the CE Task Id retrieved from <work_dir>/report-task.txt (save it as analysisId somewhere)
3. Do the Step 2 until the status is SUCCESS, CANCELED or FAILED
4. If the status is FAILED => break the build
5. If the status is SUCCESS
    a. Use the analysisId from the JSON returned by /api/ce/task?id=XXX
    b. Immediately call /api/qualitygates/project_status?analysisId=YYY to check the status of the quality gate
~~~

参考的 API 文档为

**https://next.sonarqube.com/sonarqube/web_api/api/qualitygates**

>**Tip:** groovy 有自带的 **URL** 来进行 HTTP 请求, 也有自带的 **JsonSlurper** 解析 json, 因此没有过多的外部依赖

提交到 Jenkins 中看看执行效果

![groovy](/assets/img/post/2020-09-18-groovy-script-to-get-sonarqube-qualitygates-status/groovy01.png)

![groovy](/assets/img/post/2020-09-18-groovy-script-to-get-sonarqube-qualitygates-status/groovy02.png)

通过 job 的 console 日志可以看到

~~~
...
...
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Quality Gate)
[Pipeline] tool
[Pipeline] sh
+ /var/jenkins_home/tools/hudson.plugins.gradle.GradleInstallation/Gradle_5.2.1/bin/gradle checkSonarqubeQualitygate

> Task :checkSonarqubeQualitygate
projectKey=my_spike456
serverUrl=http://10.0.0.222:9000
serverVersion=8.4.2.36762
dashboardUrl=http://10.0.0.222:9000/dashboard?id=my_spike456
ceTaskId=AXShS4F4oE4UAVln8JC7
ceTaskUrl=http://10.0.0.222:9000/api/ce/task?id=AXShS4F4oE4UAVln8JC7
retry 1
project_status is: OK

BUILD SUCCESSFUL in 11s
1 actionable task: 1 executed
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
~~~

通过 **`retry 1`** 这个信息, 我们知道由于 task 状态不 ready，我们重试过一次


---

# 总结

直接在 gradle 中通过 groovy 代码来完成一些 task 可以简化一些操作

例如

~~~
stage("Quality Gate"){
        sh "${tool 'Gradle 5.2.1'}/bin/gradle checkSonarqubeQualitygate"
    }
~~~


* TOC
{:toc}

---

[code]:http://www.atomicgain.com/category/code/
