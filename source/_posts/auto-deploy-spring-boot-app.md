---
title: 如何在Ubuntu 14.04服務器上自動化部署Spring Boot的應用
date: 2015-10-01 00:00:00
tags:
  - Spring
  - Linux
---

这篇文章主要讲解我在Ubuntu 14.04服务器上如何部署Spring Boot的网站应用的经验。以下以我的博客项目 https://github.com/Raysmond.com/SpringBlog 为例子。主要的部署需求如下：

如何在服务器上运行Spring Boot的应用，并使用production环境的配置文件application-production.yml；
如何通过Gradle构建和发布Spring Boot的项目；
如何在本地自动化部署网站到production环境中。

## 1. 在Ubuntu中安装Java8

以下是一个简单的安装方法。

```
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer

$ java -version
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
```

<!-- more -->

## 2. 本地使用Gradle发布Spring Boot应用

我这里使用Jetty9作为内置的服务器。

```
// ...
bootRun {
    systemProperties = System.properties
}

configurations {
    compile.exclude module: "spring-boot-starter-tomcat"
}

dependencies {
    // spring boot
    compile "org.springframework.boot:spring-boot-starter-web:1.3.0.M5"
    compile "org.springframework.boot:spring-boot-starter-jetty"
    // ...
}

//...
```

在本地运行默认使用src/main/resources/application.yml作为配置文件，而在production环境中我们系统它支持提供外部的配置文件application-production.yml。

```
./gradlew bootRun # 开发环境下默认使用项目里的application.yml

# 在本地测试使用外部配置文件
./gradlew bootRun -Dspring.config.location=/path/to/application-production.yml

# 发布
./gradlew build

# 运行
java -jar build/libs/SpringBlog-0.1.jar  # 默认使用jar包里面的application.yml配置文件

# 使用外部配置文件
java -jar build/libs/SpringBlog-0.1.jar --spring.config.location=/path/to/application-production.yml
```

## 3. 在Ubuntu服务器上部署Spring Boot应用

```
# 上传SpringBlog-0.1.jar到服务器
scp build/libs/SpringBlog-0.1.jar root@your_server_ip:/root/spring-blog/current

# 在服务器上配置生产环境的配置文件
scp application-production.yml root@your_server_ip:/root/spring-blog/current
```

然后SSH登录服务器，修改配置文件application-production.yml，试运行应用。

```
ssh root@your_server_ip

cd spring-blog/current

java -jar SpringBlog-0.1.jar --spring.config.location=application-production.yml
```

```
# application-production.yml

server:
    address: raysmond.com # 使用域名或者IP，启动之后就可以这个域名或IP访问网站了
    port: 80
    contextPath:

spring:
    profiles:
        active: production

    thymeleaf:
        cache: true

    jade4j:
        caching: true

    dataSource:
      driverClassName: com.mysql.jdbc.Driver
      url: jdbc:mysql://127.0.0.1/spring_blog
      username: root
      password:

    hibernate:
      dialect: org.hibernate.dialect.MySQLDialect
      hbm2ddl.auto: update
      show_sql: false

    redis:
      host: localhost
      port: 6379

```

## 4. 如何在Ubuntu中后台运行Spring Boot应用？

推荐使用nohup这个命令。

```
cd /root/spring-blog/current
nohup java -jar SpringBlog-0.1.jar --spring.config.location=application-production.yml \
> ../logs/production.log 2> ../logs/production.err &
```

在Ubuntu还可以/etc/init.d目录下新建一个脚本，把SpringBlog作为service来运行，这样不用每次都打这么繁琐的命令了。新建一个/etc/init.d/spring_blog文件，内容如下：

