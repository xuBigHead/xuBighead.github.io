# 目录

[TOC]

# 前言

## 概述

Docker是一个可以快速的创建和模拟各类环境的一个工具。Docker能够自动执行重复性任务，例如搭建和配置开发环境。

- **不受应用、语言或技术栈限制**

构建、测试、调试和部署以任何编程语言编写的 Linux 和 Windows Server 容器应用，无需担心任何不兼容或版本冲突。

- **绝佳的开发体验**

工作就绪时间缩短 65%：快速构建、测试和运行复杂的多容器应用，无需再浪费时间在服务器和开发人员机器上安装和维护软件。所有依赖资源都在容器中运行，消除“在我的机器上可正常工作”的问题。 

- **内置容器编排**

Docker 内置易于配置的 Swarm 集群功能。在使用最小设置的模拟生产环境中测试和调试应用。

- **快速扩展**

内置编排能够扩展到数千个节点和容器。Docker 容器能够在短短数秒之内启动和停止，便于扩展应用服务，以满足客户的高峰需求，并在峰值下降时缩减规模。

- **提高效率**

Docker 让客户轻松部署、识别和解决问题，降低总体 IT 运维成本。缩短部署更新的停机时间，或者迅速回滚，尽量减少中断运行情况。

- **轻松共享应用**

Docker 确保应用在任何环境中都能始终如一地工作。在 Docker 镜像中，整个技术栈和配置都是镜像的一部分，用户只需安装 Docker，无需配置主机环境。

  

## 应用场景

- **微服务**

微服务拆分后，一个项目可能部署包就成倍增加了，而且可能各微服务之间的技术栈是不同的，这时候docker就是最佳选择了。

- **持续集成和持续部署 (CI/CD)**

结合Jenkins,通过 Docker 加速应用管道自动化和应用部署，交付速度了有很大程度的提高。

- **IT** **基础设施优化**

Docker 和容器有助于优化 IT 基础设施的利用率和成本。优化不仅仅是指削减成本，还能确保在适当的时间有效地使用适当的资源。

- **容器化传统应用**

容器不仅能提高现有应用的安全性和可移植性，还能节约成本。

  

# 架构

