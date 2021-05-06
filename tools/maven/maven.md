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
           


 









 
 
 
 
 