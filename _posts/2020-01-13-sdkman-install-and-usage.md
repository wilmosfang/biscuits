---
layout: post
title: "SDKMAN Install And Usage"
date: 2020-01-13 04:04:50
image: '/assets/img/'
description: 'SDKMAN 的安装和使用'
main-class: tools
color: 
tags: 
 - sdkman
 - tools
 - java
categories: 
 - sdkman
twitter_text: 'simple process of sdkman installation'
introduction: 'installation method of sdkman'
---

# 前言


**[SDKMAN][sdkman]** 是一个在类 Unix 平台上的多版本管理工具

>SDKMAN! is a tool for managing parallel versions of multiple Software Development Kits on most Unix based systems. It provides a convenient Command Line Interface (CLI) and API for installing, switching, removing and listing Candidates. Formerly known as GVM the Groovy enVironment Manager, it was inspired by the very useful RVM and rbenv tools, used at large by the Ruby community

它参考 RVM 的工作机制对 SDK 的多版本运行环境进行管理

主要具备以下特性


* 为开发人员简化管理: 集成了下载解压，HOME PATH 的环境变量修改
* 多平台: 通行于类 Unix 平台之上， Mac OSX, Linux, Cygwin, Solaris and FreeBSD.
* 管理 Java 的很多项目版本: Install Software Development Kits for the JVM such as Java, Groovy, Scala, Kotlin and Ceylon. Ant, Gradle, Grails, Maven, SBT, Spark, Spring Boot, Vert.x and many others also supported.
* 轻量: 使用 Bash 编写的，只依赖 curl 和 zip/unxip, ZSH 上也可以正常工作
* 安装简单: 只用执行以下命令就可以安装  **`curl -s "https://get.sdkman.io" | bash`**


这里分享一下 **[SDKMAN][sdkman]** 的安装和简单使用

参考 **[Install the SDKMAN][sdkman_install]**

> **Tip:** 当前版本 **SDKMAN 5.7.4+362** 

---

# 操作

## 系统环境

~~~
➜  ~ sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.2
BuildVersion:	19C57
➜  ~
~~~

## 安装 sdkman

~~~
➜  ~ curl -s "https://get.sdkman.io" | bash

                                -+syyyyyyys:
                            `/yho:`       -yd.
                         `/yh/`             +m.
                       .oho.                 hy                          .`
                     .sh/`                   :N`                `-/o`  `+dyyo:.
                   .yh:`                     `M-          `-/osysoym  :hs` `-+sys:      hhyssssssssy+
                 .sh:`                       `N:          ms/-``  yy.yh-      -hy.    `.N-````````+N.
               `od/`                         `N-       -/oM-      ddd+`     `sd:     hNNm        -N:
              :do`                           .M.       dMMM-     `ms.      /d+`     `NMMs       `do
            .yy-                             :N`    ```mMMM.      -      -hy.       /MMM:       yh
          `+d+`           `:/oo/`       `-/osyh/ossssssdNMM`           .sh:         yMMN`      /m.
         -dh-           :ymNMMMMy  `-/shmNm-`:N/-.``   `.sN            /N-         `NMMy      .m/
       `oNs`          -hysosmMMMMydmNmds+-.:ohm           :             sd`        :MMM/      yy
      .hN+           /d:    -MMMmhs/-.`   .MMMh   .ss+-                 `yy`       sMMN`     :N.
     :mN/           `N/     `o/-`         :MMMo   +MMMN-         .`      `ds       mMMh      do
    /NN/            `N+....--:/+oooosooo+:sMMM:   hMMMM:        `my       .m+     -MMM+     :N.
   /NMo              -+ooooo+/:-....`...:+hNMN.  `NMMMd`        .MM/       -m:    oMMN.     hs
  -NMd`                                    :mm   -MMMm- .s/     -MMm.       /m-   mMMd     -N.
 `mMM/                                      .-   /MMh. -dMo     -MMMy        od. .MMMs..---yh
 +MMM.                                           sNo`.sNMM+     :MMMM/        sh`+MMMNmNm+++-
 mMMM-                                           /--ohmMMM+     :MMMMm.       `hyymmmdddo
 MMMMh.                  ````                  `-+yy/`yMMM/     :MMMMMy       -sm:.``..-:-.`
 dMMMMmo-.``````..-:/osyhddddho.           `+shdh+.   hMMM:     :MmMMMM/   ./yy/` `:sys+/+sh/
 .dMMMMMMmdddddmmNMMMNNNNNMMMMMs           sNdo-      dMMM-  `-/yd/MMMMm-:sy+.   :hs-      /N`
  `/ymNNNNNNNmmdys+/::----/dMMm:          +m-         mMMM+ohmo/.` sMMMMdo-    .om:       `sh
     `.-----+/.`       `.-+hh/`         `od.          NMMNmds/     `mmy:`     +mMy      `:yy.
           /moyso+//+ossso:.           .yy`          `dy+:`         ..       :MMMN+---/oys:
         /+m:  `.-:::-`               /d+                                    +MMMMMMMNh:`
        +MN/                        -yh.                                     `+hddhy+.
       /MM+                       .sh:
      :NMo                      -sh/
     -NMs                    `/yy:
    .NMy                  `:sh+.
   `mMm`               ./yds-
  `dMMMmyo:-.````.-:oymNy:`
  +NMMMMMMMMMMMMMMMMms:`
    -+shmNMMMNmdy+:`


                                                                 Now attempting installation...


