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
  默认情况下，当执行```docker ps```命令时，只能看到正在运行的容器