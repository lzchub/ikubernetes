1-set-labels-pod.yaml                 --> 设置标签选择器生成pod
2-set-env-pod.yaml                    --> 设置环境变量生成pod
3-expose-port-pod.yaml                --> pod暴露端口给集群外访问，2中方式
4-security-context-runasuser-pod.yaml --> 限制pod以普通用户运行(安全上下文)
5-security-context-sysctl-pod.yaml    --> 控制pod设置内核参数，没有安全上下文权限将会无法创建
6-liveness-exec-pod.yaml              --> 存活性探测，通过命令返回确定容器是否就绪
7-liveness-tcpsocket-pod.yaml         --> 端口存活性探测，通过修改iptables规则拒绝入栈请求
8-liveness-httpget-pod.yaml           --> 通过访问响应码探测服务是否健康，2xx,3xx正常，4xx，5xx错误
9-readiness-exec-pod.yaml             --> 就绪性探测，当程序未就绪时，service会将其提出，就绪后添加回来，此为通过命令探测
10-readiness-httpget-pod.yaml         --> 就绪性探测，通过访问某个资源返回响应码做出判断
11-lifecycle-pod.yaml                 --> 生命周期容器，在容器创建或者容器销毁之前做一些操作
12-initcontainer-pod.yaml             --> 初始化容器功能完成后会退出，此时主容器才会启动
13-source-limit-pod.yaml              --> 资源限制，上下阈值，超过阈值会被OOM
14-define-cmd-pod.yaml                --> 设置默认执行命令的两种形式
15-nodeselector-pod.yaml              --> 节点选择器，可以选择执行在某个节点上

all-in-one-pod.yaml                   --> Pod所有知识糅合到一个yaml文件
