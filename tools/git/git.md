

#### 问题

##### git add 无反应
```text
可能是文件锁定了，删除.git文件夹下的index文件，再次执行git add 即可
```


#### 把本地代码上传到git仓库
* 在git上创建一个仓库

* 在本地代码内初始化git管理文件
```shell script
git init
```

* 关联git仓库
```shell script
git remote add origin git@github.com:luomengtianya/ios-test.git
```

* 本地新建一个master分支
```shell script
git checkout -B master
```

* 添加文件信息
```shell script
git add .
git commit -m "init"
```

* 为此分支创建跟踪信息(本地分支信息关联远程分支信息)
```shell script
git branch --set-upstream-to=origin/master(远程分支) master(本地分支)
```

* 强制提交，覆盖远程(更新合并代码失败)
```shell script
 git push -f
```