Looking for a previous installation of SDKMAN...
Looking for unzip...
Looking for zip...
Looking for curl...
Looking for sed...
Installing SDKMAN scripts...
Create distribution directories...
Getting available candidates...
Prime the config file...
Download script archive...
######################################################################## 100.0%
Extract script archive...
Install scripts...
Set version to 5.7.4+362 ...
Attempt update of login bash profile on OSX...
Added sdkman init snippet to /Users/zheng.fang/.bash_profile
Attempt update of zsh profile...
Updated existing /Users/zheng.fang/.zshrc



All done!


Please open a new terminal, or run the following in the existing one:

    source "/Users/zheng.fang/.sdkman/bin/sdkman-init.sh"

Then issue the following command:

    sdk help

Enjoy!!!
➜  ~
~~~

这时，因为环境变量没有发生变化，所以还无法识别这个命令

~~~
➜  ~ sdk help 
zsh: command not found: sdk
➜  ~ ls $HOME/.sdkman/bin/sdkman-init.sh
/Users/zheng.fang/.sdkman/bin/sdkman-init.sh
➜  ~ wc $HOME/.sdkman/bin/sdkman-init.sh
     133     434    3627 /Users/zheng.fang/.sdkman/bin/sdkman-init.sh
➜  ~ cat $HOME/.sdkman/bin/sdkman-init.sh
#!/usr/bin/env bash

#
#   Copyright 2017 Marco Vermeulen
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.
#

# set env vars if not set
if [ -z "$SDKMAN_VERSION" ]; then
	export SDKMAN_VERSION="5.7.4+362"
fi

if [ -z "$SDKMAN_CANDIDATES_API" ]; then
	export SDKMAN_CANDIDATES_API="https://api.sdkman.io/2"
fi

if [ -z "$SDKMAN_DIR" ]; then
	export SDKMAN_DIR="$HOME/.sdkman"
fi

# infer platform
SDKMAN_PLATFORM="$(uname)"
if [[ "$SDKMAN_PLATFORM" == 'Linux' ]]; then
	if [[ "$(uname -m)" == 'i686' ]]; then
		SDKMAN_PLATFORM+='32'
	else
		SDKMAN_PLATFORM+='64'
	fi
fi
export SDKMAN_PLATFORM

