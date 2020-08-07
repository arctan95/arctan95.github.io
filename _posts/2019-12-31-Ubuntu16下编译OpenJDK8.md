---
title: Ubuntu16下编译OpenJDK8
layout: post
categories: JAVA
tags: JVM
---
* TOC
{:toc}
最近在读《深入理解Java虚拟机》，看完第一部分想着自己动手编译一套JDK，毕竟想要深入了解JDK内部实现，跟踪调试JDK源码是最便捷的路径。闲话不多说，正文开始。
<!-- more -->
#### 1.获取OpenJDK源码 
我使用的OpenJDK版本：openjdk-8u40-src-b25-10_feb_2015
从官网下载：http://jdk.java.net/java-se-ri/8va-se-ri/8
#### 2. 系统需求
安装一个ubuntu-16.04.6版本(第一次安装了18.04.2版本，但是编译OpenJDK失败了，解决方法为在./hotspot/make/linux/Makefile文件中SUPPORTED_OS_VERSION增加一个4%，但还是Ubuntu16最省心)
从官网下载：https://www.ubuntu.com/download/alternative-downloads
#### 3. 构建编译环境
* 因为OpenJDK的各个组成部分有的是使用C++编译，有的是使用Java自身实现的，所以编译这些Java代码需要一个可用的JDK，官方称这个JDK为“Bootstrap JDK”，所以我使用的版本为：jdk-7u80-linux-x64
 从官网下载(需要注册账号)：https://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html
* OpenJDK源码中使用到Ant脚本，我使用的版本为：apache-ant-1.9.14-bin
从官网下载：https://ant.apache.org/bindownload.cgi
* 配置Bootstrap JDK和Ant环境
```
sudo vi /etc/profile
#打开文件后附加以下内容，路径自行替换。
export ANT_HOME=/home/ubuntu/apache-ant-1.9.14
export JAVA_HOME=/home/ubuntu/jdk1.7.0_80
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$ANT_HOME/bin:$PATH
#退出文件后，运行下面命令
source /etc/profile
```
* 下载依赖包
```
sudo apt-get install libx11-dev libxext-dev libxrender-dev libxtst-dev libxt-dev libcups2-dev libfreetype6-dev libasound2-dev ccache
```
#### 4. 开始编译
* 解压OpenJDK源码并进入目录
```
unzip openjdk-8u40-src-b25-10_feb_2015.zip
cd openjdk/
```
* 运行命令
```
sudo bash ./configure --with-target-bits=64 --with-boot-jdk=/home/ubuntu/jdk1.7.0_80/ --with-debug-level=slowdebug --enable-debug-symbols ZIP_DEBUGINFO_FILES=0
sudo make all DISABLE_HOTSPOT_OS_VERSION_CHECK=OK ZIP_DEBUGINFO_FILES=0
```
```
说下第一条命令configure用到的参数作用：
–with-target-bits=64 ：指定生成64位jdk；
–with-boot-jdk=/home/ubuntu/jdk1.7.0_80/：启动jdk的路径；
–with-debug-level=slowdebug：编译时debug的级别，有release, fastdebug, slowdebug 三种级别；
–enable-debug-symbols ZIP_DEBUGINFO_FILES=0：生成调试的符号信息，并且不压缩；
```
编译完成后如下图（会花费15分钟左右时间） 

![1.png](http://ww1.sinaimg.cn/large/007Ns0Faly1gafy96takkj309a05vjrc.jpg)
#### 5. 验证
* 进入下面目录并运行命令
```
cd build/linux-x86_64-normal-server-slowdebug/images/j2sdk-image/bin/
./java -version
```
输出：
![2.png](http://ww1.sinaimg.cn/large/007Ns0Faly1gafycxkk5fj30oh01jdfs.jpg)
至此，OpenJDK8编译成功！