![image-20210108141617479](https://raw.githubusercontent.com/xuBigHead/pic/master/img/20210108141623.png)



对于Docker而言，主要是使用了容器技术。有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置。系统因此而变得高效、轻量、自给自足，还能保证部署在任何环境中的软件都能始终如一地运行。

![image-20210108141659361](https://raw.githubusercontent.com/xuBigHead/pic/master/img/20210108141659.png)



# 基础概念

## 客户端和服务端

Docker是一个（C/S）架构的程序。Docker客户端只需向Docker服务器或者守护进程发出请求，服务器或者守护进程将完成所有的工作并返回结果。Docker守护进程有时也称为Docker引擎。



## 镜像（Images）

镜像就是程序运行的环境的只读版本。其包含了所有程序的依赖软件和配置。



## 容器（Container）

Docker 利用容器（Container）来运行应用。容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。   

**容器**是一个应用层抽象，用于将代码和依赖资源打包在一起，多个容器可以在同一台机器上运行，共享操作系统内核，但各自作为独立的进程在用户空间中运行。与虚拟机相比，容器占用的空间较小（容器镜像大小通常只有几十兆），瞬间就能完成启动。

**虚拟机**是一个物理硬件层抽象，用于将一台服务器变成多台服务器。管理程序允许多个VM在一台机器上运行。每个VM都包含一整台操作系统、一个或多个应用、必要的二进制文件和库资源，因此占用大量空间。而且VM启动也十分缓慢。



## 仓库（Repository）

仓库用来保存镜像，可以理解为代码控制中的代码仓库。



# 常用命令

## 镜像常用命令

### search搜索镜像

```shell
$ docker search [imageName]
```



### pull拉取镜像

```shell
$ docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```



### images查看镜像
```shell
$ docker images [imageName]
```



### tag镜像拷贝

```shell
$ docker tag [oldImageName] [newImageName]
```



### build构建镜像

build基于当前目录的Dockerfile创建一个新的镜像，同时命名镜像(`-t`，指tag)。
```shell
$ docker build -t first-image 
```



### rmi删除镜像

删除镜像，参数`-f`表示存在多个同镜像ID的镜像时，强制删除，否则会抛出异常 

```shell
$ docker rmi [-f] [imageName|imageId]
```



## 容器常用命令

### run运行容器

```shell
$ docker run [imageName]
```


- `-a`: stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
- `-d`: 后台运行容器，并返回容器ID；
- `-i`: 以交互模式运行容器，通常与 -t 同时使用；
- `-p`: 端口映射，格式为：主机(宿主)端口:容器端口
- `-t`: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
- `--name="nginx-lb"`: 为容器指定一个名称；
- `--dns 8.8.8.8`: 指定容器使用的DNS服务器，默认和宿主一致；
- `--dns-search example.com`: 指定容器DNS搜索域名，默认和宿主一致；
- `-h "mars"`: 指定容器的hostname；
- `-e username="ritchie"`: 设置环境变量；
- `--env-file=[]`: 从指定文件读入环境变量；
- `--cpuset="0-2" or --cpuset="0,1,2"`: 绑定容器到指定CPU运行；
- `-m` :设置容器使用内存最大值；
- `--net="bridge"`: 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
- `--link=[]`: 添加链接到另一个容器；
- `--expose=[]`: 开放一个端口或一组端口；



### ps容器列表

使用ps命令，可以列举出当前运行的容器，需要所有容器时，加入-a选项即可。

```shell
$ docker ps [-a]
```



### stop停止容器

```shell
$ docker stop [containerId]
```



### start启动已停止容器
```shell
$ docker start [containerId] 
```



### restart重启容器

```shell
$ docker restart [containerId] 
```

 

### kill强杀容器
```shell
$ docker kill [containerId]
```



### rm删除容器
```shell
$ docker rm [containerId]
```

 

### exec进入容器

在一些场景下，比如想查看redis的客户端redis-cli时，这个时候就需要进入容器了。进入容器有很多中，这里就exec进行讲解。

```shell
$ docker exec -it [containerId] [appName]
# 进入指定容器的Redis客户端
$ docker exec -it 3ba5b7475423 redis-cli
```
- `-d`：分离模式: 在后台运行 

- `-i`：即使没有附加也保持STDIN 打开

- `-t`：分配一个伪终端

  

### commit容器中创建镜像

在制作一些私有镜像时，常常是依赖于一个基础镜像后，然后进入容器中进行相关系统环境的配置，或者相应的优化。但若容器一删除，之前的修改都会没有了。故在这些场景下，可直接从修改后的容器中创建一个自己的私有镜像，这样里面的一些环境和相关优化项还是保留的。

```shell
$ docker commit [options] [containerId] name:tag
```
- `-a`：提交的镜像作者
- `-c`：使用Dockerfile指令来创建镜像
- `-m`：提交时的说明文字
- `-p`：在commit时，将容器暂停

  

## 其他常用命令

### logs查看日志
```shell
$ docker logs [OPTIONS] [containerId]
```

-  `-f` : 跟踪日志输出
-  `--since` :显示某个开始时间的所有日志
-  `-t` : 显示时间戳
-  `--tail` :仅列出最新N条容器日志

  

### cp宿主和容器之间相互拷贝文件
```shell
# docker cp 容器名：要拷贝的文件在容器里面的路径  要拷贝到宿主机的相应路径
$ docker cp 3ba5b7475423:/opt/a.json /opt
# docker cp 要拷贝的文件路径 容器名：要拷贝到容器里面对应的路径
$ docker cp /opt/a.json 3ba5b7475423:/opt
```

  

# Dockerfile

Dockerfile是一个文本文件，里面包含了若干条指令，每条指令描述了构建镜像的细节。简单来说，它就是由一系列指令和参数构成的脚本文件，从而构建出一个新的镜像文件。



## Dockerfile示例

下面是一个简单的Dockerfile文件，可以通过build命令将其构建成一个docker镜像。

```dockerfile
FROM nginx
LABEL author="作者：xmm"
LABEL version="版本：v0.1"
LABEL desc="说明：修改nginx首页提示"
RUN echo 'hello,oKong' > /usr/share/nginx/html/index.html
CMD ["nginx", "-g", "daemon off;"]
```



## Dockerfile语法

### FROM 

指定基础镜像，放在第一行，其格式如下

 ```dockerfile
FROM <image>    
FROM <image>:<tag>   
FROM <image>:<digest>  
 ```

若想构建一个最小的镜像，不想基于其他任何镜像时，可使用如下方式：

```dockerfile
FROM scratch  
```

### LABEL 

镜像元数据，可以设置镜像的任何元数据，格式如下：

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value>
# 或  
LABEL <key>=<value>     
LABEL <key>=<value>     
LABEL <key>=<value>      
```

利用docker inspect [IMAGE_NAME|IMAGE_ID]命令进行查看镜像元素据。

 ```sh
$ docker inspect [imageName|imageId]
 ```

### ENV 

设置环境变量，之后的命令都可以用此变量进行赋值，格式如下：
```dockerfile
ENV <key> <value>   
ENV <key>=<value>  
...     
```

### VOLUME

定义匿名卷，创建挂载点，即向基于所构建镜像创始的容器添加卷，一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统，并具有以下功能：

卷可以容器间共享和重用

容器并不一定要和其它容器共享卷

修改卷后会立即生效

对卷的修改不会对镜像产生影响

卷会一直存在，直到没有任何容器在使用它

VOLUME可以将源代码、数据或其它内容添加到镜像中，而又不并提交到镜像中，并使多个容器间共享这些内容。
```dockerfile
VOLUME ["/data"]   
```

### COPY 

复制文件，主要就是构建镜像时，进行拷贝文件到镜像的指定路径下，格式为：
```dockerfile
COPY <源路径>...  <目标路径>   
COPY ["<源路径1>",...  "<目标路径>"]
```

### ADD 

更高级的复制文件，ADD指令和COPY的格式和性质基本一致。但是在COPY基础上增加了一些功能。比如<源路径>可以是一个 URL，这种情况下，Docker引擎会试图去下载这个链接的文件放到<目标路径>去。
### EXPOSE 

为镜像设置监听端口，容器运行时会监听改端口。

```dockerfile
EXPOSE <port> [<port>/<protocol>...]  
# 为镜像设置监听端口，同时，也能指定协议名   
EXPOSE 80/udp  
```
### ARG

该命令用于设置构建参数，该参数在容器运行时是获取不到的，只有在构建时才能获取。这也是其和ENV的区别。

```dockerfile
ARG <name>[=<default  value>]    
```
### RUN 

在镜像的构建过程中执行特定的命令，并生成一个中间镜像。

```dockerfile
RUN <command>   
# 或者   
RUN ["executable", "param1", "param2"]  
```
第一种后边直接跟shell命令，在linux操作系统上默认 /bin/sh -c；在windows操作系统上默认 cmd /S /C。

第二种是类似于函数调用，可将executable理解成为可执行文件，后面就是两个参数。

 ```dockerfile
RUN /bin/bash  -c 'source $*HOME*/.bashrc;  echo $*HOME *  
RUN ["/bin/bash", "-c", "echo  hello"]  
 ```

多行命令不要写多个RUN，原因是Dockerfile中每一个指令都会建立一层.多少个RUN就构建了多少层镜像，会造成镜像的臃肿、多层，不仅仅增加了构件部署的时间，还容易出错。

RUN书写时的换行符是\。

### CMD

容器启动时要运行的命令，第一种和第二种都是可执行文件加上参数的形式，第三种是shell写法。

```dockerfile
CMD ["executable","param1","param2"]   
CMD ["param1","param2"]   CMD command param1 param2  
```
  这里边包括参数的一定要用双引号，就是双引号",不能是单引号。千万不能写成单引号。原因是参数传递后，docker解析的是一个JSON array。

 ```dockerfile
CMD [ "sh", "-c", "echo  $*HOME*" ]   
CMD [ "echo", "$*HOME*"  ]  
 ```

### ENTRYPOINT 

ENTRYPOINT 用于给容器配置一个可执行程序。也就是说，每次使用镜像创建容器时，通过 ENTRYPOINT 指定的程序都会被设置为默认程序。ENTRYPOINT 有以下两种形式：

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]   
ENTRYPOINT command param1 param2  
```
ENTRYPOINT 与 CMD 非常类似，不同的是通过docker run执行的命令不会覆盖 ENTRYPOINT，而docker run命令中指定的任何参数，都会被当做参数再次传递给ENTRYPOINT。Dockerfile 中只允许有一个 ENTRYPOINT 命令，多指定时会覆盖前面的设置，而只执行最后的ENTRYPOINT 指令。

docker run运行容器时指定的参数都会被传递给ENTRYPOINT，且会覆盖 CMD 命令指定的参数。如，执行docker run <image> -d时，-d 参数将被传递给入口点。也可以通过docker run --entrypoint重写 ENTRYPOINT 入口点。如：可以像下面这样指定一个容器执行程序：

 ```dockerfile
ENTRYPOINT ["/usr/bin/nginx"] 
 ```

### WORKDIR

用于在容器内设置一个工作目录，通过WORKDIR设置工作目录后，Dockerfile 中其后的命令RUN、CMD、ENTRYPOINT、ADD、COPY等命令都会在该目录下执行。

```dockerfile
WORKDIR /opt/docker/workdir 
```
### USER

  用于指定运行镜像所使用的用户，使用USER指定用户后，Dockerfile 中其后的命令RUN、CMD、ENTRYPOINT都将使用该用户。镜像构建完成后，通过docker run运行容器时，可以通过-u参数来覆盖所指定的用户。

```dockerfile
USER okong
```
# 安装和卸载

## 安装Docker

### Linux环境

**下载安装Docker**

```shell
# 查看Docker版本
$ yum list docker-ce --showduplicates | sort -r
# 下载最新版本Docker，设置-y后表示下载并安装。
$ sudo yum [-y] install docker-ce
# 下载安装指定版本的Docker，包名是截取Docker版本列表的第一列和第二列的部分值组合而成
$ sudo yum [-y] install docker-ce-18.03.1.ce
# 启动Docker
$ sudo systemctl start docker
# 验证Docker是否安装成功
$ sudo docker info
```



**添加Docker配置**

由于国内墙的问题，安装完成后还需要设置阿里云加速器，推荐直接使用阿里云的镜像。

阿里云控制台首页(产品与服务) => 容器镜像服务 => [镜像加速器](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

```shell
# 编写文件/etc/docker/daemon.json(如果不存在，手动创建daemon.json文件)，内容为
$ sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://镜像地址.mirror.aliyuncs.com"]
}
EOF
# 重新加载daemon.json文件并重启Docker
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```



**测试官方hello-world镜像**

```shell
# 运行docker镜像
$ sudo docker run hello-world
```



### 安装时可能发生的异常

#### No package docker-ce available

这个问题需要安装基础包和设置yum源(由于国内环境，因此直接使用阿里云镜像地址)。

```shell
# 安装基础包，yum-utils提供yum-config-manager功能，另外两个是devicemapper驱动依赖。
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
# 设置yum源
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```



#### repomd.xml : HTTPS Error 404 - Not Found

查看/etc/yum.repos.d目录下的docker-ce.repo文件是否包含`https://download-stage.docker.com`，若包含则替换成`http://mirrors.aliyun.com/docker-ce`;

或者该目录下包含`download.docker.com_linux_centos_.repo`(未替换阿里云镜像加了官网的源地址时出现)文件的，删除此文件。



## 卸载Docker

### Linux环境

```shell
# 卸载Docker
$ sudo yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine
```



