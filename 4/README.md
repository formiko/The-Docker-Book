# 第4章 使用Docker镜像和仓库
## 4.1 什么是Docker镜像
## 4.2 列出镜像
* 列出Docker镜像
    ```
    sudo docker images
    ```
* 拉取ubuntu镜像
    ```
    sudo docker pull ubuntu:12.04
    ```
* 注意，我们通过docker pull得到的Ubuntu的latest镜像和12.04镜像，我们虽然称其为Ubuntu系统，但实际上它并不是一个完整的操作系统。它只是一个裁剪版，只包含最低限度的支持系统运行的组件。
* Docker Hub中有两种类型的仓库：用户仓库（user repository）和顶层仓库（top-level repository）。用户仓库的命名由用户名和仓库名两部分组成，如jamtur01/puppet。
## 4.3 拉取镜像
* 用docker run命令从镜像启动一个容器时，如果该镜像不在本地，Docker会先从Docker Hub下载该镜像。如果没有指定具体的镜像标签，那么Docker会自动下载latest标签的镜像。
* 拉取fedora镜像
    ```
    sudo docker pull fedora:20
    ```
* 查看fedora镜像
    ```
    sudo docker images fedora
    ```
* 拉取带标签的fedora镜像
    ```
    sudo docker pull fedora:21
    ```
## 4.4 查找镜像
* 我们也可以通过docker search命令来查找所有Docker Hub上公共的可用镜像。
* 查找镜像
    ```
    sudo docker search puppet
    ```
* 拉取jamtur01/puppetmaster镜像
    ```
    sudo docker pull jamtur01/puppetmaster
    ```
  （个人困惑：我这无法拉取）
* 从Puppet master镜像创建一个容器
    ```
    sudo docker run -i -t jamtur01/puppetmaster /bin/bash
    ```
## 4.5 构建镜像
* 构建镜像有以下两种方法：
    * 使用docker commit命令。
    * 使用docker build命令和Dockerfile文件。
  并不推荐使用docker commit命令，而应该使用更灵活、更强大的Dockerfile来构建Docker镜像。
### 4.5.1 创建Docker Hub账号
```
sudo docker login
```
### 4.5.2 用Docker的commit命令创建镜像
* 创建一个要进行修改的定制容器
    ```
    sudo docker run -i -t ubuntu /bin/bash
    ```
* 安装Apache软件包
    ```
    apt-get -yqq update
    apt-get -y install apache2
    ```
* 提交定制容器
    ```
    sudo docker commit 4aab3ce3cb76 jamtur01/apache2
    ```
* 检查新创建的镜像
    ```
    sudo docker images jamtur01/apache2
    ```
* 提交另一个新的定制容器
    ```
    sudo docker commit -m"A new custom image" -a"James Turnbull" 4aab3ce3cb76 jamtur01/apache2:webserver
    ```
* 查看提交的镜像的详细信息
    ```
    sudo docker inspect jamtur01/apache2:webserver
    ```
* 从提交的镜像运行一个新容器
    ```
    sudo docker run -t -i jamtur01/apache2:webserver /bin/bash
    ```
### 4.5.3 用Dockerfile构建镜像
* 创建一个示例仓库
    ```
    mkdir staticc_web
    cd static_web
    touch Dockerfile
    ```
* 我们的第一个Dockerfile
    ```
    # Version: 0.0.1
    FROM ubuntu:14.04
    MAINTAINER James Turnbull "james@example.com"
    RUN apt-get update && apt-get install -y nginx
    RUN echo 'Hi, I am in your container' > /usr/share/nginx/html/index.html
    EXPOSE 80
    ```
* 默认情况下，RUN指令会在shell里使用命令包装器/bin/sh -c来执行。如果是在个不支持shell的平台上运行或者不希望在shell中运行（比如避免shell字符串篡改），也可以使用exec格式的RUN指令，如
    ```
    RUN [ "apt-get", " install", "-y", "nginx" ]
    ```
* EXPOSE指令告诉Docker该容器内的应用程序将会使用容器的指定端口。这并不意味着可以自动访问任意容器运行中服务的端口（这里是80）。出于安全的原因，Docker并不会自动打开该端口，而是需要用户在使用docker run运行容器时来指定需要打开哪些端口。
* 可以指定多个EXPOSE指令来向外部公开多个端口。
### 4.5.4 基于Dockerfile构建新镜像
* 运行Dockerfile
    ```
    cd static_web
    sudo docker build -t="jamtur01/static_web" .
    ```
* 在构建时为镜像设置标签
    ```
    sudo docker build -t="jamtur01/static_web:v1" .
    ```
  上面命令中最后的.告诉Docker到本地目录中去找Dockerfile文件。
* 从Git仓库构建Docker镜像
    ```
    sudo docker build -t="jamtur01/static_web:v1" git@github.com:jamtur01/docker-static_web
    ```
  这里的Docker假设在这个Git仓库的根目录下存在Dockerfile文件。
* 自Docker 1.5.0开始，也可以通过-f标志指定一个区别于标准Dockerfile的构建源的位置。例如，dockerbuild-t"jamtur01/static_- web" -f path/to/file，这个文件可以不必命名为Dockerfile，但是必须要位于构建上下文之中。

* 如果在构建上下文的根目录下存在以.dockerignore命名的文件的话，那么该文件内容会被按行进行分割，每一行都是一条文件过滤匹配模式。这非常像.gitignore文件，该文件用来设置哪些文件不会被当作构建上下文的一部分，因此可以防止它们被上传到Docker守护进程中去。该文件中模式的匹配规则采用了Go语言中的filepath。

### 4.5.5 指令失败时会怎样
### 4.5.6 Dockerfile和构建缓存
* 忽略Dockerfile的构建缓存
    ```
    sudo docker build --no-cache -t="jamtur01/static_web" .
    ```
### 4.5.7 基于构建缓存的Dockerfile模板
* 构建缓存带来的一个好处就是，我们可以实现简单的Dockerfile模板（比如在Dockerfile文件顶部增加包仓库或者更新包，从而尽可能确保缓存命中）。
* Ubuntu系统的Dockerfile模板
    ```
    FROM ubuntu:14.04
    MAINTAINER James Turnbull "james@example.com"
    ENV REFRESHED_AT 2014-07-01
    RUN apt-get -qq update
    ```
