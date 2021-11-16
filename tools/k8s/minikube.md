### k8s文档
[部署文档](https://feisky.gitbooks.io/kubernetes/content/deploy/single.html)
[部署文档-中文](https://kubernetes.io/zh/docs/tasks/tools/)

## 单机版k8s[minikube](https://minikube.sigs.k8s.io/docs/start/)

### 查看linux紫铜架构
```shell script
uname -m
```

### 导入minikube
```shell script
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
使用阿里云的镜像
```shell script
curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.23.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

### 启动minikube
```shell script
minikube start
```
* 因为直接启动，会有一些网络的错误，所以指定网络启动
```shell script
minikube start --driver=docker --registry-mirror=https://registry.docker-cn.com --image-mirror-country=cn --cni=flannel
```
* 启动minikube不应该使用root账号，所以创建一个新用户
```shell script
useradd -m minikube
passwd minikube 
```
* 删除用户
```shell script
userdel minikube
```

* 用新用户启动还报错，可能是需要的驱动未安装完成 [驱动](https://minikube.sigs.k8s.io/docs/drivers/)
```text
😄  minikube v1.22.0 on Centos 7.2 (amd64)
👎  Unable to pick a default driver. Here is what was considered, in preference order:
    ▪ docker: Not healthy: "docker version --format {{.Server.Os}}-{{.Server.Version}}" exit status 1: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.26/version: dial unix /var/run/docker.sock: connect: permission denied
    ▪ docker: Suggestion: Add your user to the 'docker' group: 'sudo usermod -aG docker $USER && newgrp docker' <https://docs.docker.com/engine/install/linux-postinstall/>
    ▪ kvm2: Not installed: exec: "virsh": executable file not found in $PATH
    ▪ podman: Not installed: exec: "podman": executable file not found in $PATH
    ▪ vmware: Not installed: exec: "docker-machine-driver-vmware": executable file not found in $PATH
    ▪ virtualbox: Not installed: unable to find VBoxManage in $PATH

❌  Exiting due to DRV_NOT_HEALTHY: Found driver(s) but none were healthy. See above for suggestions how to fix installed drivers.
```
* 使用docker启动minikube
```shell script
minikube start --driver=docker
```

* 设置docker为minikube默认启动驱动
```shell script
minikube config set driver docker
```
* 启动时还是报错
```text
😄  minikube v1.22.0 on Centos 7.2 (amd64)
✨  Using the docker driver based on user configuration

💣  Exiting due to PROVIDER_DOCKER_NEWGRP: "docker version --format -" exit status 1: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.26/version: dial unix /var/run/docker.sock: connect: permission denied
💡  Suggestion: Add your user to the 'docker' group: 'sudo usermod -aG docker $USER && newgrp docker'
📘  Documentation: https://docs.docker.com/engine/install/linux-postinstall/
```
这是因为minikube没有docker连接的权限，需要把用户添加到docker组中
```text
sudo usermod -aG docker $USER && newgrp docker
```
如果是使用 yum install docker安装的docker，默认没有docker用户，则创建一个docker用户
```shell script
groupadd docker
```
再次把minikube导入docker组中
若是切换用户执行命令`docker images`还是报权限错误，切换到root下重启docker即可


可以使用docker version查询，未显示错误信息则表示成功了

* 查看组数据
```shell script
cat /etc/group
```

* 再次启动，加载一段时间后成功
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

### 查看minikube信息
```shell script
kubectl get po -A
或
minikube kubectl -- get po -A
```
显示信息
```text
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-558bd4d5db-xktpv           0/1     Running   0          35s
kube-system   etcd-minikube                      1/1     Running   0          42s
kube-system   kube-apiserver-minikube            1/1     Running   0          42s
kube-system   kube-controller-manager-minikube   1/1     Running   0          42s
kube-system   kube-proxy-sm74g                   1/1     Running   0          35s
kube-system   kube-scheduler-minikube            1/1     Running   0          42s
kube-system   storage-provisioner                1/1     Running   1          47s
```

* 启动minikube界面数据
```shell script
minikube dashboard
```
* 启动卡顿很久后报错
```text
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...

❌  Exiting due to SVC_URL_TIMEOUT: http://127.0.0.1:37708/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ is not accessible: Temporary Error: unexpected response code: 503
```


#### 设置固定的访问端口
* 使用kubectl的代理
```shell
kubectl proxy  --port=8088 --address='127.0.0.1' --accept-hosts='^.*'
```
这样就可以直接使用8088端口访问，但是只限于本机访问
```text
```text
http://127.0.0.1:8088/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```
```
或使用
```shell script
kubectl proxy
```
这样就可以使用8001端口访问
```text
http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```


```text
http://127.0.0.1:9090/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```












#### dashboard设置外网访问
```shell script
kubectl -n kubernetes-dashboard edit service kubernetes-dashboard
```
更改原文件type: ClusterIP 为type: NodePort后保存
* 下一步获取nodeport对外开放的https端口
```shell script
kubectl -n kubernetes-dashboard get service kubernetes-dashboard
```

```sql
[minikube@VM-16-12-centos root]$ minikube start
😄  minikube v1.24.0 on Centos 7.5.1804 (amd64)
❗  Both driver=docker and vm-driver=none have been set.

    Since vm-driver is deprecated, minikube will default to driver=docker.

    If vm-driver is set in the global config, please run "minikube config unset vm-driver" to resolve this warning.

✨  Using the docker driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
❗  minikube was unable to download gcr.io/k8s-minikube/kicbase:v0.0.28, but successfully downloaded docker.io/kicbase/stable:v0.0.28 as a fallback image
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
❗  This container is having trouble accessing https://k8s.gcr.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.22.3 on Docker 20.10.8 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```


#### 删除一个pod
```text
1、先删除pod

2、再删除对应的deployment

否则只是删除pod是不管用的，还会看到pod，因为deployment.yaml文件中定义了副本数量


实例如下：

删除pod

[root@test2 ~]# kubectl get pod -n jenkins
NAME                        READY     STATUS    RESTARTS   AGE
jenkins2-8698b5449c-grbdm   1/1       Running   0          8s
[root@test2 ~]# kubectl delete pod jenkins2-8698b5449c-grbdm -n jenkins
pod "jenkins2-8698b5449c-grbdm" deleted

查看pod仍然存储

[root@test2 ~]# kubectl get pod -n jenkins
NAME                        READY     STATUS    RESTARTS   AGE
jenkins2-8698b5449c-dbqqb   1/1       Running   0          8s
[root@test2 ~]# 

删除deployment

[root@test2 ~]# kubectl get deployment -n jenkins
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
jenkins2   1         1         1            1           17h
[root@test2 ~]# kubectl delete deployment jenkins2 -n jenkins

再次查看pod消失

deployment.extensions "jenkins2" deleted
[root@test2 ~]# kubectl get deployment -n jenkins
No resources found.
[root@test2 ~]# 
[root@test2 ~]# kubectl get pod -n jenkins
No resources found.
```

kubectl proxy  --port=8088 --address='49.235.198.77' --accept-hosts='^.*'



13416139728/yi067221
18810496240   1234qwer
10010015


@Jeff提供了完美的答案，为新手提供了更多提示。

使用@Jeff的脚本启动代理，默认情况下，它将在'0.0.0.0:8001'上打开代理。

kubectl proxy --address='0.0.0.0' --disable-filter=true
通过以下链接访问仪表板：

curl http://your_api_server_ip:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/
