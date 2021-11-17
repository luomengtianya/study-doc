#### 什么是minikube
minikube是本地Kubernetes，注重于降低学习难度以及快速部署Kubernetes，与其相似的还有 [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) 

#### 学习文档
[minikube部署文档](https://minikube.sigs.k8s.io/docs/start/) 

[kubectl安装文档](https://kubernetes.io/zh/docs/tasks/tools/) 

[driver](https://minikube.sigs.k8s.io/docs/drivers/)

[网络插件](https://kubernetes.io/zh/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model)

#### 部署流程
* 在文档中按照自己的机器配置生成minikube的安装命令，并执行，如
```shell script
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

* 也可以选择安装阿里云的镜像
```shell script
curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.23.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

* 按照文档步骤，接下来就是执行`minikube start`启动minkube，但是我们不急，先执行`minikube start --help`查看一下启动文档
```text
Starts a local Kubernetes cluster

Options:
      --addons=[]: Enable addons. see `minikube addons list` for a list of valid addon names.
      --apiserver-ips=[]: A set of apiserver IP Addresses which are used in the generated certificate for kubernetes.
This can be used if you want to make the apiserver available from outside the machine
      --apiserver-name='minikubeCA': The authoritative apiserver hostname for apiserver certificates and connectivity.
This can be used if you want to make the apiserver available from outside the machine
      --apiserver-names=[]: A set of apiserver names which are used in the generated certificate for kubernetes.  This
can be used if you want to make the apiserver available from outside the machine
      --apiserver-port=8443: The apiserver listening port
      --auto-update-drivers=true: If set, automatically updates drivers to the latest version. Defaults to true.

--base-image='gcr.io/k8s-minikube/kicbase:v0.0.27@sha256:89b4738ee74ba28684676e176752277f0db46f57d27f0e08c3feec89311e22de':
The base image to use for docker/podman drivers. Intended for local development.
      --cache-images=true: If true, cache docker images for the current bootstrapper and load them into the machine.
Always false with --driver=none.
      --cni='': CNI plug-in to use. Valid options: auto, bridge, calico, cilium, flannel, kindnet, or path to a CNI
manifest (default: auto)
      --container-runtime='docker': The container runtime to be used (docker, cri-o, containerd).
      --cpus='2': Number of CPUs allocated to Kubernetes. Use "max" to use the maximum number of CPUs.
      --cri-socket='': The cri socket path to be used.
      --delete-on-failure=false: If set, delete the current cluster if start fails and try again. Defaults to false.
      --disable-driver-mounts=false: Disables the filesystem mounts provided by the hypervisors
      --disk-size='20000mb': Disk size allocated to the minikube VM (format: <number>[<unit>], where unit = b, k, m or
g).
      --dns-domain='cluster.local': The cluster dns domain name used in the Kubernetes cluster
      --dns-proxy=false: Enable proxy for NAT DNS requests (virtualbox driver only)
      --docker-env=[]: Environment variables to pass to the Docker daemon. (format: key=value)
      --docker-opt=[]: Specify arbitrary flags to pass to the Docker daemon. (format: key=value)
      --download-only=false: If true, only download and cache files for later use - don't install or start anything.
      --driver='': Driver is one of: virtualbox, vmwarefusion, kvm2, vmware, none, docker, podman, ssh (defaults to
auto-detect)
      --dry-run=false: dry-run mode. Validates configuration, but does not mutate system state
      --embed-certs=false: if true, will embed the certs in kubeconfig.
      --enable-default-cni=false: DEPRECATED: Replaced by --cni=bridge
      --extra-config=: A set of key=value pairs that describe configuration that may be passed to different components.
                The key should be '.' separated, and the first part before the dot is the component to apply the configuration to.
                Valid components are: kubelet, kubeadm, apiserver, controller-manager, etcd, proxy, scheduler
                Valid kubeadm parameters: ignore-preflight-errors, dry-run, kubeconfig, kubeconfig-dir, node-name, cri-socket,
experimental-upload-certs, certificate-key, rootfs, skip-phases, pod-network-cidr
      --extra-disks=0: Number of extra disks created and attached to the minikube VM (currently only implemented for
hyperkit and kvm2 drivers)
      --feature-gates='': A set of key=value pairs that describe feature gates for alpha/experimental features.
      --force=false: Force minikube to perform possibly dangerous operations
      --force-systemd=false: If set, force the container runtime to use systemd as cgroup manager. Defaults to false.
      --host-dns-resolver=true: Enable host resolver for NAT DNS requests (virtualbox driver only)
      --host-only-cidr='192.168.99.1/24': The CIDR to be used for the minikube VM (virtualbox driver only)
      --host-only-nic-type='virtio': NIC Type used for host only network. One of Am79C970A, Am79C973, 82540EM, 82543GC,
82545EM, or virtio (virtualbox driver only)
      --hyperkit-vpnkit-sock='': Location of the VPNKit socket used for networking. If empty, disables Hyperkit
VPNKitSock, if 'auto' uses Docker for Mac VPNKit connection, otherwise uses the specified VSock (hyperkit driver only)
      --hyperkit-vsock-ports=[]: List of guest VSock ports that should be exposed as sockets on the host (hyperkit
driver only)
      --hyperv-external-adapter='': External Adapter on which external switch will be created if no external switch is
found. (hyperv driver only)
      --hyperv-use-external-switch=false: Whether to use external switch over Default Switch if virtual switch not
explicitly specified. (hyperv driver only)
      --hyperv-virtual-switch='': The hyperv virtual switch name. Defaults to first found. (hyperv driver only)
      --image-mirror-country='cn': Country code of the image mirror to be used. Leave empty to use the global one. For
Chinese mainland users, set it to cn.
      --image-repository='': Alternative image repository to pull docker images from. This can be used when you have
limited access to gcr.io. Set it to "auto" to let minikube decide one for you. For Chinese mainland users, you may use
local gcr.io mirrors such as registry.cn-hangzhou.aliyuncs.com/google_containers
      --insecure-registry=[]: Insecure Docker registries to pass to the Docker daemon.  The default service CIDR range
will automatically be added.
      --install-addons=true: If set, install addons. Defaults to true.
      --interactive=true: Allow user prompts for more information

--iso-url=[https://storage.googleapis.com/minikube/iso/minikube-v1.23.1.iso,https://github.com/kubernetes/minikube/releases/download/v1.23.1/minikube-v1.23.1.iso,https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.23.1.iso]:
Locations to fetch the minikube ISO from.
      --keep-context=false: This will keep the existing kubectl context and will create a minikube context.
      --kubernetes-version='': The Kubernetes version that the minikube VM will use (ex: v1.2.3, 'stable' for v1.22.1,
'latest' for v1.22.2-rc.0). Defaults to 'stable'.
      --kvm-gpu=false: Enable experimental NVIDIA GPU support in minikube
      --kvm-hidden=false: Hide the hypervisor signature from the guest in minikube (kvm2 driver only)
      --kvm-network='default': The KVM default network name. (kvm2 driver only)
      --kvm-numa-count=1: Simulate numa node count in minikube, supported numa node count range is 1-8 (kvm2 driver
only)
      --kvm-qemu-uri='qemu:///system': The KVM QEMU connection URI. (kvm2 driver only)
      --listen-address='': IP Address to use to expose ports (docker and podman driver only)
      --memory='': Amount of RAM to allocate to Kubernetes (format: <number>[<unit>], where unit = b, k, m or g). Use
"max" to use the maximum amount of memory.
      --mount=false: This will start the mount daemon and automatically mount files into minikube.
      --mount-string='/root:/minikube-host': The argument to pass the minikube mount command on start.
      --namespace='default': The named space to activate after start
      --nat-nic-type='virtio': NIC Type used for nat network. One of Am79C970A, Am79C973, 82540EM, 82543GC, 82545EM, or
virtio (virtualbox driver only)
      --native-ssh=true: Use native Golang SSH client (default true). Set to 'false' to use the command line 'ssh'
command when accessing the docker machine. Useful for the machine drivers when they will not start with 'Waiting for
SSH'.
      --network='': network to run minikube with. Now it is used by docker/podman and KVM drivers. If left empty,
minikube will create a new network.
      --network-plugin='': Kubelet network plug-in to use (default: auto)
      --nfs-share=[]: Local folders to share with Guest via NFS mounts (hyperkit driver only)
      --nfs-shares-root='/nfsshares': Where to root the NFS Shares, defaults to /nfsshares (hyperkit driver only)
      --no-vtx-check=false: Disable checking for the availability of hardware virtualization before the vm is started
(virtualbox driver only)
  -n, --nodes=1: The number of nodes to spin up. Defaults to 1.
  -o, --output='text': Format to print stdout in. Options include: [text,json]
      --ports=[]: List of ports that should be exposed (docker and podman driver only)
      --preload=true: If set, download tarball of preloaded images if available to improve start time. Defaults to true.
      --registry-mirror=[]: Registry mirrors to pass to the Docker daemon
      --service-cluster-ip-range='10.96.0.0/12': The CIDR to be used for service cluster IPs.
      --ssh-ip-address='': IP address (ssh driver only)
      --ssh-key='': SSH key (ssh driver only)
      --ssh-port=22: SSH port (ssh driver only)
      --ssh-user='root': SSH user (ssh driver only)
      --trace='': Send trace events. Options include: [gcp]
      --uuid='': Provide VM UUID to restore MAC address (hyperkit driver only)
      --vm=false: Filter to use only VM Drivers
      --vm-driver='': DEPRECATED, use `driver` instead.
      --wait=[apiserver,system_pods]: comma separated list of Kubernetes components to verify and wait for after
starting a cluster. defaults to "apiserver,system_pods", available options:
"apiserver,system_pods,default_sa,apps_running,node_ready,kubelet" . other acceptable values are 'all' or 'none', 'true'
and 'false'
      --wait-timeout=6m0s: max time to wait per Kubernetes or host to be healthy.

Usage:
  minikube start [flags] [options]

Use "minikube options" for a list of global command-line options (applies to all commands).
```
输出的内容包括一些默认配置和特殊事项，如
```text
      --image-mirror-country='cn': Country code of the image mirror to be used. Leave empty to use the global one. For
Chinese mainland users, set it to cn.
      --image-repository='': Alternative image repository to pull docker images from. This can be used when you have
limited access to gcr.io. Set it to "auto" to let minikube decide one for you. For Chinese mainland users, you may use
local gcr.io mirrors such as registry.cn-hangzhou.aliyuncs.com/google_containers
```
minikube所需的镜像在拉取时，要求网络能够访问k8s.gcr.io。而此地址属于著名的404公司，在国内是无法访问的。而没有代理网络时，则需要按照提示添加启动配置

* 另外，minikube在启动时需要指定 [驱动](https://minikube.sigs.k8s.io/docs/drivers/) 我们这次选择按照docker的方式启动，执行命令
```text
minikube start --driver=docker --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' --image-mirror-country=cn --cni=flannel
```
当然，我们也可以指定默认的驱动
```shell script
minikube config set driver docker
```
或者使用阿里云下载的minikube镜像时，不需要指定`--image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers`，执行命令
```shell script
minikube start --registry-mirror=https://registry.docker-cn.com --image-mirror-country=cn --cni=flannel
```
其中，两个指令都有 `--cni=flannel` 这个参数，这是给minikube添加容器间的 [网络插件](https://kubernetes.io/zh/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model) 为接下来的 `dashboard` 启动做准备 

* 按照上述命令启动，会出现启动异常，报
```text
😄  minikube v1.23.1 on Centos 7.5.1804 (amd64)
✨  Automatically selected the docker driver. Other choices: none, ssh
💨  For improved Docker performance, Upgrade Docker to a newer version (Minimum recommended version is 18.09.0)
🛑  The "docker" driver should not be used with root privileges.
💡  If you are running minikube within a VM, consider using --driver=none:
📘    https://minikube.sigs.k8s.io/docs/reference/drivers/none/

❌  Exiting due to DRV_AS_ROOT: The "docker" driver should not be used with root privileges.
```
这是因为使用docker驱动启动时，不能使用root用户直接启动，所以需要创建一个用户
```shell script
useradd minikube
```

* 切换到minikube `su minikube` 执行命令还是会报错
```text
😄  minikube v1.22.0 on Centos 7.2 (amd64)
✨  Using the docker driver based on user configuration

💣  Exiting due to PROVIDER_DOCKER_NEWGRP: "docker version --format -" exit status 1: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.26/version: dial unix /var/run/docker.sock: connect: permission denied
💡  Suggestion: Add your user to the 'docker' group: 'sudo usermod -aG docker $USER && newgrp docker'
📘  Documentation: https://docs.docker.com/engine/install/linux-postinstall/
```
这是应为用户minikube没有在docker组，没有docker权限导致的，我们可以把用户添加到docker组，次操作需要在root用户下进行
```shell script
sudo usermod -aG docker minikube && newgrp docker
```
有可能会报docke组不存在，因为直接安装docker `yum install docker` 不会默认创建docker组，则自己建立一个
```shell script
groupadd docker
```
再次添加minikube到docker组，重启docker即可

* 再次切换minikube并执行启动命令，会出现
```text
😄  minikube v1.22.0 on Centos 7.2 (amd64)
✨  Using the docker driver based on user configuration
💨  For improved Docker performance, Upgrade Docker to a newer version (Minimum recommended version is 18.09.0)
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.21.2 preload ...
    > preloaded-images-k8s-v11-v1...: 502.14 MiB / 502.14 MiB  100.00% 27.45 Mi
    > gcr.io/k8s-minikube/kicbase...: 361.08 MiB / 361.09 MiB  100.00% 8.38 MiB
🔥  Creating docker container (CPUs=2, Memory=3900MB) ...
🐳  Preparing Kubernetes v1.21.2 on Docker 20.10.7 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
此时，minikube已经启动成功 

我们可以使用命令查看启动的pod
```
minikube kubectl -- get po -A
```
如果我们已经安装里kubectl，则可以直接使用 [安装文档](https://kubernetes.io/zh/docs/tasks/tools/)
```text
kubectl get pods -A
```
执行后会出现以下内容
```text
NAMESPACE              NAME                                         READY   STATUS    RESTARTS      AGE
kube-system            coredns-7d89d9b6b8-tlt54                     1/1     Running   0             18h
kube-system            etcd-minikube                                1/1     Running   0             18h
kube-system            kube-apiserver-minikube                      1/1     Running   0             18h
kube-system            kube-controller-manager-minikube             1/1     Running   0             18h
kube-system            kube-flannel-ds-amd64-wmvct                  1/1     Running   0             18h
kube-system            kube-proxy-mgdvn                             1/1     Running   0             18h
kube-system            kube-scheduler-minikube                      1/1     Running   0             18h
kube-system            storage-provisioner                          1/1     Running   1 (18h ago)   18h
```

* 执行命令 `minikube dashboard` 一段时间后会出现
```text
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:40104/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
👉  http://127.0.0.1:40104/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```
然后会一直停顿在这里，这是启动成功了，如果机器有可视化界面，我们直接在浏览器中打开 `http://127.0.0.1:40104/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/`
就可以看到kubernetes操作界面

如果只能命令行操作，我们再打开一个窗口，使用
```text
curl http://127.0.0.1:40104/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```
执行成功会返回
```text
<!--
Copyright 2017 The Kubernetes Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
--><!DOCTYPE html><html lang="en"><head>
  <meta charset="utf-8">
  <title>Kubernetes Dashboard</title>
  <link rel="icon" type="image/png" href="assets/images/kubernetes-logo.png">
  <meta name="viewport" content="width=device-width">
<style>body,html{height:100%;margin:0;}</style><link rel="stylesheet" href="styles.f66c655a05a456ae30f8.css" media="print" onload="this.media=&#39;all&#39;"><noscript><link rel="stylesheet" href="styles.f66c655a05a456ae30f8.css"></noscript></head>

<body>
  <kd-root></kd-root>
<script src="runtime.fb7fb9bb628f2208f9e9.js" defer=""></script><script src="polyfills.49b2d5227916caf47237.js" defer=""></script><script src="scripts.72d8a72221658f3278d3.js" defer=""></script><script src="en.main.0bf75cd6c71fc0efa001.js" defer=""></script>


</body></html>
```

* 若是执行 `minikube dashboard` 的界面关闭或者断开监听，上诉地址也会不能再被访问，我们执行 `minikube kubectl -- get po -A`
```shell script
NAMESPACE              NAME                                         READY   STATUS    RESTARTS      AGE
kube-system            coredns-7d89d9b6b8-tlt54                     1/1     Running   0             18h
kube-system            etcd-minikube                                1/1     Running   0             18h
kube-system            kube-apiserver-minikube                      1/1     Running   0             18h
kube-system            kube-controller-manager-minikube             1/1     Running   0             18h
kube-system            kube-flannel-ds-amd64-wmvct                  1/1     Running   0             18h
kube-system            kube-proxy-mgdvn                             1/1     Running   0             18h
kube-system            kube-scheduler-minikube                      1/1     Running   0             18h
kube-system            storage-provisioner                          1/1     Running   1 (18h ago)   18h
kubernetes-dashboard   dashboard-metrics-scraper-758548f849-hkg5r   1/1     Running   0             39s
kubernetes-dashboard   kubernetes-dashboard-586fb768c4-z7lpd        1/1     Running   0             39s
```
可以看到 dashboard 其实还在运行中，不能访问的原因是端口没有正确映射到容器中，我们可以通过代理，固化监听端口
```shell script
kubectl proxy
```
会出现
```shell script
Starting to serve on 127.0.0.1:8001
```
这个也会一直监听，我们把 8001 端口替换上次 dashboard 的界面，会发现此地址可以再次被访问;此方式是固定 8001 端口，我们也可以指定端口
```shell script
kubectl proxy  --port=8088 --address='127.0.0.1' --accept-hosts='^.*'
```
这个监听退出也会出现问题，我们可以使用后端运行的方式启动
```shell script
nohup kubectl proxy  --port=8088 --address='127.0.0.1' --accept-hosts='^.*' &
```

* 启动代理需要在启动 minikube 的用户下执行，不然会出现网络问题

* 此方式开启的 dashboard 只能在本机访问，不能进行外部访问，若是想外部访问，还需要配置转发等其它操作


