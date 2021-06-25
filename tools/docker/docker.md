[TOC]

### 1、安装docker
####  docker文档
* [Docker 安装文档](https://docs.docker.com/engine/install/centos/)
* [Docker 菜鸟教程](https://www.runoob.com/docker/docker-image-usage.html)
* [Docker 官方主页](https://www.docker.com)
* [Docker 官方博客](https://blog.docker.com/)
* [Docker 官方文档](https://docs.docker.com/)
* [Docker Store](https://store.docker.com)
* [Docker Cloud](https://cloud.docker.com)
* [Docker Hub](https://hub.docker.com)
* [Docker 的源代码仓库](https://github.com/moby/moby)
* [Docker 发布版本历史](https://docs.docker.com/release-notes/)
* [Docker 常见问题](https://docs.docker.com/engine/faq/)
* [Docker 远端应用 API](https://docs.docker.com/develop/sdk/)

####  卸载docker
官网方式，腾讯云服务器亲测卸载失败，使用下面的指定删除方式成功
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

####  安装docker
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

####  [腾讯云服务器安装docker](https://cloud.tencent.com/document/product/213/46000?from=information.detail.%E8%85%BE%E8%AE%AF%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%89%E8%A3%85docker)
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


---
### 2、使用docker
####  命令大全
   * 容器生命周期管理
   
```shell script
    run
    start/stop/restart
    kill
    rm
    pause/unpause
    create
    exec
```   
      
   * 容器操作
```text   
    ps
    inspect
    top
    attach
    events
    logs
    wait
    export
    port
```           
   * 容器rootfs命令
```text       
    commit
    cp
    diff
```           
   * 镜像仓库
```text   
    login
    pull
    push
    search
```           
   * 本地镜像管理
```text   
    images
    rmi
    tag
    build
    history
    save
    load
    import
    info|version
    info
    version 
```

#### 管理docker守护进程
* 运行 Docker 守护进程
```shell script    
    systemctl start docker
```       
* 停止 Docker 守护进程
```shell script    
    systemctl stop docker
```        
* 重启 Docker 守护进程
```shell script    
    systemctl restart docker            
```       
        
 #### 管理镜像          
  
docker pull nginx 
    
* 修改标签：您可以修改镜像标签以便记忆区分
```shell script    
    docker tag docker.io/nginx:latest tencentyun/nginx:v1
```        
* 查看已有镜像
```shell script    
    docker images
```       
* 强制删除镜像
```shell script    
    docker rmi -f tencentyun/nginx:v1         
```        
* 导出镜像
```shell script
    docker export 1e560fca3906 > msyql.tar
    docker save -o collect.tar mysql redis 
 ```       
* 导入镜像
```
    docker
```
* 设置镜像标签  
```shell script
    docker tag 860c279d2fec runoob/centos:dev        
```
* 备注
```text    
    docker save 保存的是镜像（image），docker export 保存的是容器（container）
    docker load 用来载入镜像包，docker import 用来载入容器包，但两者都会恢复为镜像
    docker load 不能对载入的镜像重命名，而 docker import 可以为镜像指定新名称             
```        
####  管理容器
* 进入容器(其中，ImageId 可通过执行 docker images 命令获取)
 ```shell script   
    docker run -it ImageId /bin/bash
```       
* 退出容器：执行 exit 命令，退出当前容器
    
* 进入后台运行的容器
```shell script    
    docker exec -it 容器 ID /bin/bash
```        
* 将容器做成镜像
```shell script    
    docker commit <容器 ID 或容器名> [<仓库名>[:<标签>]]            
```        
* 进入后台运行的容器
```shell script    
    docker exec -it 容器 ID /bin/bash                        
```             


####  制作镜像
* 执行以下命令，打开 Dockerfile 文件
```shell script
    vim Dockerfile
```
* 按 i 切换至编辑模式，添加如下内容
```text
    FROM tencentyun/nginx:v2  #声明基础镜像来源。
    MAINTAINER DTSTACK #声明镜像拥有者。
    RUN mkdir /dtstact # RUN 后面接容器运行前需要执行的命令，由于 Dockerfile 文件不能超过127行，因此当命令较多时建议写到脚本中执行。
    ENTRYPOINT ping https://cloud.tencent.com/ #开机启动命令，此处最后一个命令需要是可在前台持续执行的命令，否则容器后台运行时会因为命令执行完而退出。
```
* 按 Esc，输入 :wq，保存文件并返回

* 执行以下命令，构建镜像
```shell script
    docker build -t nginxos:v1 .  #.是 Dockerfile 文件的路径，不能忽略。         
```     
* 执行以下命令，查看镜像是否创建成功
```shell script
    docker images         
```      
* 依次执行以下命令，运行容器并查看容器
```text
    docker run -d nginxos:v1         #后台运行容器。
    docker ps                        #查看当前运行中的容器。
    docker ps -a                     #查看所有容器，包括未运行中的。
    docker logs CONTAINER ID/IMAGE   #如未查看到刚才运行的容器，则用容器 ID 或者名字查看启动日志排错。         
```
* 依次执行以下命令，制作镜像
```text
    docker commit fb2844b6**** nginxweb:v2 #commit 参数后添加容器 ID 和构建新镜像的名称和版本号。
    docker images                    #列出本地（已下载的和本地创建的）镜像。          
```
* 执行以下命令，将镜像推送至远程仓库。
  默认推送到 Docker Hub。您需要先登录 Docker，为镜像绑定标签，将镜像命名为 Docker 用户名/镜像名:标签的格式，最终完成推送
```text
    docker login #执行后输入镜像仓库用户名及密码
    docker tag [镜像名]:[标签] [用户名]:[标签]
    docker push [用户名]:[标签]
```

####  options说明
```text 
    -d :分离模式: 在后台运行
    
    -i :即使没有附加也保持STDIN 打开
    
    -t :分配一个伪终端
```

####  docker国内镜像
```text
    阿里云的加速器：https://help.aliyun.com/document_detail/60750.html
    
    网易加速器：http://hub-mirror.c.163.com
    
    官方中国加速器：https://registry.docker-cn.com
    
    ustc 的镜像：https://docker.mirrors.ustc.edu.cn
    
    daocloud：https://www.daocloud.io/mirror#accelerator-doc（注册后使用）
```

---
### 3、安装及使用镜像
####  安装Mysql
* 安装
  * docker search mysql 查看版本后 docker pull mysql:latest 安装最新版本
  * 到 [mysql镜像](https://hub.docker.com/_/mysql?tab=tags&page=1&ordering=last_updated) 查看镜像版本，然后`docker pull mysql:5.6.51`安装指定版本

* 启动
  * $ docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=Pan@2021 mysql:5.6.51
```
    -p 3306:3306 ：映射容器服务的 3306 端口到宿主机的 3306 端口，外部主机可以直接通过 宿主机ip:3306 访问到 MySQL 的服务。
    MYSQL_ROOT_PASSWORD=Pan@2021：设置 MySQL 服务 root 用户的密码。
    :5.6.51 指定启动版本，不然会自动下载最新版本然后启动 \ 或者把mysql:5.6.51修改成镜像ID
```

####  使用Mysql
* 导入sql文件数据到mysql中
  * 将用户目录下的sql文件复制到容器中
```shell script
    sudo docker cp ~/{{sqlname}}.sql {{mysql server name}}:{{sqlname}}.sql
```

  * 进入容器中
```shell script
    docker exec -it {{mysql server name}} /bin/bash
```
 
  * 查看是否成功导入文件
```shell script
    ls -l {{sqlname}}.sql
```

  * 连接mysql服务
```shell script
    mysql -u root -p
```

  * 执行sql文件
```shell script
    source {{sqlname}}.sql
```

####  使用redis
* docker的redis没有redis.conf文件，可在 [redis官网](http://download.redis.io/redis-stable/redis.conf) 下载

* 修改redis.conf文件
```text
    bind 127.0.0.1 #注释掉这部分，这是限制redis只能本地访问
    
    protected-mode no #默认yes，开启保护模式，限制为本地访问
    
    daemonize no#默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败
    
    databases 16 #数据库个数（可选），我修改了这个只是查看是否生效。。
    
    dir  ./ #输入本地redis数据库存放文件夹（可选）
    
    appendonly yes #redis持久化（可选）
    
    logs

```

* 创建文件夹存放redis.conf文件
```shell script
    sudo docker cp redis.conf redis:/etc/redis.conf 
```

* 授权
```shell script
    chmod 755 redis.conf
```

* 按照redis.conf启动redis

```text
    docker run -p 6379:6379 --name redis -v /var/local/docker/redis.conf:/etc/redis/redis.conf -v /var/local/docker/data:/data -d redis redis-server /etc/redis/redis.conf
    
    -----
    -p 6379:6379 端口映射：前表示主机部分，：后表示容器部分。
    
    --name myredis  指定该容器名称，查看和进行操作都比较方便。
    
    -v 挂载目录，规则与端口映射相同。
    
    为什么需要挂载目录：个人认为docker是个沙箱隔离级别的容器，这个是它的特点及安全机制，不能随便访问外部（主机）资源目录，所以需要这个挂载目录机制。
    
    -d redis 表示后台启动redis
    
    redis-server /etc/redis/redis.conf  以配置文件启动redis，加载容器内的conf文件，最终找到的是挂载的目录/usr/local/docker/redis.conf
```











今天在对dockder容器重启后发现有两个镜像没有启动，于是使用以下命令查到docker镜像ID：
docker ps -a


使用一下命令对镜像从起：


docker restart imageid


在运行docker容器时可以加如下参数来保证每次docker服务重启后容器也自动重启：
docker run 
--restart=always
 
如果已经启动了则可以使用如下命令：

docker update --restart=always <CONTAINER ID>

