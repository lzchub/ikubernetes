1-clusterIP-service.yaml --> clusterip类型的service
2-clusterIP-external-service.yaml --> 指定外部IP作为流量入口，默认会在所有机器上监听
3-nodeport-service.yaml --> nodeport类型service，集群外流量引入，每个node上都会监听暴露的端口
4-loadbalancer-service.yaml --> 公有云服务商使用
5-bindendpoint-service.yaml --> 手动创建endpoint，并绑定service,endpoint的名称需与service相同
6-headless-service.yaml --> 无头service，svc无分配IP，多用于有状态应用
