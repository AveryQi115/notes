# Docker简介

- 沙盒化（容器）的环境，隔离的文件环境
- 需要已经存在的linux内核
- 支持响应式部署
- docker daemon：运行在后台，负责部署，运行conatiners的程序，接受docker clients的通讯
- docker client：用来让用户和docker daemon交互的命令行界面
- docker hub：类似github，用来下载分享镜像

# 基本使用示例

- `docker pull <\imageName>`
- `docker images`: 展示所有images状态
- `docker run <\imageName> <\command>`：根据指定镜像运行一个容器，并在该容器里运行此条命令然后退出该容器
- `docker ps`:展示所有正在运行的container以及相关信息
- `docker ps -a`:展示所有运行过的container，当前没有运行的container会显示status Exited
- `docker run -it <\imageName>`:根据指定镜像运行一个容器，并打开该容器的一个交互式终端，exit退出
- `docker run -it <\imageName>`每次创建的是一个新的容器，在之前的容器中做的操作效果不会出现在新容器中（例如：之前创建的文件夹等）
- `docker rm <\containerID>` 删除对应的container，**注意**：即使container已经退出了，还是会占用磁盘空间，需要及时清理
- `docker rm $(docker ps -a -q -f status=exited)` 删除所有已经退出的container，-q 只返回containerID，-f 根据后面的等式过滤
- `docker container prune`和上面的命令一样的效果
- `docker run --rm <\imageName>` 退出后自动删除容器
- `docker stop <\containerID>` 停止或退出一个运行中的容器

# 网站部署相关操作

- 普通的docker run并没有暴露出可以与container交互的接口
- `docker run -d -P --name static-site prakhar1989/static-site`
  - -d：detach mode，分离终端和container的运行状态，终端关闭了，container仍然在后台运行
  - -P：暴露所有需要的接口
  - --name：给当前运行的container取个名字，之后可以通过名字代替containerID
  - 末尾： image

- `docker port <\container>` 查看暴露出的端口
- `docker run -p 8888:80 prakhar1989/static-site` -p自定义端口配置，其中8888为本机开放用来连接container的端口，80为原images设置端口

# 镜像

- Images就像是git repos，有不同的版本，可以做多次commit来修改，没有指定版本的时候images默认tag为latest
- **Base Image**是没有任何父镜像的镜像，一般来说是操作系统级别的镜像文件
- **Child Image**是建立在父镜像上的文件，在其基础上添加了多余的功能
- **Official Image**是由Docker官方人员发布的镜像文件
- **User Image**是由用户分享的镜像文件，一般名称格式为<\UserName>/<\ImageName>，所有的User Image都是base在base image上

## 创建自己的镜像文件

- **Dockerfile**是一个文本文件，其中包含了创建镜像文件时docker client会调用的所有指令
- 在工作文件夹（包含了程序文件的文件夹）下创建一个Dockerfile，dockerfile包含以下内容：
  - FROM <\baseImageName>
  - WORKDIR <\workDirName>
  - COPY src dst
  - RUN command to install dependencies
  - EXPOSE <\port>
  - CMD["arg1","arg2"] // 用于告诉container当它启动的时候哪一个指令会被运行
  - 示例：<img src="/Users/a77/hw/毕业实训/img/command.jpg" style="zoom:50%;" />
- `docker build -t <userName>/<imageName>:<tagName> <dir>`
  - -t ：填写image名称，标签
  - dir：包含当前Dockerfile的文件夹
- `docker push <userName>/<imageName>`将自己的镜像文件push到docker hub上（默认public），这样其他的用户可以通过pull 拉取

# 多容器环境

- 在现有的微服务环境下，将服务及相关依赖与其他的服务彼此分离是比较友好的，即一个容器承载一个服务
- `docker container logs </containerIDs>`

# Docker Network

- 当docker安装好时，它自动创建了三个网络

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
c2c695315b3a        bridge              bridge              local
a875bec5d6fd        host                host                local
ead0e804a67b        none                null                local
```

- container运行起来的时候，会跑在bridge网络里

```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "c2c695315b3aaf8fc30530bb3c6b8f6692cedd5cc7579663f0550dfdd21c9a26",
        "Created": "2018-07-28T20:32:39.405687265Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "277451c15ec183dd939e80298ea4bcf55050328a39b04124b387d668e3ed3943": {
                "Name": "es",
                "EndpointID": "5c417a2fc6b13d8ec97b76bbd54aaf3ee2d48f328c3f7279ee335174fbb4d6bb",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

- Containers属性可以看到运行的容器和它的相关属性，诸如ip，名称等等
- 多容器环境下需要查看具体ip来access相应服务
- 上述默认配置有两个问题：
  - 如果ip变动，怎么能让另一个container知道变动后的container ip
  - 所有container都运行在bridge网络中是一个不太安全的部署，怎么分离网络
- `docker network create <networkName> `会自动创建一个相应名称的bridge network，docker保证了同一bridge network中的container可以互相通讯，不同bridge network的container彼此不能通讯
  - ps：更多类型的network文档：https://docs.docker.com/network/

- 接下来使用`--net`参数使container在指定network中launch `docker run -d --name es --net foodtrucks-net -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.2`
- 当多个container都运行在用户定义的bridge network中时，container之间可以直接使用container Name来找到对方

# 扩展

- docker还有一些帮助部署的工具或第三方软件：docker machine，docker compose，docker swam，k8s（其中，docker machine和docker compose都比较deprecated）
- k8s学习