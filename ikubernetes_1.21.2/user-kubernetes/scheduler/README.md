1-nodename-deploy.yaml --> 直接选定节点进行调度，预选阶段就只会剩下唯一节点，不满足运行条件直接挂起
2-nodeaffinity-required-deploy.yaml --> 亲和性调度，pod会选择满足条件的节点,硬亲和，若不满足条件，pod将会被挂起
3-nodeaffinity-preferred-deploy.yaml --> 节点亲和性调度，pod会选择满足条件的节点，软亲和，若不满足条件，也有可能被调度
4-podaffinity-required-deploy.yaml --> pod亲和性调度，根据相关的pod进行调度到对应节点上，硬亲和
5-podaffinity-preferred-deploy.yaml --> pod软亲和，根据相关的pod进行调度到对应节点上，但是并不是完全符合
6-podantiaffinity-required-deploy.yaml --> pod反亲和性，可以使pod分别运行在不同节点
