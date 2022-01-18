---
title: docker指令汇总
date: 2021-12-05 00:35:46
tags: docker
---

## Part1：定位和安装

#### 运行教程容器

```
 docker run -d -p 80:80 docker/getting-started
```



## Part2：简单的应用

#### 构建应用容器的镜像

1. 创建DockerFile文件

   ```
   # syntax=docker/dockerfile:1
   
   FROM node:12-alpine
   RUN apk add --no-cache python3 g++ make
   WORKDIR /app
   COPY . .
   RUN yarn install --production
   CMD ["node", "src/index.js"]
   ```

2. 构建容器镜像

   ```
   docker build -t getting-started .
   ```

   

#### 启动应用容器

1. 启动容器

   ```
   docker run -dp 3000:3000 getting-started
   ```

2. 在[http://localhost:3000](http://localhost:3000/)中查看应用



## Part3：更新应用程序

#### 更新源码

1. 修改`src/static/js/app.js`

2. 构建新的镜像

   ```
    docker build -t getting-started .
   ```

   

#### 删除旧的容器

1. 获取容器ID

   ```
   docker ps
   ```

2. 停止旧容器

   ```
    # Swap out <the-container-id> with the ID from docker ps
    $ docker stop <the-container-id>
   ```

3. 移除旧容器

   ```
    docker rm <the-container-id>
   ```

*合并停止和移除指令*

```
docker rm -f <container-id>
```



### 启动新的应用容器

1. 启动新容器

   ```
   docker run -dp 3000:3000 getting-started
   ```

2. 刷新浏览器

## Part4：应用程序共享

#### 创建仓库

1. 登录[Docker Hub](https://hub.docker.com/)
1. 点击Create Respository按钮
1. 命名，确保可见性为Public
1. 点击Create按钮

#### 推送镜像

1. 登录Docker Hub

   ```
   docker login -u YOUR-USER-NAME
   ```

2. 重命名镜像

   ```
    docker tag getting-started YOUR-USER-NAME/getting-started
   ```

3. 推送镜像

   ```
   docker push YOUR-USER-NAME/getting-started
   ```

#### 在一个新实例上运行镜像

1. 打开[Play with Docker](https://labs.play-with-docker.com/)

2. 点击登录，选择docker

3. 连接至Docker Hub账号

4. 点击ADD NEW INSTANCE

5. 启动刚刚推送的应用

   ```
   docker run -dp 3000:3000 YOUR-USER-NAME/getting-started
   ```

6. 查看3000端口

## Part5：数据持久化

#### 新建两个容器

1. 新建一个`ubuntu`容器，同时创建内容为一个随机数字的`/data.txt`文件

   ```
   docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
   ```

2. 查看文件内容

   *ubuntu容器shell终端命令：*

   ```
   cat /data.txt
   ```

   *使用dockers exec命令行：*

    ```
    docker exec <container-id> cat /data.txt
    ```

3. 启动相同镜像的另一个ubuntu容器，并查看目录

   ```
    docker run -it ubuntu ls /
   ```

#### 持久化todo数据

1. 创建卷

   ```
    docker volume create todo-db
   ```

2. 创建应用程序容器，指定卷挂载：

   ```
   docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
   ```

3. 在浏览器中查看todo数据，删除容器后重新创建，可以看到todo数据仍然在列表中

#### 查看数据存储信息：

```
docker volume inspect todo-db
```



## Part6：使用绑定挂载

#### 启动一个开发模式容器

1. 在PowerShell下输入命令：

   ```
   docker run -dp 3000:3000 `
        -w /app -v "$(pwd):/app" `
        node:12-alpine `
        sh -c "yarn install && yarn run dev"
   ```

2. 查看日志

   ```
   docker logs -f 8f660e5ddb02
   ```

3. 修改`src/static/js/app.js`中的代码，刷新浏览器，可以看到更新

## part7：混合容器应用程序

#### 启动MySQL

1. 创建网络

   ```
   docker network create todo-app
   ```

2. 创建mySQL容器并将它连接到网络，并定义环境变量

   ```
   docker run -d `
        --network todo-app --network-alias mysql `
        -v todo-mysql-data:/var/lib/mysql `
        -e MYSQL_ROOT_PASSWORD=secret `
        -e MYSQL_DATABASE=todos `
        mysql:5.7
   ```

3. 连接数据库

   ```
   docker exec -it <mysql-container-id> mysql -u root -p
   ```

4. 输入密码后，显示数据库：

   ```
    SHOW DATABASES;
   ```

#### 连接mySQL

1. 使用nicklaka/netshoot镜像创建一个新的容器：

   ```
   docker run -it --network todo-app nicolaka/netshoot
   ```

2. 查找mySQL主机的IP地址

   ```
   dig mysql
   ```

**用MySQL运行应用程序**

1. 修改数据库权限

   ```
    mysql> ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'secret';
    mysql> flush privileges;
   ```

2. 指定环境变量，并将容器连接到应用程序网络。

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

3. 查看日志

   ```
   docker logs <container-id>
   ```

4. 在浏览器中打开应用程序并向todo列表中添加内容

5. 连接mysql数据库并验证内容已经被写入数据库

   ```
   docker exec -it <mysql-container-id> mysql -p todos
   ```

	在mysql sell中运行

    ```
    select * from todo_items;
    ```

## Part8：使用Docker Compose

#### 安装Docker Compose

查看版本

```
 docker-compose version
```

#### 创建Compose文件

1. 在app项目的根目录下，创建`docker-compose.yml`文件
2. 定义模式版本
3. 定义希望作为应用程序的一部分运行的服务(或容器)列表

```
 version: "3.7"

 services:
```

#### 定义应用服务

```
PS> docker run -dp 3000:3000 `
  -w /app -v "$(pwd):/app" `
  --network todo-app `
  -e MYSQL_HOST=mysql `
  -e MYSQL_USER=root `
  -e MYSQL_PASSWORD=secret `
  -e MYSQL_DB=todos `
  node:12-alpine `
  sh -c "yarn install && yarn run dev"
```

1. 定义容器的服务条目和镜像

2. 定义指令

3. 定义服务端口

4. 定义工作目录和卷

5. 迁移环境变量

   ```
    version: "3.7"
   
    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
        ports:
          - 3000:3000
        working_dir: /app
        volumes:
          - ./:/app
        environment:
          MYSQL_HOST: mysql
          MYSQL_USER: root
          MYSQL_PASSWORD: secret
          MYSQL_DB: todos
   ```

#### 定义MySQL服务

```
PS> docker run -d `
  --network todo-app --network-alias mysql `
  -v todo-mysql-data:/var/lib/mysql `
  -e MYSQL_ROOT_PASSWORD=secret `
  -e MYSQL_DATABASE=todos `
  mysql:5.7
```

1. 定义新服务并命名为`mysql`
2. 定义卷映射
3. 指定环境变量

```
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

#### 运行应用程序堆栈

1. 启动应用程序堆栈

   ```
   docker-compose up -d
   ```

2. 查看日志

   ```
   docker-compose logs -f
   ```

## Part9：镜像构建的最佳实践

#### 安全性扫描

```
docker scan getting-started
```

#### 镜像层

```
docker image history getting-started
```

展开所有内容：

```
docker image history --no-trunc getting-started
```

#### 层缓存

1. 更新Dcokerfile

   ```
   # syntax=docker/dockerfile:1
    FROM node:12-alpine
    WORKDIR /app
    COPY package.json yarn.lock ./
    RUN yarn install --production
    COPY . .
    CMD ["node", "src/index.js"]
   ```

2. 创建`dockerignore`

   ```
    node_modules
   ```

3. 创建一个新镜像

   ```
    docker build -t getting-started .
   ```

4. 修改`src/static.index.html`

5. 重新构建镜像

   ```
   docker build -t getting-started .
   ```

   这一次构建使用了缓存

#### 多级构建

React的例子：

```
# syntax=docker/dockerfile:1
FROM node:12 AS build
WORKDIR /app
COPY package* yarn.lock ./
RUN yarn install
COPY public ./public
COPY src ./src
RUN yarn run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```