# OS specific support (must be 'true' or 'false').
cygwin=false
darwin=false
solaris=false
freebsd=false
case "${SDKMAN_PLATFORM}" in
	CYGWIN*)
		cygwin=true
		;;
	Darwin*)
		darwin=true
		;;
	SunOS*)
		solaris=true
		;;
	FreeBSD*)
		freebsd=true
esac

# Determine shell
zsh_shell=false
bash_shell=false

if [[ -n "$ZSH_VERSION" ]]; then
	zsh_shell=true
else
	bash_shell=true
fi

# Source sdkman module scripts and extension files.
#
# Extension files are prefixed with 'sdkman-' and found in the ext/ folder.
# Use this if extensions are written with the functional approach and want
# to use functions in the main sdkman script. For more details, refer to
# <https://github.com/sdkman/sdkman-extensions>.
OLD_IFS="$IFS"
IFS=$'\n'
scripts=($(find "${SDKMAN_DIR}/src" "${SDKMAN_DIR}/ext" -type f -name 'sdkman-*'))
for f in "${scripts[@]}"; do
	source "$f"
done
IFS="$OLD_IFS"
unset scripts f

# Load the sdkman config if it exists.
if [ -f "${SDKMAN_DIR}/etc/config" ]; then
	source "${SDKMAN_DIR}/etc/config"
fi

# Create upgrade delay file if it doesn't exist
if [[ ! -f "${SDKMAN_DIR}/var/delay_upgrade" ]]; then
	touch "${SDKMAN_DIR}/var/delay_upgrade"
fi

# set curl connect-timeout and max-time
if [[ -z "$sdkman_curl_connect_timeout" ]]; then sdkman_curl_connect_timeout=7; fi
if [[ -z "$sdkman_curl_max_time" ]]; then sdkman_curl_max_time=10; fi

# set curl retry
if [[ -z "${sdkman_curl_retry}" ]]; then sdkman_curl_retry=0; fi

# set curl retry max time in seconds
if [[ -z "${sdkman_curl_retry_max_time}" ]]; then sdkman_curl_retry_max_time=60; fi

# set curl to continue downloading automatically
if [[ -z "${sdkman_curl_continue}" ]]; then sdkman_curl_continue=true; fi

# Read list of candidates and set array
SDKMAN_CANDIDATES_CACHE="${SDKMAN_DIR}/var/candidates"
SDKMAN_CANDIDATES_CSV=$(<"$SDKMAN_CANDIDATES_CACHE")
__sdkman_echo_debug "Setting candidates csv: $SDKMAN_CANDIDATES_CSV"
if [[ "$zsh_shell" == 'true' ]]; then
	SDKMAN_CANDIDATES=(${(s:,:)SDKMAN_CANDIDATES_CSV})
else
	OLD_IFS="$IFS"
	IFS=","
        SDKMAN_CANDIDATES=(${SDKMAN_CANDIDATES_CSV})
	IFS="$OLD_IFS"
fi

export SDKMAN_CANDIDATES_DIR="${SDKMAN_DIR}/candidates"

for candidate_name in "${SDKMAN_CANDIDATES[@]}"; do
	candidate_dir="${SDKMAN_CANDIDATES_DIR}/${candidate_name}/current"
	if [[ -h "$candidate_dir" || -d "${candidate_dir}" ]]; then
		__sdkman_export_candidate_home "$candidate_name" "$candidate_dir"
		__sdkman_prepend_candidate_to_path "$candidate_dir"
	fi
done
unset OLD_IFS candidate_name candidate_dir
export PATH
➜  ~ 
~~~

## 初始环境变量

~~~
➜  ~ source "$HOME/.sdkman/bin/sdkman-init.sh"
➜  ~ sdk help 
==== BROADCAST =================================================================
* 2020-01-08: Asciidoctorj 2.2.0 released on SDKMAN! #asciidoctorj
* 2020-01-07: Gradle 6.1-rc-2 released on SDKMAN! #gradle
* 2020-01-06: Jbang 0.4.0.1 released on SDKMAN! #jbang
================================================================================

