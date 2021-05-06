`JDK是很多环境运行的基础，所以在搭建其它环境之前需要先搭建好。搭建方式有直接命令行安装和手动安装。`

---
**自动安装**  

 - centerOS版本  
    
    `yum install java-1.8.0-openjdk* -y`

---
**手动安装**
- 下载jdk安装包，此处下载jdk8版本的
    - 下载地址：https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
    
    
- 上传到服务器
    - 把下载的文件上传到Linux服务器，tar -zxvf `source-name.tar` 解压文件

- 编辑环境变量
    - 打开环境变量文件  
     `vi /etc/profile`
     
    - 添加环境变量,JAVA_HOME后面的地址为实际解压的jdk地址  
    
         `JAVA_HOME="/usr/local/java/jdk1.8.0_171/"`
         
         `CLASSPATH=".:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH"`
         
         `PATH=".:$JAVA_HOME/bin:$PATH"`
         
         `export JAVA_HOME CLASSPATH PATH`
    
    - 刷新环境变量  
      `刚刚配置的环境变量不会立即生效，需要执行 source /etc/profile 才行`
    
- 验证  

    `执行两个命令，能够正常输出数据`
    - 执行 java 出现以下信息  
    
    
       用法: java [-options] class [args...] (执行类) 或  java [-options] -jar jarfile [args...]
                  (执行 jar 文件)
       其中选项包括:
           -d32	  使用 32 位数据模型 (如果可用)
           -d64	  使用 64 位数据模型 (如果可用)
           -server	  选择 "server" VM
                         默认 VM 是 server.
       
           -cp <目录和 zip/jar 文件的类搜索路径>
           -classpath <目录和 zip/jar 文件的类搜索路径>
                         用 : 分隔的目录, JAR 档案
                         和 ZIP 档案列表, 用于搜索类文件。
           -D<名称>=<值>
                         设置系统属性
           -verbose:[class|gc|jni]
                         启用详细输出
           -version      输出产品版本并退出
           -version:<值>
                         警告: 此功能已过时, 将在
                         未来发行版中删除。
                         需要指定的版本才能运行
           -showversion  输出产品版本并继续
           -jre-restrict-search | -no-jre-restrict-search
                         警告: 此功能已过时, 将在
                         未来发行版中删除。
                         在版本搜索中包括/排除用户专用 JRE
           -? -help      输出此帮助消息
           -X            输出非标准选项的帮助
           -ea[:<packagename>...|:<classname>]
           -enableassertions[:<packagename>...|:<classname>]
                         按指定的粒度启用断言
           -da[:<packagename>...|:<classname>]
           -disableassertions[:<packagename>...|:<classname>]
                         禁用具有指定粒度的断言
           -esa | -enablesystemassertions
                         启用系统断言
           -dsa | -disablesystemassertions
                         禁用系统断言
           -agentlib:<libname>[=<选项>]
                         加载本机代理库 <libname>, 例如 -agentlib:hprof
                         另请参阅 -agentlib:jdwp=help 和 -agentlib:hprof=help
           -agentpath:<pathname>[=<选项>]
                         按完整路径名加载本机代理库
           -javaagent:<jarpath>[=<选项>]
                         加载 Java 编程语言代理, 请参阅 java.lang.instrument
           -splash:<imagepath>
                         使用指定的图像显示启动屏幕
    
    
  - 执行 javac  出现以下信息
  
  ```
  用法: javac <options> <source files>
     其中, 可能的选项包括:
       -g                         生成所有调试信息
       -g:none                    不生成任何调试信息
       -g:{lines,vars,source}     只生成某些调试信息
       -nowarn                    不生成任何警告
       -verbose                   输出有关编译器正在执行的操作的消息
       -deprecation               输出使用已过时的 API 的源位置
       -classpath <路径>            指定查找用户类文件和注释处理程序的位置
       -cp <路径>                   指定查找用户类文件和注释处理程序的位置
       -sourcepath <路径>           指定查找输入源文件的位置
       -bootclasspath <路径>        覆盖引导类文件的位置
       -extdirs <目录>              覆盖所安装扩展的位置
       -endorseddirs <目录>         覆盖签名的标准路径的位置
       -proc:{none,only}          控制是否执行注释处理和/或编译。
       -processor <class1>[,<class2>,<class3>...] 要运行的注释处理程序的名称; 绕过默认的搜索进程
       -processorpath <路径>        指定查找注释处理程序的位置
       -parameters                生成元数据以用于方法参数的反射
       -d <目录>                    指定放置生成的类文件的位置
       -s <目录>                    指定放置生成的源文件的位置
       -h <目录>                    指定放置生成的本机标头文件的位置
       -implicit:{none,class}     指定是否为隐式引用文件生成类文件
       -encoding <编码>             指定源文件使用的字符编码
       -source <发行版>              提供与指定发行版的源兼容性
       -target <发行版>              生成特定 VM 版本的类文件
       -profile <配置文件>            请确保使用的 API 在指定的配置文件中可用
       -version                   版本信息
       -help                      输出标准选项的提要
       -A关键字[=值]                  传递给注释处理程序的选项
       -X                         输出非标准选项的提要
       -J<标记>                     直接将 <标记> 传递给运行时系统
       -Werror                    出现警告时终止编译
       @<文件名>                     从文件读取选项和文件名
       
  ```

