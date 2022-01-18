---
title: 使用navicat连接云服务器的docker中的mysql数据库过程
date: 2021-12-10 21:34:12
tags: docker mysql navicat
---

## 第一步：在CentOS中安装docker引擎

要安装Docker引擎，需要CentOS 7或CentOS 8的维护版本。

查看CentOs版本：

```
cat  /etc/redhat-release
```

#### 

*使用仓库安装的方法*：

#### 设置仓库

安装`yum-utils`包（提供了`yum-config-manager`工具）并启动一个稳定的仓库

```
 sudo yum install -y yum-utils
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

```

错误：`Invalid configuration value: failovermethod=priority`

解决方法：将 `/etc/yum.repos.d/CentOS-Epel.repo` 文件中 `failovermethod=priority` 注释掉。



#### 安装Docker引擎

1. 安装最新版Docker引擎

   ```
    sudo yum install docker-ce docker-ce-cli containerd.io
   ```

2. 启动Docker

   ```
    sudo systemctl start docker
   ```

3. 运行`hello-world`镜像

   ```
    sudo docker run hello-world
   ```

   如果可以运行，说明Docker引擎安装成功。

## 第二步：在Docker中运行mysql容器

#### 启动mysql

1. 启动Docker

   ```
    sudo systemctl start docker
   ```

2. 拉取镜像

   ```
   docker pull mysql
   ```

3. 运行容器

   ```
   docker run --name mysql01 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
   ```

4. 查看容器启动情况

   ```
   docker ps
   ```

*这个时候用navicat连接mysql，报错错误码为2003，多半是因为没有启动mysql，或者需要设置云服务器开放3306端口*

#### 容器内登录mysql

1. 进入mysql

   docker exec -it 容器ID mysql -p

2. 输入mysql密码（123456）

## 第三步：客户端登录mysql

打开navicat，输入主机（云服务器IP）、端口（默认3306），用户名（root）连接。如果遇到1251错误，需要修改MySQL用户登录的加密规则：

```
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY MySQL密码;
ALTER USER 'root'@'%' IDENTIFIED BY MySQL密码 PASSWORD EXPIRE NEVER;
FLUSH PRIVILEGES;
```

`ALTER USER 'root'@'%'`@后面是指定开放的IP，%就是所有IP均可访问

navicat中新建数据库和表，在服务器中可以看到同步更新：

1. 查看所有数据库

   ```
   show databases;
   ```

2. 查看表

   ```
   use <新建数据库>;
   show tables;
   select * from <新建表>;
   ```

   

