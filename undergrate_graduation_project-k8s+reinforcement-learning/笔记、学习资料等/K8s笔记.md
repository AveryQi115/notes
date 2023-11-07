# K8s tutorial(deprecated)

## Cluster

- cluster由一群nodes组成，每一个node是一个worker machine
- 一个cluster中至少还有一个worker node，一个worker node上面部署了pods

- 基本结构：<img src="/Users/a77/Library/Application Support/typora-user-images/image-20211018173705795.png" alt="image-20211018173705795" style="zoom:33%;" />

- Control Plane：负责application的调度；scale up/down；维持状态；滚动更新
- Node：VM/物理机 
  - 每一个Node拥有一个Kubelet，负责和Control Plane通讯
  - 每一个Node需要有容器化的工具，eg:Docker
  - 一个cluster至少三个Node

## Minikube

- 轻量级k8s
- 在本地创建一个虚拟机用于部署一个只含1个Node的cluster

## 启动和查看基本信息

- Minikube版本：`minikube version`
- 启动集群：`minikube start`
- 查看server和client相关信息：`kubectl version`
- 查看cluster信息（包括control plane网址和DNS域名）：`kubectl cluster-info`
- 查看所有node信息和status：`kubectl get nodes`

## 部署App

<img src="/Users/a77/Library/Application Support/typora-user-images/image-20211018180347112.png" alt="image-20211018180347112" style="zoom:33%;" />

- Deployment: 部署在control plane上
- Pod：一个或多个容器，为了方便注册和网络通讯被绑在一起；Deployment负责监控Pod，如果Pod停止，则重启Pod中Container
- Dashboard: 一般来说Cluster的状态只能在其内部的网络获取，创建dashboard允许外界查看状态
- Pod只能在cluster内部通过IP地址接触，如果外界需要access Pod对应的容器，需要暴露出来作为service

- 创建一个deployment用来监控Pods，Pod中的容器基于image参数镜像 `kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4`
- 查看deployment: `kubectl get deployments`
- 查看Pod:`kubectl get pods`
- 查看events:`kubectl get events`
- 查看config:`kubectl config view`
- 暴露Pod作为服务:`kubectl expose deployment hello-node --type=LoadBalancer --port=8080`

