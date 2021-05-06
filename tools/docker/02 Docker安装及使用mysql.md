### 安装Mysql

#### 安装
* 安装
  * docker search mysql 查看版本后 docker pull mysql:latest 安装最新版本
  * 到[mysql镜像](https://hub.docker.com/_/mysql?tab=tags&page=1&ordering=last_updated)查看镜像版本，然后docker pull mysql:5.6.51安装指定版本

* 启动
  * $ docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.6.51
```
-p 3306:3306 ：映射容器服务的 3306 端口到宿主机的 3306 端口，外部主机可以直接通过 宿主机ip:3306 访问到 MySQL 的服务。
MYSQL_ROOT_PASSWORD=123456：设置 MySQL 服务 root 用户的密码。
:5.6.51 指定启动版本，不然会自动下载最新版本然后启动
```


#### 使用Mysql
* 导入sql文件数据到mysql中
  * 将用户目录下的sql文件复制到容器中
```ssh
sudo docker cp ~/{{sqlname}}.sql {{mysql server name}}:{{sqlname}}.sql
```

  * 进入容器中
```ssh
docker exec -it {{mysql server name}} /bin/bash
```
 
  * 查看是否成功导入文件
```ssh
ls -l {{sqlname}}.sql
```

  * 连接mysql服务
```ssh
mysql -u root -p
```

  * 执行sql文件
```ssh
source {{sqlname}}.sql
```

--------------

### 安装redis







#### 使用redis
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
```ssh
sudo docker cp redis.conf redis:/etc/redis.conf 
```

* 授权
```ssh
chmod 755 redis.conf
```

* 按照redis.conf启动redis

```ssh
docker run -p 6379:6379 --name redis -v /var/local/docker/redis.conf:/etc/redis/redis.conf -v /var/local/docker/data:/data -d redis redis-server /etc/redis/redis.conf

-----
-p 6379:6379 端口映射：前表示主机部分，：后表示容器部分。

--name myredis  指定该容器名称，查看和进行操作都比较方便。

-v 挂载目录，规则与端口映射相同。

为什么需要挂载目录：个人认为docker是个沙箱隔离级别的容器，这个是它的特点及安全机制，不能随便访问外部（主机）资源目录，所以需要这个挂载目录机制。

-d redis 表示后台启动redis

redis-server /etc/redis/redis.conf  以配置文件启动redis，加载容器内的conf文件，最终找到的是挂载的目录/usr/local/docker/redis.conf

```





