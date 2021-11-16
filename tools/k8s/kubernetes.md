#### 资源
* [k8s linux文档](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
* [k8s 下载地址](https://kubernetes.io/releases/download/)


#### Linux安装k8s
##### 下载k8s
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
* 安装kubeadm
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
yum install -y kubelet-1.11.2 kubeadm-1.11.2 kubectl-1.11.2 ipvsadm
```

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
























##### 安装k8s
* 安装
```shell script
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
# 设置在root组root用户下并赋予读、写、执行权限
```

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
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479", GitTreeState:"clean", BuildDate:"2021-04-08T16:31:21Z", GoVersion:"go1.16.1", Compiler:"gc", Platform:"linux/amd64"}
```
















