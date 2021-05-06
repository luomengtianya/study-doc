###jenkins的安装和使用  

---

#### jenkins的安装
    
---
- jdk的安装  

    `安装目录下教程`
 
 
- jenkins的下载  
    
    [下载地址](https://jenkins.io/download/)
    
    选择 `Generic Java Package(.war)`   
    
    下载后上传到服务器或者复制链接直接在服务器上下载
    
    `wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war`

---    
-  启动 `jenkins.war`
    
    下载的通用war包是可执行文件，直接在文件目录下命令启动就可以了
    
    ```java -jar jenkins.war --httpPort=8080```
     
    文件安装路径默认为： ```/root/.jenkins/plugins```
    
    
    
    Jenkins储存所有的数据文件在这个目录下. 你可以通过以下几种方式更改：
        使用你Web容器的管理工具设置JENKINS_HOME环境参数.
        在启动Web容器之前设置JENKINS_HOME环境变量.
        (不推荐)更改Jenkins.war(或者在展开的Web容器)内的web.xml配置文件.
    这个值在Jenkins运行时是不能更改的. 
      

启动后控制台会有一串密钥输出
    
   
    *************************************************************
    *************************************************************
    *************************************************************
    
    Jenkins initial setup is required. An admin user has been created and a password generated.
    Please use the following password to proceed to installation:
    
    06c7add0dde0485997a6fe324f948114
    
    This may also be found at: /root/.jenkins/secrets/initialAdminPassword
    
    *************************************************************
    *************************************************************
    *************************************************************

      
    
记住这一串密钥 `06c7add0dde0485997a6fe324f948114` 
                                                           
- 打开浏览器，输入：http://localhost:8080 出现以下界面

 ![项目地址：https://github.com/luomengtianya/environments-duilding](/jenkins/jenkins-01.png) 
 

 输入记录的密钥或者按照界面上的路径找到文件后复制密钥到 `输入框` 中，点击确定。

- 下一个界面选择 `安装推荐的插件`，会自动安装基础插件

 ![项目地址：https://github.com/luomengtianya/environments-duilding](/jenkins/jenkins-02.png) 
 

若是不能自动安装，则复制[plugin文件](https://github.com/luomengtianya/plugins/blob/master/plugins.zip)到 `/root/.jenkins/plugins` 目录下

- 输入注册信息
 ![项目地址：https://github.com/luomengtianya/environments-duilding](/jenkins/jenkins-03.png)

- 按照默认的配置实例和进入jenkins即可
    

---
     
####jenkins的使用

---

   
    
- 启动错误: jenkins启动时不能下载插件，现实connect time out，需要修改验证网络的地址 
    
    ```
     默认情况下Jenkins通过google来检查网络连接，但是在大天朝
     google被一些众所周知的原因和谐了，因此要想再墙内正常使用jenkins，
     可以通过修改配置文件来改变这个检查URL，修改方法为，
     将/root/.jenkins/updates/default.json文件中的"connectionCheckUrl":"http://www.google.com/"
     更改为"connectionCheckUrl":"http://www.baidu.com/"即可，更改完成后需要重启生效```    

- 清华插件镜像: 
     `https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json`

    
- 插件
    - ssh插件  
     `Publish Over SSH`
     
    - 邮件配送  
    `Editable Email Notification` 
       
    - 用户权限  
    `Role-based Authorization Strategy`   
    
    - 查看配置  
     ``
    - 批量修改配置  
     `Configuration Slicing Plugin`
    - findbugs  
     `FindBugs Plugin，Report Info Plugin，Static Analysis Collector Plug-in`
    
   

