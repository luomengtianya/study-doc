#### docker

#### 打包镜像-GO

#### [dockerfile-指令](https://www.runoob.com/docker/docker-dockerfile.html)

* FROM

  ```
  定制的镜像都是基于 FROM 的镜像; 如果后面接的是 nginx，后续的操作都是基于 nginx
  ```



   * RUN

     ```
     用于执行后面跟着的命令行命令, 有shell和exec两种格式
     shell:
     	RUN <命令行命令>  # <命令行命令> 等同于，在终端操作的 shell 命令
     	
     exec:
     	RUN ["可执行文件", "参数1", "参数2"] # 如--RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
     	
     注意: Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。可以使用 && 合并多个RUN指令，这样只会创建一层镜像
     	
     ```

     

   * COPY

     ```
     复制指令，从上下文目录中复制文件或者目录到容器里指定路径
     格式：
     	COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
     	COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
     	
     例如：
     	COPY hom* /mydir/
     	COPY hom?.txt /mydir/
     ```

     

   * ADD

     ```
     ADD 指令和 COPY 的使用格类似（同样需求下，官方推荐使用 COPY）
     
     ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>。
     ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定
     ```

     

   * CMD

     ```
     类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:
       CMD 在docker run 时运行。
       RUN 是在 docker build
       
     作用：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。
     
     注意：如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效  
     
     格式：
     	CMD <shell 命令> 
       CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
       CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
     ```

     

   * ENTRYPOINT

     ```
     类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。
     
     但是, 如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 CMD 指令指定的程序。
     
     优点：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。
     
     注意：如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。
     
     格式：
     	ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
     
     可以搭配 CMD 命令使用：一般是变参才会使用 CMD
     
     如：
     	FROM nginx
       ENTRYPOINT ["nginx", "-c"] # 定参
       CMD ["/etc/nginx/nginx.conf"] # 变参 
       
     docker run  nginx:test 最终执行命令为 nginx -c /etc/nginx/nginx.conf
     
     docker run  nginx:test -c /etc/nginx/new.conf 最终执行命令为 nginx -c /etc/nginx/new.conf
     ```

     

   * ENV

     ```
     设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量
     
     格式:
     	ENV <key> <value>
     	ENV <key1>=<value1> <key2>=<value2>...
     
     例如:
     	ENV NODE_VERSION 7.2.0
     
     	RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
       && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
     ```

     

   * ARG

     ```
     构建参数，与 ENV 作用一致。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。
     
     构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖
     
     ARG <参数名>[=<默认值>]
     ```

     

   * VOLUME

     ```
     定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷
     
     作用:
     	避免重要的数据，因容器重启而丢失，这是非常致命的。
     	避免容器不断变大
     	
     格式:
     	VOLUME ["<路径1>", "<路径2>"...]
     	VOLUME <路径>
     
     在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点
     ```

     

   * EXPOSE

     ```
     仅仅只是声明端口
     
     作用:
     	帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
     	在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口
     	
     格式:
     	EXPOSE <端口1> [<端口2>...]
     ```

     

   * WORKDIR

     ```
     指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。
     
     docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在
     
     格式:
      	WORKDIR <工作目录路径>
     ```

     

   * USER

     ```
     用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）
     
     格式:
     	USER <用户名>[:<用户组>]
     ```

     

   * HEALTHCHECK

     ```
     用于指定某个程序或者指令来监控 docker 容器服务的运行状态
     
     格式:
     	HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
     	HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令
     
     	HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。
     ```

     

   * ONBUILD

     ```
     用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这时执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令
     
     格式:
     	ONBUILD <其它指令>
     ```

     

   * LABEL

     ```
     LABEL 指令用来给镜像添加一些元数据（metadata），以键值对的形式
     
     格式:
     	LABEL <key>=<value> <key>=<value> <key>=<value> ...
     
     例如添加镜像作者: 
     	LABEL org.opencontainers.image.authors="runoob"
     ```

     

##### 编译GO项目

* 默认项目为 `blog`

* 因为Docker内核依赖于Linux开发的，所以要把GO的编译环境设置成linux，不然镜像启动的时候会报`exec format error`错误

  ```
  CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
  ```

  其中`CGO_ENABLED=0`是关闭c/c++的交叉编译；`GOOS=linux`是设置linux环境；`GOARCH=amd64`是设置64位系统环境

* GOOS的可选项

  ```
  linux   Linux系统
  darwin  MacOSxitong
  windows Windows系统
  freebsd Unix系统
  ```



##### 创建Dockerfile并填写内容

默认在项目根目录

```shell
tee <<EOF > Dockerfile
FROM alpine:latest

RUN mkdir -p /data/service/blog/bin
COPY blog /data/service/blog/bin
WORKDIR  /data/service/blog

EXPOSE 8888

CMD ./bin/blog
EOF
```

此处使用`alpine`作为基础镜像，是因为这个镜像包非常小，若果以`golang`作为基础镜像，打出来的包会有几百兆。



##### 打包镜像

```shell
docker build -f Dockerfile -t blog:v4 .
```

其中，最后的 *.* 是Dockerfile的路径，不能省略



这样打包，每次都需要修改tag，不方便，我们可以创建一个打包脚本`build.sh`，用变量作为tag的数据进行打包

```shell
#!/bin/bash

Version=`date +%Y%m%d.%H%M%S`.`git rev-parse HEAD|cut -c1-6`.dev

CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build

docker build -f Dockerfile -t blog:v${Version} .
```

再赋予执行权限 `chmod +x build.sh` 后执行

```shell
./build.sh
```

使用 `docker images`查看，会发现我们打包的镜像已经在docker中存在了

```
REPOSITORY       TAG                           IMAGE ID       CREATED          SIZE
blog             v20211118.161627.b79686.dev   e2334212fdf9   37 minutes ago   22.3MB
blog             v20211118.162330.b79686.dev   e2334212fdf9   37 minutes ago   22.3MB

```



当我们多次以同一个编译文件执行打包镜像的时候，会出现多个镜像imageid相同的情况，以imageid删除镜像就会报错，这时候就需要以`docker rmi REPOSITORY:TAG`来删除镜像



为了避免镜像出现相同imageid的情况，我们可以在编译的时候，修改下文件版本数据

```
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
修改为
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags "-X main.Version=${Version}"
```

这样每次打包的镜像imageid就不会相同了



##### 提交镜像到仓库

如果我们想提交打包的镜像到远程仓库中，可以先登录到远程仓库，再提交上去

```shell
docker login ${Repostry} --username ${username} --password ${password}
docker push ${ImageName}
```









