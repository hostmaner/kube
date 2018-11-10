# 1、说明

  官方地址：https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns

# 2、yaml文件说明

  kube-dns-v1.14.13.yaml 将Service，ServiceAccount，ConfigMap，Deployment等4中服务放置在1个yaml文件中

# 3、需要关注内容

  clusterIP与kubelet启动参数--cluster-dns一致即可，在service cidr(在APIserver启动参数)中预选1个地址做dns地址
修改后的kube-dns.yaml：