Usage: sdk <command> [candidate] [version]
       sdk offline <enable|disable>

   commands:
       install   or i    <candidate> [version] [local-path]
       uninstall or rm   <candidate> <version>
       list      or ls   [candidate]
       use       or u    <candidate> <version>
       default   or d    <candidate> [version]
       current   or c    [candidate]
       upgrade   or ug   [candidate]
       version   or v
       broadcast or b
       help      or h
       offline           [enable|disable]
       selfupdate        [force]
       update
       flush             <broadcast|archives|temp>

   candidate  :  the SDK to install: groovy, scala, grails, gradle, kotlin, etc.
                 use list command for comprehensive list of candidates
                 eg: $ sdk list
   version    :  where optional, defaults to latest stable if not provided
                 eg: $ sdk install groovy
   local-path :  optional path to an existing local installation
                 eg: $ sdk install groovy 2.4.13-local /opt/groovy-2.4.13

➜  ~ sdk version 

SDKMAN 5.7.4+362
➜  ~ 
~~~


## 基本用法

参考 **[Usage][sdkman_usage]** 

### 列出 java 可用版本

下面是系统自带的 java 版本

~~~
➜  ~ java -version
java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
➜  ~ 
~~~

这里分别可以看到版本，发布方，状态和安装标识

~~~
➜  ~ sdk list java 
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
               |     | 8.0.202      | amzn    |            | 8.0.202-amzn        
 Azul Zulu     |     | 13.0.1       | zulu    |            | 13.0.1-zulu         
               |     | 12.0.2       | zulu    |            | 12.0.2-zulu         
               |     | 11.0.5       | zulu    |            | 11.0.5-zulu         
               |     | 10.0.2       | zulu    |            | 10.0.2-zulu         
               |     | 9.0.7        | zulu    |            | 9.0.7-zulu          
               |     | 8.0.232      | zulu    |            | 8.0.232-zulu        
               |     | 8.0.202      | zulu    |            | 8.0.202-zulu        
               |     | 7.0.181      | zulu    |            | 7.0.181-zulu        
 Azul ZuluFX   |     | 11.0.2       | zulufx  |            | 11.0.2-zulufx       
               |     | 8.0.202      | zulufx  |            | 8.0.202-zulufx      
 BellSoft      |     | 13.0.1       | librca  |            | 13.0.1-librca       
               |     | 12.0.2       | librca  |            | 12.0.2-librca       
               |     | 11.0.5       | librca  |            | 11.0.5-librca       
               |     | 8.0.232      | librca  |            | 8.0.232-librca      
 GraalVM       |     | 19.3.0.r11   | grl     |            | 19.3.0.r11-grl      
               |     | 19.3.0.r8    | grl     |            | 19.3.0.r8-grl       
               |     | 19.3.0.2.r11 | grl     |            | 19.3.0.2.r11-grl    
               |     | 19.3.0.2.r8  | grl     |            | 19.3.0.2.r8-grl     
               |     | 19.2.1       | grl     |            | 19.2.1-grl          
               |     | 19.1.1       | grl     |            | 19.1.1-grl          
               |     | 19.0.2       | grl     |            | 19.0.2-grl          
               |     | 1.0.0        | grl     |            | 1.0.0-rc-16-grl     
 Java.net      |     | 15.ea.4      | open    |            | 15.ea.4-open        
               |     | 14.ea.30     | open    |            | 14.ea.30-open       
               |     | 13.0.1       | open    |            | 13.0.1-open         
               |     | 12.0.2       | open    |            | 12.0.2-open         
               |     | 11.0.2       | open    |            | 11.0.2-open         
               |     | 10.0.2       | open    |            | 10.0.2-open         
               |     | 9.0.4        | open    |            | 9.0.4-open          
 SAP           |     | 12.0.2       | sapmchn |            | 12.0.2-sapmchn      
               |     | 11.0.4       | sapmchn |            | 11.0.4-sapmchn      
================================================================================
Use the Identifier for installation:

    $ sdk install java 11.0.3.hs-adpt
