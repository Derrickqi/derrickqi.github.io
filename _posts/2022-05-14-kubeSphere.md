---
layout:     post   				    # 使用的布局（不需要改）
title:      Centos多节点安装KubeSphere			# 标题 
subtitle:    All-in-One 模式安装 KubeSphere #副标题
date:       2022-05-14 				# 时间
author:     Derrick 				# 作者
header-img: img/post-bg-kubeSphere.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - kubernetes
---


# Linux多节点以All-in-One模式安装KubeSphere




## 1. 概述

KubeSphere 是在 Kubernetes 之上构建的面向云原生应用的分布式操作系统，完全开源，支持多云与多集群管理，提供全栈的 IT 自动化运维能力，简化企业的 DevOps 工作流。它的架构可以非常方便地使第三方应用与云原生生态组件进行即插即用 (plug-and-play) 的集成。
<br/><br/><br/>
## 2. 准备基本环境


| 角色       | IP            |
| ---------- | ------------- |
| k8s-master | 192.168.1.10 |
| k8s-node1  | 192.168.1.11 |
| k8s-node2  | 192.168.1.12 |

环境初始化(仅在master机器)与安装k8s相同详细请参考👉<a href="http://www.seven7nu.website/2022/01/13/k8sCreate/">使用kubeadm快速部署kubernetesV1.23.1集群</a>
<br/><br/><br/>
## 3. 准备KubeSphere环境

### 3.1 下载 KubeKey
```
#下载kk脚本并赋予可执行权限
curl -sfL https://get-kk.kubesphere.io | VERSION=v1.2.0 sh -

chmod +x kk
```

<br/>

### 3.2 创建多节点集群
```
./kk create cluster --with-kubernetes v1.21.5 --with-kubesphere v3.2.1
```



**系统将创建默认的 config-sample.yaml 文件。您可以根据您的环境修改此文件**
```
cat config-sample.yaml

apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: config-sample
spec:
  hosts:
  - {name: master, address: 192.168.1.10, internalAddress: 192.168.1.10, user: root, password: passwd}
  - {name: node1, address: 192.168.1.11, internalAddress: 192.168.1.11, user: root, password: passwd}
  - {name: node2, address: 192.168.1.12, internalAddress: 192.168.1.12, user: root, password: passwd}
  roleGroups:
    etcd:
    - master
    master:
    - master
    worker:
    - node1
    - node2
```



**备注**
```
安装 KubeSphere 3.2.1 的建议 Kubernetes 版本：1.19.x、1.20.x、1.21.x 或 1.22.x（实验性支持）。如果不指定 Kubernetes 版本，KubeKey 将默认安装 Kubernetes v1.21.5。有关受支持的 Kubernetes 版本的更多信息，请参见支持矩阵。

一般来说，对于 All-in-One 安装，您无需更改任何配置。

如果您在这一步的命令中不添加标志 --with-kubesphere，则不会部署 KubeSphere，KubeKey 将只安装 Kubernetes。如果您添加标志 --with-kubesphere 时不指定 KubeSphere 版本，则会安装最新版本的 KubeSphere。

KubeKey 会默认安装 OpenEBS 为开发和测试环境提供 LocalPV 以方便新用户。对于其他存储类型，请参见持久化存储配置。

```



<br/>

### 3.3 使用自定义的配置文件创建集群

**先安装一个前置工具(master和node都要安装)**
```
yum install -y conntrack socat

./kk create cluster -f config-sample.yaml
```

