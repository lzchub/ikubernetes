1-emptydir-volume.yaml --> 临时存储卷，容器销毁时就消失
2-hostpath-volume.yaml --> 与宿主机绑定的存储卷，宿主机上的目录与容器共享
3-nfs-volume.yaml --> 使用nfs网络存储，只要可以连接存储，容器可以随意迁移
