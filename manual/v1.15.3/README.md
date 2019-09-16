# 一、Kubernetes 1.15 二进制集群安装

本系列文档将介绍如何使用二进制部署Kubernetes v1.15.3 集群的所有部署，而不是使用自动化部署(kubeadm)集群。在部署过程中，将详细列出各个组件启动参数，以及相关配置说明。在学习完本文档后，将理解k8s各个组件的交互原理，并且可以快速解决实际问题。

## 1.1、组件版本

```
Kubernetes 1.15.3
Docker 18.09 (docker使用官方的脚本安装，后期可能升级为新的版本，但是不影响)
Etcd 3.3.13
Flanneld 0.11.0
```

## 1.2、组件说明

kube-apiserver

```
使用节点本地Nginx 4层透明代理实现高可用 (也可以使用haproxy，只是起到代理apiserver的作用)
关闭非安全端口8080和匿名访问
使用安全端口6443接受https请求
严格的认知和授权策略 (x509、token、rbac)
开启bootstrap token认证，支持kubelet TLS bootstrapping；
使用https访问kubelet、etcd
```

kube-controller-manager
```
3节点高可用 (在k8s中，有些组件需要选举，所以使用奇数为集群高可用方案)
关闭非安全端口，使用10252接受https请求
使用kubeconfig访问apiserver的安全扣
使用approve kubelet证书签名请求(CSR)，证书过期后自动轮转
各controller使用自己的ServiceAccount访问apiserver
```
kube-scheduler
```
3节点高可用；
使用kubeconfig访问apiserver安全端口
```
kubelet
```
使用kubeadm动态创建bootstrap token
使用TLS bootstrap机制自动生成client和server证书，过期后自动轮转
在kubeletConfiguration类型的JSON文件配置主要参数
关闭只读端口，在安全端口10250接受https请求，对请求进行认真和授权，拒绝匿名访问和非授权访问
使用kubeconfig访问apiserver的安全端口
```
kube-proxy
```
使用kubeconfig访问apiserver的安全端口
在KubeProxyConfiguration类型JSON文件配置为主要参数
使用ipvs代理模式
```
集群插件
```
DNS 使用功能、性能更好的coredns
网络 使用Flanneld 作为集群网络插件
```

# 二、初始化环境

## 1.1、集群机器
```
#master节点
192.168.0.50 k8s-01
192.168.0.51 k8s-02
192.168.0.52 k8s-03

#node节点
192.168.0.53 k8s-04     #node节点只运行node，但是设置证书的时候要添加这个ip
```
本文档的所有etcd集群、master集群、worker节点均使用以上三台机器，并且初始化步骤需要在所有机器上执行命令。如果没有特殊命令，所有操作均在192.168.0.50上进行操作

node节点后面会有操作，但是在初始化这步，是所有集群机器。包括node节点，我上面没有列出node节点

## 1.2、修改主机名

所有机器设置永久主机名

```
hostnamectl set-hostname abcdocker-k8s01  #所有机器按照要求修改
bash        #刷新主机名
```
接下来我们需要在所有机器上添加hosts解析
```
cat >> /etc/hosts <<EOF
192.168.0.50  k8s-01
192.168.0.51  k8s-02
192.168.0.52  k8s-03
192.168.0.53  k8s-04
EOF
```




参考资料

https://i4t.com/4253.html   Kubernetes 1.14 二进制集群安装

https://github.com/kubernetes/kubernetes/releases/tag/v1.15.3   下载链接