# ikubernetes
my k8s notes

> 这是我学习kubernetes的一点笔记与心得

## 目录简介
```
ikubernetes/
├── README.md
├── sys-kubernetes    #搭建环境用到的一些资源文件
│   ├── canal         #用于为k8s集群提供网络策略
│   ├── dashboard     #可视化界面
│   ├── flannel       #提供pod网络
│   └── ingress-nginx #提供七层的负载均衡
└── user-kubernetes   #自己写的一些资源清单
    ├── base          #基础资源pod,无状态应用控制器deployment、replicaset、daemonset、cronjob等等
    ├── configmap     #configmap资源，主要用于为pod提供配置
    ├── ingress-nginx #七层负载
    ├── networkpolicy #网络策略
    ├── RBAC          #访问控制
    ├── scheduler     #资源调度
    ├── service       #service资源
    ├── statefulset   #有状态应用控制器
    └── volume        #存储
```
## kubernetes集群搭建
```
使用工具:kubeadm
系统:CentOS 7 * 3
    192.168.179.50  k8s-master
    192.168.179.51  k8s-node1
    192.168.179.52  k8s-node2
容器引擎:Docker 18.09
集群版本:kubernetes 1.13.3
```
> 注:master的CPU至少2核,实验前请关闭防火墙和selinux
### Master
```
1.安装docker
~]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -P /etc/yum.repos.d/  #可选择对应版本rpm包直接安装
#选择docker版本:
~]# yum list docker-ce --showduplicates
~]# yum install -y --setopt=obsoletes=0 docker-ce-18.06.1.ce-3.el7

2.集群安装

~]# cat /etc/yum.repos.d/kubernetes.repo 
	[kubernetes]
	name=kubernetes repo
	baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
	gpgcheck=1
	gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
		   https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
	enabled=1

~]# yum install kubelet kubeadm kubectl -y
~]# systemctl start docker  #不能先启动kubelet，否则会报错
~]# systemctl enable kubelet docker

禁用swap分区，Kubernetes 1.8开始要求必须禁用Swap，如果不关闭，默认配置下kubelet将无法启动。


```
