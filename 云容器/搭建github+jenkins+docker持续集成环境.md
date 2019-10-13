---
title: 搭建github+jenkins+docker持续集成环境
date: 2018/12/1 23:15:00
updated: 2018/12/1 23:15:00
categories: 
- 云容器
tags: 
- 云容器
- docker
- 开发环境
- CI/CD
---

使用jenkins+github+docker是一个很常用的持续集成、持续交付方案了，这篇文章描述了搭建这套系统的流程，记录了一些坑点。

# 配置github
1. 在'setting'里找到配置SSH key的地方：[https://github.com/settings/keys](https://github.com/settings/keys)
2. 根据[文档](https://help.github.com/en/articles/connecting-to-github-with-ssh)，[新建](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)或[使用一个已有的key](https://help.github.com/en/articles/adding-a-new-ssh-key-to-your-github-account)
3. 在[开发设置(setting-Developer settings-)](https://github.com/settings/tokens)中配置一个带权限的access_token，注意，'repo'和'admin:repo hook'这两项是必选的，这个简单

# 安装docker
1. 阅读官方CentOS[安装文档](https://docs.docker.com/install/linux/docker-ce/centos/)，按步骤装，简单的很

# 安装jdk8
1. 上[java官网](https://www.oracle.com/technetwork/java/javase/downloads/index.html)下一个，然后安装，配环境变量，比上面更简单

```shell
# 下载解压（注意用wget要先点下载，然后把带auth参数的链接放到wget）
$ wget jdk-download-url
$ tar -zxvf jdk-8u60-linux-x64.tar.gz

# 移动目录
$ mkdir /usr/java
$ cp -r jdk1.8.0_201 /usr/java/

# 配置环境变量
$ vim /etc/profile
```

在底部加入如下，注意目录名别配错了：
```
JAVA_HOME=/usr/java/jdk1.8.0_201
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
export PATH JAVA_HOME CLASSPATH
```
运行`source /etc/profile`让配置立即生效

验证安装：`java -version`，`javac`

# 安装Jenkins
1. 去官网[下载](https://jenkins.io/zh/doc/pipeline/tour/getting-started/)
```shell
$ wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
$ java -jar jenkins.war --httpPort=8080
```
然后访问 ip:8080即可
2. 也可以在docker里装，这样就不用什么jdk了
```shell
$ docker pull jenkins/jenkins:lts

# 建个目录，给他权限
$ mkdir /home/jenkins
$ chown -R 1000:1000 jenkins/
$ ls -nd jenkins/

# 给镜像打个标签
$ docker images
$ docker tag jenkins/jenkins:lts jenkins:lts
$ docker images

# 容器跑起来，映射目录
$ docker run -itd -p 8080:8080 -p 50000:50000 --name jenkins --privileged=true -v /home/jenkins:/var/jenkins_home jenkins:lts
```
这时候用 ip:8080 就可以访问了，进去以后设置好初始admin账户密码，插件选推荐就行了。

# 配置Jenkins
1. 左侧'系统管理'-右侧'系统设置'，找github服务器，添加一个，添加凭据，凭据类型选secret text，凭据添刚才github生成的access_token，id不用填，保存，勾上manage hooks，然后点连接测试，通过即可。
2. 新建个任务（job），把配置都填好，shell脚本写好即可。

至此，github+jenkins+docker的环境就搭好了，随后可以在jenkins中执行预先编写好的makefile和dockerfile来打包docker镜像和部署应用，本文不赘述makefile和dockerfile等的具体使用方法。