**打印信息**
```
+---------+------+------+---------+----------+-------+-------+-----------+---------+------------+-------------+------------------+--------------+
| name    | sudo | curl | openssl | ebtables | socat | ipset | conntrack | docker  | nfs client | ceph client | glusterfs client | time         |
+---------+------+------+---------+----------+-------+-------+-----------+---------+------------+-------------+------------------+--------------+
| worker2 | y    | y    | y       | y        |       | y     | y         | 20.10.7 |            |             |                  | CST 15:48:21 |
| worker1 | y    | y    | y       | y        |       | y     | y         | 20.10.7 |            |             |                  | CST 15:48:21 |
| master  | y    | y    | y       | y        |       | y     | y         | 20.10.7 |            |             |                  | CST 15:48:21 |
+---------+------+------+---------+----------+-------+-------+-----------+---------+------------+-------------+------------------+--------------+

This is a simple check of your environment.
Before installation, you should ensure that your machines meet all requirements specified at
https://github.com/kubesphere/kubekey#requirements-and-recommendations

Continue this installation? [yes/no]: yes
INFO[21:43:30 CST] Downloading Installation Files               
INFO[21:43:30 CST] Downloading kubeadm ...                      
INFO[21:43:32 CST] Downloading kubelet ...                      
INFO[21:43:34 CST] Downloading kubectl ...                      
INFO[21:43:35 CST] Downloading helm ...                         
INFO[21:43:36 CST] Downloading kubecni ...                      
INFO[21:43:36 CST] Downloading etcd ...                         
INFO[21:43:37 CST] Downloading docker ...                       
INFO[21:43:37 CST] Downloading crictl ...                       
INFO[21:43:38 CST] Configuring operating system ...             
[node1 192.168.1.11] MSG:
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-arptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_local_reserved_ports = 30000-32767
vm.max_map_count = 262144
vm.swappiness = 1
fs.inotify.max_user_instances = 524288
[master 192.168.1.10] MSG:
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-arptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_local_reserved_ports = 30000-32767
vm.max_map_count = 262144
vm.swappiness = 1
fs.inotify.max_user_instances = 524288
no crontab for root
INFO[21:44:47 CST] Get cluster status                           
INFO[21:44:48 CST] Installing Container Runtime ...             
INFO[21:44:49 CST] Start to download images on all nodes        
[node1] Downloading image: kubesphere/pause:3.4.1
[master] Downloading image: kubesphere/pause:3.4.1
[node1] Downloading image: kubesphere/kube-proxy:v1.21.5
[master] Downloading image: kubesphere/kube-apiserver:v1.21.5
[node1] Downloading image: coredns/coredns:1.8.0
[node1] Downloading image: kubesphere/k8s-dns-node-cache:1.15.12
[master] Downloading image: kubesphere/kube-controller-manager:v1.21.5
[node1] Downloading image: calico/kube-controllers:v3.20.0
[node1] Downloading image: calico/cni:v3.20.0
[node1] Downloading image: calico/node:v3.20.0
[master] Downloading image: kubesphere/kube-scheduler:v1.21.5
[node1] Downloading image: calico/pod2daemon-flexvol:v3.20.0
[master] Downloading image: kubesphere/kube-proxy:v1.21.5
[master] Downloading image: coredns/coredns:1.8.0
[master] Downloading image: kubesphere/k8s-dns-node-cache:1.15.12
[master] Downloading image: calico/kube-controllers:v3.20.0
[master] Downloading image: calico/cni:v3.20.0
[master] Downloading image: calico/node:v3.20.0
[master] Downloading image: calico/pod2daemon-flexvol:v3.20.0
INFO[21:47:54 CST] Getting etcd status                          
[master 192.168.1.10] MSG:
Configuration file will be created
INFO[21:47:55 CST] Generating etcd certs                        
INFO[21:47:57 CST] Synchronizing etcd certs                     
INFO[21:47:57 CST] Creating etcd service                        
Push /root/kubekey/v1.21.5/amd64/etcd-v3.4.13-linux-amd64.tar.gz to 192.168.1.10:/tmp/kubekey/etcd-v3.4.13-linux-amd64.tar.gz   Done
INFO[21:47:58 CST] Starting etcd cluster                        
INFO[21:47:59 CST] Refreshing etcd configuration                
[master 192.168.1.10] MSG:
Created symlink from /etc/systemd/system/multi-user.target.wants/etcd.service to /etc/systemd/system/etcd.service.
INFO[21:48:02 CST] Backup etcd data regularly                   
INFO[21:48:09 CST] Installing kube binaries                     
Push /root/kubekey/v1.21.5/amd64/kubeadm to 192.168.1.11:/tmp/kubekey/kubeadm   Done
Push /root/kubekey/v1.21.5/amd64/kubeadm to 192.168.1.10:/tmp/kubekey/kubeadm   Done
Push /root/kubekey/v1.21.5/amd64/kubelet to 192.168.1.10:/tmp/kubekey/kubelet   Done
Push /root/kubekey/v1.21.5/amd64/kubelet to 192.168.1.11:/tmp/kubekey/kubelet   Done
Push /root/kubekey/v1.21.5/amd64/kubectl to 192.168.1.11:/tmp/kubekey/kubectl   Done
Push /root/kubekey/v1.21.5/amd64/kubectl to 192.168.1.10:/tmp/kubekey/kubectl   Done
Push /root/kubekey/v1.21.5/amd64/helm to 192.168.1.11:/tmp/kubekey/helm   Done
Push /root/kubekey/v1.21.5/amd64/helm to 192.168.1.10:/tmp/kubekey/helm   Done
Push /root/kubekey/v1.21.5/amd64/cni-plugins-linux-amd64-v0.9.1.tgz to 192.168.1.10:/tmp/kubekey/cni-plugins-linux-amd64-v0.9.1.tgz   Done
Push /root/kubekey/v1.21.5/amd64/cni-plugins-linux-amd64-v0.9.1.tgz to 192.168.1.11:/tmp/kubekey/cni-plugins-linux-amd64-v0.9.1.tgz   Done
INFO[21:48:22 CST] Initializing kubernetes cluster              
[master 192.168.1.10] MSG:
W0514 21:48:24.657465   13122 utils.go:69] The recommended value for "clusterDNS" in "KubeletConfiguration" is: [10.233.0.10]; the provided value is: [169.254.25.10]
[init] Using Kubernetes version: v1.21.5
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local lb.kubesphere.local localhost master master.cluster.local node1 node1.cluster.local] and IPs [10.233.0.1 192.168.1.10 127.0.0.1 192.168.1.11]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] External etcd mode: Skipping etcd/ca certificate authority generation
[certs] External etcd mode: Skipping etcd/server certificate generation
[certs] External etcd mode: Skipping etcd/peer certificate generation
[certs] External etcd mode: Skipping etcd/healthcheck-client certificate generation
[certs] External etcd mode: Skipping apiserver-etcd-client certificate generation
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 15.006442 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.21" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: lhjm9r.q8j9mg5oc6q7mxh8
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join lb.kubesphere.local:6443 --token lhjm9r.q8j9mg5oc6q7mxh8 \
        --discovery-token-ca-cert-hash sha256:31b4841e7da671b91f3ca5869d450b04a6abaf136f272724cf810d8ed7a8df3f \
        --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join lb.kubesphere.local:6443 --token lhjm9r.q8j9mg5oc6q7mxh8 \
        --discovery-token-ca-cert-hash sha256:31b4841e7da671b91f3ca5869d450b04a6abaf136f272724cf810d8ed7a8df3f
[master 192.168.1.10] MSG:
service "kube-dns" deleted
[master 192.168.1.10] MSG:
service/coredns created
Warning: resource clusterroles/system:coredns is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
clusterrole.rbac.authorization.k8s.io/system:coredns configured
[master 192.168.1.10] MSG:
serviceaccount/nodelocaldns created
daemonset.apps/nodelocaldns created
[master 192.168.1.10] MSG:
configmap/nodelocaldns created
INFO[21:49:13 CST] Get cluster status                           
INFO[21:49:16 CST] Joining nodes to cluster                     
[node1 192.168.1.11] MSG:
[preflight] Running pre-flight checks
W0514 21:49:18.017187   33278 removeetcdmember.go:79] [reset] No kubeadm config, using etcd pod spec to get data directory
[reset] No etcd config found. Assuming external etcd
[reset] Please, manually reset etcd to prevent further issues
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
[reset] Deleting contents of stateful directories: [/var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
[node1 192.168.1.11] MSG:
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W0514 21:49:22.238819   33564 utils.go:69] The recommended value for "clusterDNS" in "KubeletConfiguration" is: [10.233.0.10]; the provided value is: [169.254.25.10]
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
[node1 192.168.1.11] MSG:
node/node1 labeled
INFO[21:49:30 CST] Deploying network plugin ...                 
[master 192.168.1.10] MSG:
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
Warning: policy/v1beta1 PodDisruptionBudget is deprecated in v1.21+, unavailable in v1.25+; use policy/v1 PodDisruptionBudget
poddisruptionbudget.policy/calico-kube-controllers created
[master 192.168.1.10] MSG:
storageclass.storage.k8s.io/local created
serviceaccount/openebs-maya-operator created
clusterrole.rbac.authorization.k8s.io/openebs-maya-operator created
clusterrolebinding.rbac.authorization.k8s.io/openebs-maya-operator created
deployment.apps/openebs-localpv-provisioner created
INFO[21:49:37 CST] Deploying KubeSphere ...                     
v3.2.0
[master 192.168.1.10] MSG:
namespace/kubesphere-system created
namespace/kubesphere-monitoring-system created
[master 192.168.1.10] MSG:
secret/kube-etcd-client-certs created
[master 192.168.1.10] MSG:
namespace/kubesphere-system unchanged
serviceaccount/ks-installer unchanged
customresourcedefinition.apiextensions.k8s.io/clusterconfigurations.installer.kubesphere.io unchanged
clusterrole.rbac.authorization.k8s.io/ks-installer unchanged
clusterrolebinding.rbac.authorization.k8s.io/ks-installer unchanged
deployment.apps/ks-installer unchanged
clusterconfiguration.installer.kubesphere.io/ks-installer created
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://192.168.1.10:30880
Account: admin
Password: P@88w0rd

NOTES：
  1. After you log into the console, please check the
     monitoring status of service components in
     "Cluster Management". If any service is not
     ready, please wait patiently until all components 
     are up and running.
  2. Please change the default password after login.

#####################################################
https://kubesphere.io             2022-05-14 21:59:58
#####################################################
INFO[22:00:00 CST] Installation is complete.

Please check the result using the command:

       kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```


<br/>

### 3.4 登录测试

```
Console: http://192.168.1.10:30880
Account: admin
Password: P@88w0rd
```



<br/><br/><br/>
## 4. 关于踩坑
 
### 4.1 Error:Failed to download kube binaries
安装时候总是报错`Error:Failed to download kube binaries:......`

**解决方法:由于网络延迟导致下载失败;可以尝试手动下载安装到本地,然后上传到服务器,重新执行脚本**
```
# ls -1 kubeSphere_file

kubelet
kubectl
kubeadm
helm
docker-20.10.8.tgz
crictl-v1.22.0-linux-amd64.tar.gz
etcd-v3.4.13-linux-amd64.tar
cni-plugins-linux-amd64-v0.9.1.tgz


# cp -r kubeSphere_file/* /root/kubekey/v1.21.5/amd64
```
<br/>

### 4.2 Please wait for the installation to complete

安装时一直卡在`Please wait for the installation to complete <<----`

**解决方法:磁盘空间不足导致,可以尝试扩充磁盘空间**






<br/><br/>
**👉👉👉👉喜欢作者的文章可以给我点个start哦!!!!**