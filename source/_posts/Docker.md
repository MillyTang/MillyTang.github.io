---
title: Docker
date: 2020-11-07 16:10:21
tags: docker, shell, tools
---

## Set up your Docker environment

* 准备工作： 下载
* Dockerfile: 项目中根目录下增加一个Dockerfile配置文件，以前端项目为例
* docker 几个概念和常用命令:
  * 概念：
    * `container`: 容器，一个被隔离的且有自己的文件系统，网络的正常的操作系统进程，进程树独立于主机
    * `image`: 镜像，一个配置文件用来生成容器的
  * 常用命令：
    1. `docker run your-Docker-image`,
    2. `docker ps --all`
    3. Build and test: `docker build --tag tagName:Vsesion .`
    4. Run your image as a container: `docker run --publish 8000:8080 --detach --name container-alias-name tagName:Vsesion`
    5. Delete container: `docker rm --force container-alias-name`

```Dockerfile
# FORM: 从 node:12.18.1 这个线上的image继承
FROM node:12.18.1
# ENV 执行环境是生产环境还是开发环境
ENV NODE_ENV=production
# WORKDIR: 工作目录/app
WORKDIR /app
COPY ["package.json", "package-lock.json*", "./"]
RUN npm install --production
COPY . .
CMD ["node", "server.js"]
```

## Build and run your image: In projects

```bash
```
