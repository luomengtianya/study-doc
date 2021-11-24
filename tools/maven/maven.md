- **安装jdk**  
 
    `maven需要jdk环境的支持，所以先要安装jdk，安装步骤可在目录下寻找。`
    
--- 
- **下载maven的包**
 
    - maven 3.0版本以上的包[下载地址](https://archive.apache.org/dist/maven/maven-3/)  
    
        `https://archive.apache.org/dist/maven/maven-3/`
        
    - 选择一个版本下载  
        ```
        直接下载上传到Linux或者获取当前下载链接，然后用服务器下载 
               
        wget https://archive.apache.org/dist/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz`
        
        ```
---     
- **到下载目录解压压缩包**
    
  `tar -zxvf apache-maven-3.6.1-bin.tar.gz ` 
  
--- 
- **添加环境变量**

  - 打开配置文件  
  
    `vi /etc/profile`
  
  - 添加环境变量 MAVEN_HOME   为实际地址
    ```
       #######MAVEN_PATH
       MAVEN_HOME="/usr/local/java/maven/apache-maven-3.6.1"
       PATH="$MAVEN_HOME/bin:$PATH"
       
       export MAVEN_HOME PATH
       
    ``` 
  
  - 刷新环境变量--必须  
  
    `source /etc/profile`
 
--- 
- **查看是否成功**
  
    `mvn -v` 
    
    出现类似如下信息则是安装成功
    
        Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
        Maven home: /usr/local/java/maven/apache-maven-3.6.1
        Java version: 1.8.0_171, vendor: Oracle Corporation, runtime: /usr/local/java/jdk1.8.0_171/jre
        Default locale: zh_CN, platform encoding: UTF-8
        OS name: "linux", version: "3.10.0-514.26.2.el7.x86_64", arch: "amd64", family: "unix"
    

---    
- **自定义仓库地址**
    
    maven安装好之后会有一个默认仓库--${user.home}/.m2/repository，若是要自定义仓库的位置，可以修改配置文件
    
    配置文件: MAVEN_HOME/bin/conf/settings.xml
    
    配置原文  
    
        <!-- localRepository
           | The path to the local repository maven will use to store artifacts.
           |
           | Default: ${user.home}/.m2/repository
          <localRepository>/path/to/local/repo</localRepository>
          -->
    
    删除注释 `<!-- -->` 然后在 `<localRepository>` 标签内填写自定义的路径 
    
        <localRepository>/usr/local/java/maven/repository</localRepository>
    
    **此路径必须存在**
    

--- 
- **添加国内源镜像**   

    有些jar包下载可能比较慢或者直接下载不下来，這种情况下，添加一个淘宝的镜像可能就会有很大的改善。
    
    修改 `settings.xml` 文件
    
    配置原文  
        
          <mirrors>
            <!-- mirror
             | Specifies a repository mirror site to use instead of a given repository. The repository that
             | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
             | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
             |
            <mirror>
              <id>mirrorId</id>
              <mirrorOf>repositoryId</mirrorOf>
              <name>Human Readable Name for this Mirror.</name>
              <url>http://my.repository.com/repo/path</url>
            </mirror>
             -->
          </mirrors>

    在 `<mirrors>` 标签中添加 
    
         <mirror>
             <id>alimaven</id>
             <name>aliyun maven</name>
             <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
             <mirrorOf>central</mirrorOf>        
         </mirror>  
      
    当然还可以添加其它的镜像，添加方式一致。    
           


---
- **maven基础命令**
```
maven 命令的格式为 mvn [plugin-name]:[goal-name]，可以接受的参数如下，

-D 指定参数，如 -Dmaven.test.skip=true 跳过单元测试；

-P 指定 Profile 配置，可以用于区分环境；

-e 显示maven运行出错的信息；

-o 离线执行命令,即不去远程仓库更新包；

-X 显示maven允许的debug信息；

-U 强制去远程更新snapshot的插件或依赖，默认每天只更新一次。
```


* 常用maven命令
```
创建maven项目：mvn archetype:create
指定 group： -DgroupId=packageName

指定 artifact：-DartifactId=projectName

创建web项目：-DarchetypeArtifactId=maven-archetype-webapp

创建maven项目：mvn archetype:generate
验证项目是否正确：mvn validate
maven 打包：mvn package
只打jar包：mvn jar:jar
生成源码jar包：mvn source:jar
产生应用需要的任何额外的源代码：mvn generate-sources
编译源代码： mvn compile
编译测试代码：mvn test-compile
运行测试：mvn test
运行检查：mvn verify
清理maven项目：mvn clean
生成eclipse项目：mvn eclipse:eclipse
清理eclipse配置：mvn eclipse:clean
生成idea项目：mvn idea:idea
安装项目到本地仓库：mvn install
发布项目到远程仓库：mvn:deploy
在集成测试可以运行的环境中处理和发布包：mvn integration-test
显示maven依赖树：mvn dependency:tree
显示maven依赖列表：mvn dependency:list
下载依赖包的源码：mvn dependency:sources
安装本地jar到本地仓库：mvn install:install-file -DgroupId=packageName -DartifactId=projectName -Dversion=version -Dpackaging=jar -Dfile=path
```

* web项目相关命令
```
启动tomcat：mvn tomcat:run
启动jetty：mvn jetty:run
运行打包部署：mvn tomcat:deploy
撤销部署：mvn tomcat:undeploy
启动web应用：mvn tomcat:start
停止web应用：mvn tomcat:stop
重新部署：mvn tomcat:redeploy
部署展开的war文件：mvn war:exploded tomcat:exploded
```








 
 
 
 
 