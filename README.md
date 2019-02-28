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
镜像加速:由于国内拉取dockerhub上的镜像速度较慢或会拉取失败,使用国内镜像加速访问
~]# cat /etc/docker/daemon.json 
{
	"registry-mirrors":["https://registry.docker-cn.com"]
}

2.集群安装
2.1 基础准备
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

2.2 禁用swap分区
Kubernetes 1.8开始要求必须禁用Swap，如果不关闭，默认配置下kubelet将无法启动。
法一:
~]# vim /etc/fstab
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
~]# swapoff -a
修改内核参数：
~]# cat /proc/sys/vm/swappiness
~]# vim /etc/sysctl.d/k8s.conf
vm.swappiness=0	  #swappiness=0的时候表示最大限度使用物理内存，然后才是 swap空间，swappiness＝100的时候表示积极的使用swap分区，并且把内存上的数据及时的搬运到swap空间里面。
~]# sysctl -p /etc/sysctl.d/k8s.conf

法二:允许跳过此错误
~]# cat /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--fail-swap-on=false"

2.3 集群初始化
~]# kubeadm init --kubernetes-version=v1.13.1	\     #正在使用的k8s程序组件的版本号，需要与kubelet的版本号相同。 
		--pod-network-cidr=10.244.0.0/16 \    #pod的网络地址，使用flannel插件时，默认是10.244.0.0/16
		--service-cidr=10.96.0.0/12 \	      #service的网络地址，默认地址为10.96.0.0/12
		--image-repository registry.aliyuncs.com/google_containers \	#默认仓库的gcr.io，国内无法访问，显示定义仓库地址
		#--ignore-preflight-errors=Swap	\     #忽略 swap分区未关闭的错误
		#apiserver-advertise-address=192.168.179.50	#apiserver通告给其他组件的IP地址，一般为该节点的地址，0.0.0.0表示节点上所有的可用地址
		...
		[addons] Applied essential addon: CoreDNS
		[addons] Applied essential addon: kube-proxy
		
		Your Kubernetes master has initialized successfully!
		
		To start using your cluster, you need to run the following as a regular user:
		
		  mkdir -p $HOME/.kube
		  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		  sudo chown $(id -u):$(id -g) $HOME/.kube/config
		
		...
		
		kubeadm join 192.168.179.50:6443 --token z95afq.1lolsqdcy2gi4py2 --discovery-token-ca-cert-hash sha256:2c4a029e1816adf4ac438a9fb02e97d134f59f9c4c9b1457f639744f59cd7ff7

注:记录kubeadm join... ,这是node节点加入集群的验证码

#我使用的root用户，建议使用普通用户
~]# mkdir -p $HOME/.kube
~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

2.4 部署pod网络插件flannel
~]# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml		#一键部署flannel的资源清单，由于我访问不了quay.io站点，修改镜像拉取地址（若需要其他版本，可到[阿里云](https://cr.console.aliyun.com/)或其他可访问到的仓库查找）


```
