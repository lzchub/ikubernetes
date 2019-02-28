# ikubernetes
my k8s notes

> 这是我学习kubernetes的一点笔记与心得

##目录简介
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
