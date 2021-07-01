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
