```
#!/bin/sh
SERVICE_NAME=spring_blog
HOME=/root/spring-blog
PATH_TO_JAR=$HOME/current/SpringBlog-0.1.jar
PID_PATH_NAME=/tmp/spring_blog.pid
LOG=$HOME/logs/production.log
ERROR_LOG=$HOME/logs/production.err
CONFIG=$HOME/application-production.yml
case $1 in
    start)
        echo "Starting $SERVICE_NAME ..."
        if [ ! -f $PID_PATH_NAME ]; then
            cd $HOME/current
            nohup java -jar $PATH_TO_JAR --spring.config.location=application-production.yml > $LOG 2> $ERROR_LOG &
                        echo $! > $PID_PATH_NAME
            echo "$SERVICE_NAME started ..."
        else
            echo "$SERVICE_NAME is already running ..."
        fi
    ;;
    stop)
        if [ -f $PID_PATH_NAME ]; then
            PID=$(cat $PID_PATH_NAME);
            echo "$SERVICE_NAME stoping ..."
            kill $PID;
            echo "$SERVICE_NAME stopped ..."
            rm $PID_PATH_NAME
        else
            echo "$SERVICE_NAME is not running ..."
        fi
    ;;
    restart)
        if [ -f $PID_PATH_NAME ]; then
            PID=$(cat $PID_PATH_NAME);
            echo "$SERVICE_NAME stopping ...";
            kill $PID;
            echo "$SERVICE_NAME stopped ...";
            rm $PID_PATH_NAME
            echo "$SERVICE_NAME starting ..."
            cd $HOME/current
            nohup java -jar $PATH_TO_JAR --spring.config.location=application-production.yml > $LOG 2> $ERROR_LOG &
                        echo $! > $PID_PATH_NAME
            echo "$SERVICE_NAME started ..."
        else
            echo "$SERVICE_NAME is not running ..."
        fi
    ;;
esac
```

现在就可以使用service的方式来运行网站了。

```
sudo service spring_blog start

sudo service spring_blog stop

sudo service spring_blog restart
```

## 5. 在本地自动化部署网站到远程服务器

在本地我用了一个shell脚本和一个python脚本来配合自动化部署。

- deploy.sh 使用gradle的命令发布jar包，使用scp命令吧jar包上传到服务器上；
- deploy.py 使用SSH远程登录服务器，并在服务器上执行部署命令。

```
# deploy.sh

#!/bin/bash

SERVER="your_server_ip"
JAR="build/libs/SpringBlog-0.1.jar"

echo "Building $JAR..."
./gradlew build

echo "Upload $JAR to server $SERVER..."
scp $JAR root@$SERVER:/root/spring-blog/

python deploy.py
```

deploy.py主要使用了一个paramiko库，用于SSH远程登录服务器，并执行命令。这个脚本会把服务器上/root/spring-blog/current/SpringBlog-0.1.jar备份到/root/spring-blog/releases中，并把新发布的jar包放到/root/spring-blog/current中，然后重启spring_blog服务。

```
#!/usr/bin/python

import paramiko
import threading
import time

ip = 'your_server_ip'
user = 'root'
password = ''
jar = 'SpringBlog-0.1.jar'
home='/root/spring-blog'
current=home+"/current"
releases=home+"/releases"

def execute_cmds(ip, user, passwd, cmd):
    try:
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect(ip,22,user,passwd,timeout=5)
        for m in cmd:
            print m
            stdin, stdout, stderr = ssh.exec_command(m)
#           stdin.write("Y")
            out = stdout.readlines()
            for o in out:
                print o,
        print '%s\tOK\n'%(ip)
        ssh.close()
    except :
        print '%s\tError\n'%(ip)


if __name__=='__main__':
    print 'Start deploying %s to server %s'%(jar, ip)

    now = time.strftime("%Y%m%d%H%M%S")
    cmd = [
        'echo Stop spring_blog service... && service spring_blog stop',
        'echo Flush all redis cache data... && redis-cli -r 1 flushall',
        'echo Stop redis server... && service redis_6379 stop',
        'echo Use new jar... ' + \
        ' && mv ' + current + '/' + jar + ' ' + releases + '/' + now + '_' + jar ,
        'mv ' + home + '/' + jar + ' ' + current + '/' + jar,
        'echo Stop redis... && service redis_6379 start',
        'echo Start spring_blog service... && service spring_blog start ' + \
        ' && echo All done.'
    ]
    a=threading.Thread(target=execute_cmds, args=(ip,user,password,cmd))
    a.start()
```

配置完以后，在本地写完代码就可以运行`./deploy.sh`一键部署到远程服务器了。

以上就是我部署 http://raysmond.com 网站的一些经验。项目源代码已开源放在Github上：https://github.com/Raysmond/SpringBlog。
