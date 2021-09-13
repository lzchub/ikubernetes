1-replicaSet-controller.yaml --> replocaset控制器，最基础的控制器，遵循删除更新的原则
2-deployment-controller.yaml --> deplyment控制器，控制replicaset控制器
3-deployment-controller-v2.yaml --> 可定制更新策略，一般为滚动更新
4-daemonSet-controller.yaml --> 通过节点选择或者直接运行在每个节点，且只有一个副本，类似于守护进程
5-job-controller.yaml --> 不定时作业，运行完就会退出
6-job-controller-v2.yaml --> 不定时作业，运行完就会退出，可以指定多少个任务并行进行作业
7-cronJob-controller.yaml --> 定制周期性作业，类似于计划任务，在指定时间执行作业
