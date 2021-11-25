#### 资料

* [registry](https://docs.docker.com/registry/deploying/)
* [docker-registry-web](https://hub.docker.com/r/hyper/docker-registry-web)
* [docker-daemon](https://docs.docker.com/engine/reference/commandline/dockerd/)
* [harbor](https://github.com/goharbor/harbor)
* [harbor安装](https://goharbor.io/docs/2.0.0/install-config/)

#### 查看仓库镜像

```shell
docker search registry
docker pull registry
```


#### 启动仓库镜像

```shell
$ docker run -d -p 5000:5000 --restart=always --name=docker-registry -v /var/local/docker-registry:/var/lib/registry registry
```


```
-d：后台运行
-p：将容器的5000端口映射到宿主机的5000端口
--restart：docker服务重启后总是重启此容器
--name：容器的名称
-v：将容器内的/var/lib/registry映射到宿主机的/var/local/docker-registry目录
```



#### 搭建web服务

##### 下载镜像

```shell
docker pull hyper/docker-registry-web
```

##### 启动服务

```shell
docker run -d -p 8080:8080 --restart=always --name registry-web --link docker-registry -e REGISTRY_URL=http://docker-registry:5000/v2 -e REGISTRY_NAME=localhost:5000 hyper/docker-registry-web
```

```
--link：链接其它容器(registry-srv)，在此容器中，使用registry-srv等同于registry-srv容器的局域网地址
-e：设置环境变量
```



#### 上传镜像

##### 打包tag 

打包的tag按照 registry为仓库地址的前缀数据，如 

```
REPOSITORY               TAG                           IMAGE ID       CREATED         SIZE
9.134.242.16:5000/blog   v20211120.111431.b79686.dev   f4d70d8410b4   9 seconds ago   22.2MB
```



##### push镜像

```
docker push 9.134.242.16:5000/blog:v20211120.111431.b79686.de
```

会报错

```
The push refers to repository [9.134.242.16:8080/blog]
Get https://9.134.242.16:8080/v2/: http: server gave HTTP response to HTTPS clie
```


##### push失败(https)问题

这是因为Docker在1.3.x之后默认docker registry使用的是https，为了解决这个问题，修改`本地主机`的docker启动配置文件

```shell
tee <<EOF >/etc/docker/daemon.json 
 {
   "insecure-registries" : ["9.134.242.16:5000"]
 }
EOF
```

之后重启docker即可

```
systemctl daemon-reload && systemctl restart docker
```


注意这是push或者pull的机器上配置修改，而不是私有化仓库中修改


macos的deamon.json在 `~/.docker` 文件夹下


另外，`linux`下还可以修改 `/etc/sysconfig/docker`

```
···
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false'
if [ -z "${DOCKER_CERT_PATH}" ]; then
    DOCKER_CERT_PATH=/etc/docker
fi
···
```

修改 `OPTIONS`,添加 `--insecure-registry=192.168.59.100:5000`


或者修改 `/usr/lib/systemd/system/docker.service`

```
···
Environment=PATH=/usr/libexec/docker:/usr/bin:/usr/sbin
ExecStart=/usr/bin/dockerd-current \
          --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current \
          --default-runtime=docker-runc \
          --exec-opt native.cgroupdriver=systemd \
          --userland-proxy-path=/usr/libexec/docker/docker-proxy-current \
          --init-path=/usr/libexec/docker/docker-init-current \
          --seccomp-profile=/etc/docker/seccomp.json \
          $OPTIONS \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $ADD_REGISTRY \
          $BLOCK_REGISTRY \
          $INSECURE_REGISTRY \
          $REGISTRIES
ExecReload=/bin/kill -s HUP $MAINPID
···
```

在 `ExecStart`中添加一行配置 `--insecure-registry=192.168.59.100:5000 \`



##### 创建一个TLS

`https访问-问题较多，私有仓库可以忽略`

使用上述的方式修改https为http的方式，是一种简单粗暴的方法，我们可以为registry添加TLS(*Transport Layer Security*)访问权限。这需要

* 我们的路由、防火墙、DNS支持以443的端口访问registry
* 提前准备好CA(certificate authority)证书

我们设置的CA数据为`test.crt`以及`test.key`

```
 docker run -d \
--restart=always \
--name=docker-registry \
-v /var/local/docker-registry:/var/lib/registry \
-v /var/local/jgaonet/Apache:/certs \
-e REGISTRY_HTTP_ADDR=0.0.0.0:5000 \
-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/2_jgaonet.com.crt \
-e REGISTRY_HTTP_TLS_KEY=/certs/3_jgaonet.com.key \
-p 5000:5000 \
registry
```



##### pull镜像

```shell
docker pull 9.134.242.16:5000/blog:v20211120.113355.b79686.dev
```

若是机器设置了insecure-registries配置，会直接下载成功



##### 复制一个镜像到本地仓库

###### 下载镜像

```
docker pull nginx
```



###### 修改镜像tag

```
docker tag nginx 9.134.242.16:5000/nginx
```



###### push镜像

```
docker push 9.134.242.16:5000/nginx
```

这样本地仓库中就会存在一个nginx的镜像了



##### 添加登陆账号

* 添加账号信息

  ```
  mkdir auth
   docker run \
    --entrypoint htpasswd \
    httpd:2 -Bbn admin 123456 > auth/htpasswd
  ```



* 重新启动registry

  ```
  docker run -d -p 5000:5000 \
  --restart=always \
  --name=docker-registry \
  -v /var/local/auth:/auth \
  -v /var/local/docker-registry:/var/lib/registry \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry
  ```



* 重新启动registry-web

  添加 `config.yaml` 配置

  ```
  registry:
    # Docker registry url
    url: http://docker-registry:5000/v2
    # Docker registry fqdn
    name: localhost:5000
    # To allow image delete, should be false
    readonly: false
    auth:
      # Disable authentication
      enabled: true
  ```

  

  ```
  docker run -d -p 8080:8080 \
  --name registry-web \
  --link docker-registry \
  -v /var/local/config.yaml:/conf/config.yml:ro \
  hyper/docker-registry-web
  
  ```

  



#### 部署harbor

##### 什么是harbor

```
Docker容器应用的开发和运行离不开可靠的镜像管理，虽然Docker官方也提供了公共的镜像仓库，但是从安全和效率等方面考虑，部署我们私有环境内的Registry也是非常必要的。Harbor是由VMware公司开源的企业级的Docker Registry管理项目，它包括权限管理(RBAC)、LDAP、日志审核、管理界面、自我注册、镜像复制和中文支持等功能。
官网地址：https://github.com/goharbor/harbor
```



#####  下载harbor

```
# 下载离线安装包
wget https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-offline-installer-v1.8.0.tgz

# 解压安装包
tar xvf harbor-offline-installer-v1.8.0.tgz

```



也可以直接到 [git](https://github.com/goharbor/harbor/releases) 上面下载安装包







`待续`
