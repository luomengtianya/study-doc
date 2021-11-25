#### 资源
* [k8s linux文档](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
* [k8s 下载地址](https://kubernetes.io/releases/download/)


#### Linux安装k8s
##### 下载kubectl（可先不安装，后续也会安装上）
* 下载最新版本
```shell script
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

* 下载指定版本
```shell script
curl -LO https://dl.k8s.io/release/v1.21.0/bin/linux/amd64/kubectl
```

##### 校验下载数据
* 下载 `kubectl checksum`
```shell script
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```
或者
```shell script
curl -LO https://dl.k8s.io/v1.21.0/bin/linux/amd64/kubectl.sha256
```
* 校验文件
```shell script
echo "$(<kubectl.sha256) kubectl" | sha256sum --check
```

* 校验成功返回数据
```text
kubectl: OK
```

* 校验失败返回数据
```text
kubectl: FAILED
sha256sum: WARNING: 1 computed checksum did NOT match
```

* 注
```text
需要下载同一版本的kubernetes和checksum
```

##### 验证kubectl配置
* 为了让 kubectl 能发现并访问 Kubernetes 集群，你需要一个 kubeconfig 文件， 该文件在 kube-up.sh 创建集群时，或成功部署一个 Miniube 集群时，均会自动生成。 通常，kubectl 的配置信息存放于文件 ~/.kube/config 中。
```text
The connection to the server <server-name:port> was refused - did you specify the right host or port?
（访问 <server-name:port> 被拒绝 - 你指定的主机和端口是否有误？）
```
如果你看到如下所示的消息，则代表 kubectl 配置出了问题，或无法连接到 Kubernetes 集群 
如果返回一个 URL，则意味着 kubectl 成功的访问到了你的集群。


##### kubectl的自动补全插件
kubectl 的 Bash 补全脚本可以用命令 kubectl completion bash 生成。 在 shell 中导入（Sourcing）补全脚本，将启用 kubectl 自动补全功能。
可以使用
```shell script
yum install bash-completion
```
命令来安装
上述命令将创建文件 /usr/share/bash-completion/bash_completion，它是 bash-completion 的主脚本。 依据包管理工具的实际情况，你需要在 ~/.bashrc 文件中手工导入此文件。
要查看结果，请重新加载你的 shell，并运行命令 type _init_completion。 如果命令执行成功，则设置完成，否则将下面内容添加到文件 ~/.bashrc 中：
```shell script
export="$PATH:/usr/share/bash-completion/bash_completion"
```
保存后加载配置
```shell script
source ~/.bashrc
```
```shell script
source /usr/share/bash-completion/bash_completion
```

启动 kubectl 自动补全功能
你现在需要确保一点：kubectl 补全脚本已经导入（sourced）到 shell 会话中。 这里有两种验证方法：

* 在文件 ~/.bashrc 中导入（source）补全脚本：
```shell script
echo 'source <(kubectl completion bash)' >>~/.bashrc
```
* 将补全脚本添加到目录 /etc/bash_completion.d 中：
```shell script
kubectl completion bash >/etc/bash_completion.d/kubectl
```
如果 kubectl 有关联的别名，你可以扩展 shell 补全来适配此别名：

echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc



#### 安装kubeadm
##### 什么是kubeadm
```text
你可以使用 kubeadm 工具来 创建和管理 Kubernetes 集群。 该工具能够执行必要的动作并用一种用户友好的方式启动一个可用的、安全的集群。

安装 kubeadm 展示了如何安装 kubeadm 的过程。 一旦安装了 kubeadm，你就可以使用它来 创建一个集群。
```

##### 安装kubeadm
* 确保节点上的mac地址和product_uuid的唯一性
    * 你可以使用命令 ip link 或 ifconfig -a 来获取网络接口的 MAC 地址
    * 可以使用 sudo cat /sys/class/dmi/id/product_uuid 命令对 product_uuid 校验
* 检查网络适配器
    * 如果有一个以上的网络适配器，同时 Kubernetes 组件通过默认路由不可达，可以预先添加 IP 路由规则，这样 Kubernetes 集群就可以通过对应的适配器完成连接    
* 允许iptables检查桥接流量
    确保 br_netfilter 模块被加载。这一操作可以通过运行 lsmod | grep br_netfilter 来完成。若要显式加载该模块，可执行 sudo modprobe br_netfilter。
    为了让你的 Linux 节点上的 iptables 能够正确地查看桥接流量，你需要确保在你的 sysctl 配置中将 net.bridge.bridge-nf-call-iptables 设置为 1
```shell script
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```
若是执行 `modprobe br_netfilter` 报

```
modprobe: FATAL: Module br_netfilter not found.
```

则需要安装模块

```
yum install bridge-utils  -y
```



* 安装kubeadm(没有外网会超时)

```shell script
# 安装依赖
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# 没有外网会超时
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet

```
* 使用阿里云镜像安装
```shell script
# 配置源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet
```



* 下载所需要的依赖

  ```
  kubeadm config images pull
  ```

  

* 国内下载依赖

  * 查询镜像

  ```
  kubeadm config images list
  ```

  

  * 下载

    ```
    # kube-adm-images.sh
    # 如下镜像列表和版本，请运行kubeadm config images list命令获取
    # 
    images=(
      kube-apiserver:v1.22.4
      kube-controller-manager:v1.22.4
      kube-scheduler:v1.22.4
      kube-proxy:v1.22.4
      pause:3.5
      etcd:3.5.0-0
      coredns/coredns:v1.8.4
    )
    
    for imageName in ${images[@]} ; do
        docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
        docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    done
    ```
  
  
  
  
    * 修改tag



* 注 

如果没有root的权限，可以把k8s导入到 ~/.local/bin 文件夹
```shell script
mkdir -p ~/.local/bin/kubectl
mv ./kubectl ~/.local/bin/kubectl
# and then add ~/.local/bin/kubectl to $PATH
```

* 检查是否导入成功
```shell script
kubectl version --client
```

* 成功则返回数据
```text
Client Version: version.Info{Major:"1", Minor:"22", GitVersion:"v1.22.4", GitCommit:"b695d79d4f967c403a96986f1750a35eb75e75f1", GitTreeState:"clean", BuildDate:"2021-11-17T15:48:33Z", GoVersion:"go1.16.10", Compiler:"gc", Platform:"linux/amd64"}
```



##### 启动k8s

* 使用 `kubeadm init` 排查错误
```text
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
```
配置
```shell script
tee <<EOF > /proc/sys/net/ipv4/ip_forward
1
EOF
```


* 阿里云镜像启动

  ```
  kubeadm init --image-repository registry.aliyuncs.com/google_containers
  ```

  

* 当所有问题解除后 运行`kubeadm init` 会启动 Kubernets

```text
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

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 9.134.242.16:6443 --token 5g9xeo.vlnywiwck0cqw78q \
        --discovery-token-ca-cert-hash sha256:81fa54d3f74991d62d1d74ebb34b091dc7c980e004288487aa2ce02fdd143140 
```

* 你必须部署一个基于 Pod 网络插件的 容器网络接口 (CNI)，以便你的 Pod 可以相互通信。 在安装网络之前，集群 DNS (CoreDNS) 将不会启动
```text
https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml
```
安装报错
```shell script
unable to recognize "https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml": no matches for kind "ClusterRole" in version "rbac.authorization.k8s.io/v1beta1"
unable to recognize "https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
unable to recognize "https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml": no matches for kind "DaemonSet" in version "extensions/v1beta1"
```
* 先添加全局权限
```shell script
kubectl create clusterrolebinding permissive-binding   --clusterrole=cluster-admin   --user=admin   --user=kubelet   --group=system:serviceaccounts
```
再下载修改文件



* 初始化master节点

  ```
  sudo kubeadm config print init-defaults
  sudo kubeadm config print join-defaults
  ```



* 查看kubelet的状态

  ```
  [root@VM-242-16-centos ~]# systemctl status kubelet
  ● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2021-11-24 11:13:48 CST; 7min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 29254 (kubelet)
     CGroup: /system.slice/kubelet.service
             └─29254 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.c...
  
  ```

  

##### 安装网络插件

* 下载

  ```
  curl -LO https://docs.projectcalico.org/manifests/calico.yaml
  ```



* 安装

  ```
   kubectl apply -f /root/calico.yaml
   
   configmap/calico-config created
  customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
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
  ```



* 查看

```
[root@VM-242-16-centos ~]# kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS              RESTARTS   AGE
kube-system   calico-kube-controllers-56b8f699d9-88pd4   0/1     Pending             0          41s
kube-system   calico-node-pmjct                          0/1     Init:ErrImagePull   0          41s
kube-system   coredns-78fcd69978-5w48s                   0/1     Pending             0          19m
kube-system   coredns-78fcd69978-9xcj8                   0/1     Pending             0          19m
kube-system   etcd-vm-242-16-centos                      1/1     Running             0          19m
kube-system   kube-apiserver-vm-242-16-centos            1/1     Running             0          19m
kube-system   kube-controller-manager-vm-242-16-centos   1/1     Running             0          19m
kube-system   kube-proxy-2cv82                           1/1     Running             0          19m
kube-system   kube-scheduler-vm-242-16-centos            1/1     Running             0          19m
```



```
[root@VM-242-16-centos ~]# kubectl logs calico-node-pmjct -n kube-system
Error from server (BadRequest): container "calico-node" in pod "calico-node-pmjct" is waiting to start: PodInitializing
```



这个可能是镜像没有下载成功，等一段时间可能就会自动自动完成

```
[root@VM-242-16-centos ~]# kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-56b8f699d9-88pd4   1/1     Running   0          3h37m
kube-system   calico-node-pmjct                          1/1     Running   0          3h37m
kube-system   coredns-78fcd69978-5w48s                   1/1     Running   0          3h56m
kube-system   coredns-78fcd69978-9xcj8                   1/1     Running   0          3h56m
kube-system   etcd-vm-242-16-centos                      1/1     Running   0          3h56m
kube-system   kube-apiserver-vm-242-16-centos            1/1     Running   0          3h56m
kube-system   kube-controller-manager-vm-242-16-centos   1/1     Running   0          3h56m
kube-system   kube-proxy-2cv82                           1/1     Running   0          3h56m
kube-system   kube-scheduler-vm-242-16-centos            1/1     Running   0          3h56m
```



##### 添加dashboard

* 下载文件

  ```
  curl -LO https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
  ```

  

* 修改NodePort映射

  ```
  [root@xuegod63 ~]# vim recommended.yaml
  kind: Service
  apiVersion: v1
  metadata:
    labels:
      k8s-app: kubernetes-dashboard
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard
  spec:
    ports:
      - port: 443
        targetPort: 8443
        nodePort: 30000
    type: NodePort
    selector:
      k8s-app: kubernetes-dashboard
      
      
  添加数据：
        nodePort: 30000
    type: NodePort
  
  ```

  

* 追加权限数据

  ```
  [root@xuegod63 ~]# cat >> recommended.yaml << EOF
  ---
  # ------------------- dashboard-admin ------------------- #
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: dashboard-admin
    namespace: kubernetes-dashboard
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: dashboard-admin
  subjects:
  - kind: ServiceAccount
    name: dashboard-admin
    namespace: kubernetes-dashboard
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: cluster-admin
  EOF
  ```

* ![image-20211124151632386](/Users/panjianghong/Documents/study-doc/tools/k8s/image-20211124151632386.png)



* 启动dashboard

  ```
  [root@VM-242-16-centos ~]# kubectl apply -f recommended.yaml
  namespace/kubernetes-dashboard created
  serviceaccount/kubernetes-dashboard created
  service/kubernetes-dashboard created
  secret/kubernetes-dashboard-certs created
  secret/kubernetes-dashboard-csrf created
  secret/kubernetes-dashboard-key-holder created
  configmap/kubernetes-dashboard-settings created
  role.rbac.authorization.k8s.io/kubernetes-dashboard created
  clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
  rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
  clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
  deployment.apps/kubernetes-dashboard created
  service/dashboard-metrics-scraper created
  Warning: spec.template.metadata.annotations[seccomp.security.alpha.kubernetes.io/pod]: deprecated since v1.19; use the "seccompProfile" field instead
  deployment.apps/dashboard-metrics-scraper created
  serviceaccount/dashboard-admin created
  clusterrolebinding.rbac.authorization.k8s.io/dashboard-admin created
  ```



* 查看安装

  ```
  [root@VM-16-12-centos ~]# kubectl get pods -A
  NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
  kube-system            calico-kube-controllers-56b8f699d9-xwbt8     1/1     Running   0          152m
  kube-system            calico-node-z26cq                            1/1     Running   0          152m
  kube-system            coredns-7f6cbbb7b8-7bq4p                     1/1     Running   0          153m
  kube-system            coredns-7f6cbbb7b8-ccb8x                     1/1     Running   0          153m
  kube-system            etcd-vm-16-12-centos                         1/1     Running   0          154m
  kube-system            kube-apiserver-vm-16-12-centos               1/1     Running   0          154m
  kube-system            kube-controller-manager-vm-16-12-centos      1/1     Running   0          154m
  kube-system            kube-proxy-9v4ng                             1/1     Running   0          153m
  kube-system            kube-scheduler-vm-16-12-centos               1/1     Running   0          154m
  kubernetes-dashboard   dashboard-metrics-scraper-5594697f48-gr794   1/1     Running   0          78m
  kubernetes-dashboard   kubernetes-dashboard-57c9bfc8c8-4xj6t        1/1     Running   0          78m
  ```



* 浏览网站

  ```
  https://49.235.198.77:30000/
  ```

  提示 `您的连接不是私密连接` 并限制了访问。这是因为https证书无效导致的，可以自己新建一个证书



* 查看是否安装成功

  ```
  [root@VM-16-12-centos ~]# kubectl get service -n kubernetes-dashboard
  NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
  dashboard-metrics-scraper   ClusterIP   10.103.242.208   <none>        8000/TCP        80m
  kubernetes-dashboard        NodePort    10.110.154.160   <none>        443:30000/TCP   80m
  ```



* 删除以前的老证书

  ```
  kubectl delete secret -n kubernetes-dashboard kubernetes-dashboard-certs
  ```



* 自己签发证书

```
mkdir keys & cd keys

openssl genrsa -out tls.key 2048

openssl req -new -out tls.csr -key tls.key -subj '/CN=49.235.198.77'

openssl x509 -req -in tls.csr -signkey tls.key -out tls.crt

ls
tls.crt tls.csr tls.key
```



* 根据新证书，创建secret

  ```
  cd keys # 如果本身就在keys文件夹下，则可以省略该步骤
  
  kubectl create secret generic kubernetes-dashboard-certs --from-file=./ -n kubernetes-dashboard
  ```



* 修改kubernetes-dashboard deployment,启用新的secret

  ```
   kubectl edit deploy kubernetes-dashboard -n kubernetes-dashboard
  ...
   31   template:
   32     metadata:
   33       creationTimestamp: null
   34       labels:
   35         k8s-app: kubernetes-dashboard
   36     spec:
   37       containers:
   38       - args:
   41         - --auto-generate-certificates
   42         - --namespace=kubernetes-dashboard
   43         image: kubernetesui/dashboard:v2.0.0
   44         imagePullPolicy: Always
  ...
  
  # 在args中添加两行
         containers:
         - args:
           - --tls-cert-file=/tls.crt
           - --tls-key-file=/tls.key
        
        
  # 添加之后
  ...
   31   template:
   32     metadata:
   33       creationTimestamp: null
   34       labels:
   35         k8s-app: kubernetes-dashboard
   36     spec:
   37       containers:
   38       - args:
   39         - --tls-cert-file=/tls.crt   < ----- 这里
   40         - --tls-key-file=/tls.key    < ----- 这里
   41         - --auto-generate-certificates
   42         - --namespace=kubernetes-dashboard
   43         image: kubernetesui/dashboard:v2.0.0
   44         imagePullPolicy: Always
   ...
  ```

  这种方式签发的证书也是不安全的，浏览器上会有红色的提示

  ![image-20211125114547930](/Users/panjianghong/Documents/study-doc/tools/k8s/image-20211125114547930.png)

​		因为这不是被信赖的机构颁发的证书，但是这只是我们自己使用的话，可以忽略这个问题；若是需要对外提供服务，最好还是使用被信赖的证书。



* 查看登陆令牌

  ```
  [root@VM-16-12-centos keys]# kubectl describe secrets -n kubernetes-dashboard dashboard-admin
  Name:         dashboard-admin-token-8h6tp
  Namespace:    kubernetes-dashboard
  Labels:       <none>
  Annotations:  kubernetes.io/service-account.name: dashboard-admin
                kubernetes.io/service-account.uid: 748973a3-baa1-434e-a5ea-0fad79711443
  
  Type:  kubernetes.io/service-account-token
  
  Data
  ====
  ca.crt:     1099 bytes
  namespace:  20 bytes
  token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImFJZFBHTV9VcHNhd215UGpWV3FCTVpnWUJZQmgxV1Z3LW45cTllUHBodkUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tOGg2dHAiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNzQ4OTczYTMtYmFhMS00MzRlLWE1ZWEtMGZhZDc5NzExNDQzIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmVybmV0ZXMtZGFzaGJvYXJkOmRhc2hib2FyZC1hZG1pbiJ9.P1SMmg432oZBufegLPrR0T-lYPkplCRiSvmPVCvGuln07epNCx-CFeqF-npIF0rzY9hlo3TFCIJpOzn83IHl9Nxz2HPFKx9ryD6dKLz9A19QL8DZu-wNSK9EGr_SKX-4NQQjPguP4qbBmjCtWOvhsn5z2C1Hs-uAiYAy2yGWnWSFm9jbNasfdj0hpI51_ZxJDOr4IN93EC3ejcdjy9e9IS9KR8a68tgW1jYz1o7PPjei6lvkyLl-G87cVkMVRMWZlvQQYf9btu_E_6mCbaR6A7J_ol7AOtThCNeRJER8ZZDgvsxde2ik5ys_oCpIct1X7Hy-IzNysawyoJodTAJq4Q
  ```

  

* 登陆网站并设置token数据

  ![image-20211124175201076](/Users/panjianghong/Documents/study-doc/tools/k8s/image-20211124175201076.png)





#### 节点加入集群