================================================================================
➜  ~ 
~~~


### 安装 java 11


~~~
➜  ~ sdk install java 11.0.2-open

Downloading: java 11.0.2-open

In progress...

######################################################################### 100.0%

Repackaging Java 11.0.2-open...

Done repackaging...
Cleaning up residual files...

Installing: java 11.0.2-open
Done installing!


Setting java 11.0.2-open as default.
➜  ~ 
~~~

再次查看

~~~
➜  ~ sdk ls java 
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
               |     | 8.0.202      | amzn    |            | 8.0.202-amzn        
 Azul Zulu     |     | 13.0.1       | zulu    |            | 13.0.1-zulu         
               |     | 12.0.2       | zulu    |            | 12.0.2-zulu         
               |     | 11.0.5       | zulu    |            | 11.0.5-zulu         
               |     | 10.0.2       | zulu    |            | 10.0.2-zulu         
               |     | 9.0.7        | zulu    |            | 9.0.7-zulu          
               |     | 8.0.232      | zulu    |            | 8.0.232-zulu        
               |     | 8.0.202      | zulu    |            | 8.0.202-zulu        
               |     | 7.0.181      | zulu    |            | 7.0.181-zulu        
 Azul ZuluFX   |     | 11.0.2       | zulufx  |            | 11.0.2-zulufx       
               |     | 8.0.202      | zulufx  |            | 8.0.202-zulufx      
 BellSoft      |     | 13.0.1       | librca  |            | 13.0.1-librca       
               |     | 12.0.2       | librca  |            | 12.0.2-librca       
               |     | 11.0.5       | librca  |            | 11.0.5-librca       
               |     | 8.0.232      | librca  |            | 8.0.232-librca      
 GraalVM       |     | 19.3.0.r11   | grl     |            | 19.3.0.r11-grl      
               |     | 19.3.0.r8    | grl     |            | 19.3.0.r8-grl       
               |     | 19.3.0.2.r11 | grl     |            | 19.3.0.2.r11-grl    
               |     | 19.3.0.2.r8  | grl     |            | 19.3.0.2.r8-grl     
               |     | 19.2.1       | grl     |            | 19.2.1-grl          
               |     | 19.1.1       | grl     |            | 19.1.1-grl          
               |     | 19.0.2       | grl     |            | 19.0.2-grl          
               |     | 1.0.0        | grl     |            | 1.0.0-rc-16-grl     
 Java.net      |     | 15.ea.4      | open    |            | 15.ea.4-open        
               |     | 14.ea.30     | open    |            | 14.ea.30-open       
               |     | 13.0.1       | open    |            | 13.0.1-open         
               |     | 12.0.2       | open    |            | 12.0.2-open         
               |     | 11.0.2       | open    | installed  | 11.0.2-open         
               |     | 10.0.2       | open    |            | 10.0.2-open         
               |     | 9.0.4        | open    |            | 9.0.4-open          
 SAP           |     | 12.0.2       | sapmchn |            | 12.0.2-sapmchn      
               |     | 11.0.4       | sapmchn |            | 11.0.4-sapmchn      
================================================================================
Use the Identifier for installation:

    $ sdk install java 11.0.3.hs-adpt
================================================================================
➜  ~ 
~~~


### 切换 java 版本

~~~
➜  ~ java -version
java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
➜  ~ 
➜  ~ sdk u java 11.0.2-open

Using java version 11.0.2-open in this shell.
➜  ~ java -version
openjdk version "11.0.2" 2019-01-15
OpenJDK Runtime Environment 18.9 (build 11.0.2+9)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.2+9, mixed mode)
➜  ~
~~~

安装 java 13

~~~
➜  maven-hello sdk i java 13.0.1-open 

Downloading: java 13.0.1-open

In progress...

####################################################################################################### 100.0%

Repackaging Java 13.0.1-open...

Done repackaging...
Cleaning up residual files...

Installing: java 13.0.1-open
Done installing!


