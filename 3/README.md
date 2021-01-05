# 第3章 Docker入门
## 3.1 确保Docker已经就绪
```
sudo docker info
```
## 3.2 运行我们的第一个容器
```
sudo docker run -i -t ubuntu /bin/bash
```
-i 标志保证容器中STDIN是开启的，尽管我们并没有附着到容器中。  
-t 告诉Docker为要创建的容器分配一个伪tty终端。
## 3.3 使用第一个容器
* 检查容器的主机名
    ```
    hostname
    ```
* 检查容器的/etc/hosts文件
    ```
    cat /etc/hosts
    ```
  Dokcer已在hosts文件中为该容器的IP地址添加了一条主机配置项。
* 检查容器的接口
    ```
    ip a
    ```
  可以看到，这里有lo的环回接口，还有IP为...的标准eth0网络接口，和普通宿主机是完全一样的。
* 检查容器的进程
    ```
    ps -aux
    ```
* 在第一个容器中安装软件包
    ```
    apt-get update && apt-get install vim
    ```
* 查看当前系统中容器的列表
    ```
    docker ps -a
    ```
  默认情况下，当执行`docker ps`命令时，只能看到正在运行的容器。
* 列出最后一个运行的容器，无论其正在运行还是已经停止
    ```
    docker ps -l
    ```
* 通过--format标志，进一步控制显示哪些信息，以及如何显示这些信息
  [参见此处使用format的方法](https://docs.docker.com/engine/reference/commandline/ps/)
  如
    ```
    docker ps --format "{{.ID}}: {{.Command}}"
    ```
  或
    ```
    docker ps --format "table {{.ID}}\t{{.Labels}}"
    ```
* 有3种方式可以唯一指代容器：短UUID、长UUID或名称
## 3.4 容器命名
* 给容器命名,--name选项
    ```
    sudo docker run --name bob_container -i -t ubuntu /bin/bash
    ```
  合法的容器名[a-zA-Z0-9_.-]
## 3.5 重新启动已经停止的容器
* 启动已经停止运行的容器
    ```
    sudo docker start bob_the_container
    ```
* 通过ID启动已经停止运行的容器
    ```
    sudo docker start aa3f365f0f4e
    ```
## 3.6 附着到容器上
* 附着到正在运行的容器
    ```
    sudo docker attach bob_the_container
    ```
* 通过ID附着到正在运行的容器
    ```
    sudo docker attach aa3f365f0f4e
    ```
## 3.7 创建守护式容器
```
sudo docker run --name daemon_dave -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```
## 3.8 容器内部都在干些什么
* 获取守护式容器的日志
    ```
    sudo docker logs daemon_dave
    ```
* 跟踪守护式容器的日志
    ```
    sudo docker logs -f daemon_dave
    ```
* 获取日志的最后10行内容
    ```
    sudo docker logs --tail 10 daemon_dave
    ```
* 跟踪某个容器的最新日志而不必读取整个日志文件
    ```
    sudo docker logs --tail 0 -f daemon_dave
    ```
* -t时间戳，跟踪守护式容器的最新日志
    ```
    sudo docker logs -ft daemon_dave
    ```
## 3.9 Docker日志驱动
```
sudo docker run --log-driver="syslog" --name daemon_dwayne -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```
## 3.10 查看容器内的进程
```
sudo docker top daemon_dave
```
## 3.11 Docker统计信息
```
sudo docker stats daemon_dave daemon_kate daemon_clare daemon_sarah
```
## 3.12 在容器内部运行进程
* 在容器中运行后台任务
    ```
    sudo docker exec -d daemon_dave touch /etc/new_config_file
    ```
  从Docker 1.7 开始，可以对docker exec启动的进程使用-u标识为新启动的进程指定一个用户属主。
* 在容器内运行交互命令
    ```
    sudo docker exec -t -i daemon_dave /bin/bash
    ```
## 3.13 停止守护式容器
* 停止正在运行的Docker容器
    ```
    sudo docker stop daemon_dave
    ```
* 通过容器ID停止正在运行的容器
    ```
    sudo docker stop c2c4e57c12c4
    ```
## 3.14 自动重启容器
* 自动重启容器
    ```
    sudo docker run --restart=always --name daemon_dave -d ubuntu /bin/sh -c "while true; do ehco hello world; sleep 1; done"
    ```
* 为on-failure指定count参数
    ```
    --reastart=on-failure:5
    ```
## 3.15 深入容器
* 查看容器
    ```
    sudo docker inspect daemon_dave
    ```
* 有选择地获取容器信息
    ```
    sudo docker inspect --format='{{ .State.Running}}' daemon_dave
    ```
* 查看容器的IP地址
    ```
    sudo docker inspect --format '{{ .NetwordSettings.IPAddress}}' daemon_dave
    ```
* 查看多个容器
    ```
    sudo docker inspect --format '{{.Name}} {{.State.Running}}' daemon_dave bob_the_container
    ```
* 除了查看容器，还可以通过浏览/var/lib/docker目录来深入了解Docker的工作原理。该目录存放着Docker镜像、容器以及容器的配置。所有的容器都保存在/var/lib/docker/containers目录下
## 3.16 删除容器
* 删除容器 
    ```
    sudo docker rm 80430f8d0921
    ```
  从Docker1.6.2开始，可以通过给docker rm命令传递-f标志来删除运行中的Docker容器。这之前的版本必须先使用docker stop或docker kill命令停止容器，才能将其删除。
* 删除所有容器
    ```
    sudo docker rm `sudo docker ps -a -q`
    ```
  -q表示只需要返回容器的ID而不会返回容器的其他信息
