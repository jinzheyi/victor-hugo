+++
title = "Linux系统安装jdk8"
date = "2022-02-09 14:21:52"
url = "archives/2"
tags = ["Linux"]
categories = ["运维","Linux环境部署"]
featuredImage = "![jdk8.jpeg](http://121.43.32.181:9000/blog/images/20220212/1bc6bd7d4afd4e918abaa7456e5296c5.png)"

+++

### 使用解压命令解压 ###

[root@localhost local]# tar -zxvf jdk-8u181-linux-x64.tar.gz

### 顺手删掉jdk源码包 ###

[root@localhost local]# rm -f jdk-8u181-linux-x64.tar.gz


/etc/profile文件的改变会涉及到系统的环境，也就是有关Linux环境变量的东西

所以，我们要将jdk配置到/etc/profile，才可以在任何一个目录访问jdk

[root@localhost local]# vim /etc/profile


按i进入编辑，在profile文件尾部添加如下内容
```
export JAVA_HOME=/usr/local/jdk1.8.0_181  #jdk安装目录
 
export JRE_HOME=${JAVA_HOME}/jre
 
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
 
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
 
export PATH=$PATH:${JAVA_PATH}
```

![配置图](http://121.43.32.181:9000/blog/images/20220212/29ba0de3ae3d450a9d1e08ffaec6a927.png)

Esc --> :wq

保存并退出编辑

通过命令source /etc/profile让profile文件立即生效

[root@localhost local]# source /etc/profile