Setting java 13.0.1-open as default.
➜  maven-hello 
~~~

查看当前版本

~~~
➜  maven-hello sdk ls java            
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
               |     | 8.0.202      | amzn    |            | 8.0.202-amzn        
 Azul Zulu     |     | 13.0.1       | zulu    |            | 13.0.1-zulu         
               |     | 12.0.2       | zulu    |            | 12.0.2-zulu         
               |     | 11.0.5       | zulu    |            | 11.0.5-zulu         
               |     | 10.0.2       | zulu    |            | 10.0.2-zulu         
               |     | 9.0.7        | zulu    |            | 9.0.7-zulu          
               |     | 8.0.232      | zulu    |            | 8.0.232-zulu        
               |     | 8.0.202      | zulu    |            | 8.0.202-zulu        
               |     | 7.0.181      | zulu    |            | 7.0.181-zulu        
 Azul ZuluFX   |     | 11.0.2       | zulufx  |            | 11.0.2-zulufx       
               |     | 8.0.202      | zulufx  |            | 8.0.202-zulufx      
 BellSoft      |     | 13.0.1       | librca  |            | 13.0.1-librca       
               |     | 12.0.2       | librca  |            | 12.0.2-librca       
               |     | 11.0.5       | librca  |            | 11.0.5-librca       
               |     | 8.0.232      | librca  |            | 8.0.232-librca      
 GraalVM       |     | 19.3.0.r11   | grl     |            | 19.3.0.r11-grl      
               |     | 19.3.0.r8    | grl     |            | 19.3.0.r8-grl       
               |     | 19.3.0.2.r11 | grl     |            | 19.3.0.2.r11-grl    
               |     | 19.3.0.2.r8  | grl     |            | 19.3.0.2.r8-grl     
               |     | 19.2.1       | grl     |            | 19.2.1-grl          
               |     | 19.1.1       | grl     |            | 19.1.1-grl          
               |     | 19.0.2       | grl     |            | 19.0.2-grl          
               |     | 1.0.0        | grl     |            | 1.0.0-rc-16-grl     
 Java.net      |     | 15.ea.4      | open    |            | 15.ea.4-open        
               |     | 14.ea.30     | open    |            | 14.ea.30-open       
               | >>> | 13.0.1       | open    | installed  | 13.0.1-open         
               |     | 12.0.2       | open    |            | 12.0.2-open         
               |     | 11.0.2       | open    | installed  | 11.0.2-open         
               |     | 10.0.2       | open    |            | 10.0.2-open         
               |     | 9.0.4        | open    |            | 9.0.4-open          
 SAP           |     | 12.0.2       | sapmchn |            | 12.0.2-sapmchn      
               |     | 11.0.4       | sapmchn |            | 11.0.4-sapmchn      
================================================================================
Use the Identifier for installation:

    $ sdk install java 11.0.3.hs-adpt
================================================================================
➜  maven-hello 
➜  maven-hello 
➜  maven-hello java -version
openjdk version "13.0.1" 2019-10-15
OpenJDK Runtime Environment (build 13.0.1+9)
OpenJDK 64-Bit Server VM (build 13.0.1+9, mixed mode, sharing)
➜  maven-hello 
~~~

切换版本

~~~
➜  maven-hello sdk u java 11.0.2-open 

Using java version 11.0.2-open in this shell.
➜  maven-hello java -version
openjdk version "11.0.2" 2019-01-15
OpenJDK Runtime Environment 18.9 (build 11.0.2+9)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.2+9, mixed mode)
➜  maven-hello
~~~


到此 sdkman 的安装和简单使用就演示完毕


---

# 总结

**[SDKMAN][sdkman]** 可以提升开发人员对于 java 环境的管理效率

是一个居家旅行必备的好工具

* TOC
{:toc}


---


[sdkman]:https://sdkman.io/
[sdkman_install]:https://sdkman.io/install
[sdkman_usage]:https://sdkman.io/usage