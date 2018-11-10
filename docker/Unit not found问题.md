# docker.service: Unit not found

操作系统：Red Hat Enterprise Linux 7

### 原因1：docker.socket
最初在启动docker时遇到问题，是因为docker.socket引起的，虽然记不清问题是表现为Unit not found还是执行systemctl start docker.service命令时hang住了，但是也一并记录在这里。

##### 问题描述
我是从Docker 1.10.3升级到1.13.1版本，通过rpm包安装的。由于要保留自定义的一些Docker配置，所以在升级后，使用原来的/usr/lib/systemd/system/docker.service覆盖了新的docker.service。但是在1.10.3版本中，docker.service的[UNIT]里规定了Requires=docker.socket，也就是说，docker.service默认依赖于docker.socket，因为需要使用docker.socket来获取容器的信息。

[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target docker.socket
Requires=docker.socket
但是在1.13.1版本中，已经不再依赖于docker.socket了，所以系统里没有docker.socket，而我继续使用原来的docker.service，在启动时就会出错。

##### 解决办法
删除/usr/lib/systemd/system/docker.service的[UNIT]里包含的docker.socket，然后systemctl daemon-reload，最后systemctl start docker.service，发现启动成功了。

如果是类似的情况，缺少docker.socket，但是新版本需要docker.socket。有两种方法可以解决该问题:

可以卸载docker，再重新安装，即可出现docker.socket。

创建一个/usr/lib/systemd/system/docker.socket文件，然后systemctl daemon-reload，最后systemctl start docker.service，即可启动成功。

/usr/lib/systemd/system/docker.socket文件如下：

```
[Unit]
Description=Docker Socket for the API
PartOf=docker.service

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
```

### 原因2：flanneld.service
就如背景里描述的，我恰好在这台出问题的机器上，安装过Kubernetes，以及flannel，然后又删掉了我之前以为的“所有”相关的文件。正是由于flannel的文件没有删除干净，导致出现了docker.service: Unit not found的问题。

问题描述
在确定不是因为docker.socket的问题导致的之后，我第一反应就是删除flannel导致的，因为我了解flanneld.service与docker.service直接是有启动顺序的关联的：

[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service
真正困扰了我很久的是，/usr/lib/systemd/system/flanneld.service我已经删除了，也systemctl daemon-reload了，究竟还有哪个文件漏删了。

经过检查，/etc/systemd/system/flanneld.service依然存在，并且存在/etc/systemd/system/docker.service.requires目录，在该目录下包含了软连接flanneld.service，该软链接指向了真正的flanneld.service，从而实现了两个服务的启动顺序的关联。

定位该类问题，经常会用到的命令有：

systemctl list-unit-files 列出所有可用的Unit
systemctl list-units 列出所有正在运行的Unit
systemctl --failed 列出所有失败单元
systemctl mask httpd.service 禁用服务
systemctl unmask httpd.service
systemctl kill httpd 杀死服务
systemd-analyze critical-chain：分析启动时的关键链
systemd-analyze blame 分析启动时各个进程花费的时间
解决办法
使用systemctl unmask flanneld.service禁止flanneld服务，然后删除
/etc/systemd/system/docker.service.requires/flanneld.service，使用systemctl daemon-reload重新加载服务配置文件，最后systemctl start docker.service，发现docker启动成功了。
