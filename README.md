**该脚本用于处理已过期或者即将过期的 kubernetes 集群证书**  

**该脚本适用于所有 k8s 版本集群证书更新(使用 kubeadm 初始化的集群)**  

kubeadm 生成的证书有效期为 1 年，该脚本可将 kubeadm 生成的证书有效期更新为 10 年  

该脚本只处理 master 节点上的证书，node 节点的 kubelet 证书默认自动轮换更新，无需关心过期问题，只需关心 master 节点上的证书即可  

# 1. 使用说明

**该脚本仅需要在 master 节点执行，无需在 node 节点执行**  

- 若没有 etcd 相关证书，只需要更新 master 证书即可，[见这里](/other.md#1-只更新-master-证书)（小于等于 `v1.9` 版本，etcd 默认不使用 TLS 连接）

- 默认情况按照下面步骤进行证书更新

## 1.1. 拉取脚本

```
git clone https://github.com/yuyicai/update-kube-cert.git
cd update-kube-cert
chmod 755 update-kubeadm-cert.sh
```
执行时请使用 `./update-kubeadm-cert.sh all` 或者 `bash update-kubeadm-cert.sh all` ，不要使用 `sh update-kubeadm-cert.sh all`，因为某些 Linux 发行版 sh 并不是链接到 bash，可能会不兼容  

## 1.2. 更新证书  
如果有多个 master 节点，在每个 master 节点都执行一次  

执行命令：

```
./update-kubeadm-cert.sh all
```
将更新以下证书和 kubeconfig 配置文件  
```
/etc/kubernetes
├── admin.conf
├── controller-manager.conf
├── scheduler.conf
├── kubelet.conf
└── pki
    ├── apiserver.crt
    ├── apiserver-etcd-client.crt
    ├── apiserver-kubelet-client.crt
    ├── front-proxy-client.crt
    └── etcd
        ├── healthcheck-client.crt
        ├── peer.crt
        └── server.crt
```

[更多](/other.md)

# 2. 证书更新失败回滚

脚本会自动备份 `/etc/kubernetes` 目录到 `/etc/kubernetes.old-$(date +%Y%m%d)` 目录（备份目录命名示例：`kubernetes.old-20200325`）

若更新证书失败需要回滚，手动将备份 `/etc/kubernetes.old-$(date +%Y%m%d)`目录覆盖 `/etc/kubernetes` 目录  



# 3. 其他

**以下内容与该脚本无关，只是啰嗦几句**

大于等于 `v1.15` 的版本可直接使用 `kubeadm alpha certs renew <cert_name>` 来更新证书有效期，更新后证书有效期延长 1 年 

若小于小于 `v1.17` 版本实用 `kubeadm alpha certs renew <cert_name>` 来更新证书，需要手动处理一下，这是一个 [bug](<https://github.com/kubernetes/kubeadm/issues/1753>)  

若使用该脚本更新证书，无需再手动处理，可忽略该 bug  

`kubeadm alpha certs renew <cert_name>` 后手动处理[见这里](/other.md#4-kubeadm-命令更新证书手动处理)  

