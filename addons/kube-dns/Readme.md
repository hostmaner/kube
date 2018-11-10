# 1、说明
kube-dns-v1.14.13.yaml 将Service，ServiceAccount，ConfigMap，Deployment等4中服务放置在1个yaml文件中




clusterIP与kubelet启动参数--cluster-dns一致即可，在service cidr(在APIserver启动参数)中预选1个地址做dns地址
修改后的kube-dns.yaml：
