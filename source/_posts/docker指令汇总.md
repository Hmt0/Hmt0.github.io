---
title: docker指令汇总
date: 2021-12-05 00:35:46
tags: docker
---

构建镜像

DockerFile

```
# syntax=docker/dockerfile:1
FROM node:12-alpine
RUN apk add --no-cache python3 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

构建镜像

```
docker build -t getting-started .
```

启动容器

```
 docker run -dp 3000:3000 getting-started
```

更新应用程序

```
 docker build -t getting-started .
```

删除旧的容器

获取ID

```
docker ps
```

停止旧容器

```
 Swap out <the-container-id> with the ID from docker ps
 $ docker stop <the-container-id>
```

移除旧容器

```
 docker rm <the-container-id>
```

合并

```
docker rm -f <container-id>
```



应用程序共享

登录

`docker login -u YOUR-USER-NAME`.

重命名镜像

```
 docker tag getting-started YOUR-USER-NAME/getting-started
```

推送

```
docker push YOUR-USER-NAME/getting-started
```



新建文件

```
docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
```

dockers cli查看文件内容

```
cat /data.txt
```

使用dockers exec命令行查看文件内容：

```
 docker exec <container-id> cat /data.txt
```

数据持久化

创建卷

```
 docker volume create todo-db
```

创建应用程序容器，指定卷挂载：

```
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

查看数据存储信息：

```
docker volume inspect todo-db
```



使用绑定安装

```
docker run -dp 3000:3000 `
     -w /app -v "$(pwd):/app" `
     node:12-alpine `
     sh -c "yarn install && yarn run dev"
```

查看日志

```
docker logs -f 8f660e5ddb02
```

part 7混合容器应用程序

创建网络应用

```
docker network create todo-app
```

创建mySQL容器并将它连接到网络

```
docker run -d `
     --network todo-app --network-alias mysql `
     -v todo-mysql-data:/var/lib/mysql `
     -e MYSQL_ROOT_PASSWORD=secret `
     -e MYSQL_DATABASE=todos `
     mysql:5.7
```

连接数据库

```
docker exec -it <mysql-container-id> mysql -u root -p
```

输入密码后，显示数据库：

```
 SHOW DATABASES;
```

连接mySQL

使用nicklaka/netshoot镜像创建一个新的容器：

```
docker run -it --network todo-app nicolaka/netshoot
```

查找mySQL主机的IP地址

```
dig mysql
```

**用MySQL运行应用程序**

修改数据库权限

```
 mysql> ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'secret';
 mysql> flush privileges;
```

指定环境变量，并将容器连接到应用程序网络。

```
docker run -dp 3000:3000 `
   -w /app -v "$(pwd):/app" `
   --network todo-app `
   -e MYSQL_HOST=mysql `
   -e MYSQL_USER=root `
   -e MYSQL_PASSWORD=secret `
   -e MYSQL_DB=todos `
   node:12-alpine `
   sh -c "yarn install && yarn run dev"
```

查看日志

```
docker logs <container-id>
```

连接mysql数据库
