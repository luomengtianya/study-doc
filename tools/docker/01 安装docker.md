* [docker的官方安装文档](https://docs.docker.com/engine/install/centos/)

* 卸载docker -- 官网方式，腾讯云服务器亲测不适用
```shell script
 sudo yum remove docker \
                 docker-client \
                 docker-client-latest \
                 docker-common \
                 docker-latest \
                 docker-latest-logrotate \
                 docker-logrotate \
                 docker-engine
```    
注：运行中的docker是不能用此方法卸载的，卸载前先查看docker的运行状态
```shell script
systemctl status docker
```
若是 Active: active(running)，则需要先停止docker
```shell script
systemctl stop docker
```
查看yum安装的docker文件包
```shell script
yum list installed | grep docker
```
查看docker的相关rpm源文件
```shell script
rpm -pa | grep docker
```
删除所有的docker安装文件包(按实际情况修改)
```shell script
yum -y remove containerd.io.x86_64
yum -y remove docker-ce.x86_64
yum -y remove docker-ce-cli.x86_64
```
删除docker的镜像文件，默认在/var/lib/docker目录下 
```shell script
rm -rf /var/lib/docker
```



* 安装docker
设置仓库
```shell script
sudo yum install -y yum-utils
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
安装docker
```shell script
sudo yum install docker-ce docker-ce-cli containerd.io
```
* [腾讯云服务器安装docker](https://cloud.tencent.com/document/product/213/46000?from=information.detail.%E8%85%BE%E8%AE%AF%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%89%E8%A3%85docker)
```shell script
yum update
yum install epel-release -y
yum clean all
yum list
```
```shell script
yum install docker-io -y
systemctl start docker
docker info
```
