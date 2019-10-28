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
> 注:master的CPU至少2核,实验前请关闭所有节点防火墙和selinux
## 1.集群搭建
### 1.1 Master
```
1. 安装docker
~]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -P /etc/yum.repos.d/  #可选择对应版本rpm包直接安装
选择docker版本:
~]# yum list docker-ce --showduplicates
~]# yum install -y --setopt=obsoletes=0 docker-ce-18.06.1.ce-3.el7
镜像加速:由于国内拉取dockerhub上的镜像速度较慢或会拉取失败,使用国内镜像加速访问
~]# cat /etc/docker/daemon.json 
{
	"registry-mirrors":["https://registry.docker-cn.com"]
}

2. 集群搭建
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
		#--ignore-preflight-errors=Swap	\     #忽略 swap分区未关闭的错误,若使用法二需要添加此项
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
~]# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml		#一键部署flannel的资源清单，由于我访问不了quay.io站点，修改镜像拉取地址（具体可看我的清单,阿里云仓库(https://cr.console.aliyun.com/)或其他可访问到的仓库查找）
~]# kubectl apply -f kube-flannel.yml
~]# kubectl get nodes
...
k8s-master   Ready    master   2d6h   v1.13.3
```
### 1.2 Node
```
1.安装docker(同master)
2.加入集群
2.1 基础准备(参考master 2.1)
2.2 禁用swap(参考master 2.2)
2.3 加入集群
~]# kubeadm join 192.168.179.50:6443 --token z95afq.1lolsqdcy2gi4py2 --discovery-token-ca-cert-hash sha256:2c4a029e1816adf4ac438a9fb02e97d134f59f9c4c9b1457f639744f59cd7ff7

其他node配置相同
```
```
注:
kubeadm：是Kubernetes官方提供的用于快速安装Kubernetes集群的工具，伴随Kubernetes每个版本的发布都会同步更新，kubeadm会对集群配置方面的一些实践做调整，通过实验kubeadm可以学习到Kubernetes官方在集群配置上一些新的最佳实践。
kubelet：是运行于集群中每个节点上的kubernetesdialing程序，它的核心功能在于通过API Server获取调度至自身运行的Pod资源的PodSpec并依之运行Pod。事实上，以自托管方式部署的Kubernetes集群，除了kubelet和Docker之外的所有组件均以Pod对象的形式运行。
kubectl: 操作集群的命令行工具。通过 kubectl 可以部署和管理应用，查看各种资源，创建、删除和更新各种组件。
```
## 2.附件安装
### 2.1 ingress-nginx安装配置
```
service的nodeport只能实现基于端口的tcp四层代理，ingress可实现基于http/https的七层代理
常用反向代理：
	nginx、envoy、vulcand、haproxy、traefik

GitHub地址：https://github.com/kubernetes/ingress-nginx/tree/master/deploy

1.一键部署
~]# wget https://github.com/kubernetes/ingress-nginx/blob/master/deploy/mandatory.yaml
修改镜像拉取地址：registry.cn-hangzhou.aliyuncs.com/quay-image/nginx-ingress-controller:0.22.0

~]# kubectl apply -f mandatory.yaml
~]# kubectl get pod POD_NAME -n ingress-nginx

2.暴露服务
~]# kubectl apply -f service-nodeport.yaml
```
### 2.2 dashboard搭建
```
~]# cd /etc/kubernetes/pki/

~]# openssl genrsa -out dashboard.key 2048
~]# openssl req -new -key dashboard.key -out dashboard.csr -subj "/O=iK8s/CN=dashboard"
~]# openssl x509 -req -in dashboard.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out dashboard.crt -days 3650
~]# kubectl create secret generic kubernetes-dashboard-certs -n kube-system --from-file=dashboard.crt --from-file=dashboard.key



github地址：https://github.com/kubernetes/dashboard

1.获取资源清单
~]# wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

2.修改镜像拉取地址，暴露服务(使用nodeport暴露服务)
修改镜像拉取地址（若需要其他版本，可到阿里云（https://cr.console.aliyun.com/）或其他可访问到的仓库查找）：
registry.cn-hangzhou.aliyuncs.com/quay-image/kubernetes-dashboard:v1.10.1	

service中暴露端口：
		spec:
  		  type: NodePort
  	 	  ports:
   		  - port: 443
      		targetPort: 8443
      		nodePort: 30888
3.访问
  注意使用的是https，访问时需要为 https://IP:PORT，高版本google可能无法访问，可使用firefox或其他浏览器

4.认证
  支持 kubeconfig 和 令牌 认证，我这儿使用令牌认证
	
绑定管理员角色：
	~]# kubectl apply -f clusterrolebinding-dashboard.yaml
得到令牌：
	~]# kubectl describe secret -n kube-system `kubectl get secret -n kube-system | grep dashboard-admin | awk '{print $1}'`

5.得到token输入即可登录
```
### 2.3 部署canal提供网络策略功能
```
flannel插件提供了pod网络，但是没提供网络策略功能，需要借助calico来实现网络策略，而且官网也说了可以flannel+calico一起使用。
官网地址：https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/flannel

~]# wget https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml  #修改镜像拉取地址后直接部署即可
~]# kubectl apply -f canal.yaml
```


