#### OpenSSL简介

OpenSSL是一种加密工具套件，可实现安全套接字层（SSL v2 / v3）和传输层安全性（TLS v1）网络协议以及它们所需的相关加密标准。

openssl命令行工具用于从shell程序使用OpenSSL加密库的各种加密功能。 它可以用于：

- 创建和管理私钥，公钥和参数
- 公钥加密操作
- 创建X.509证书，CSR和CRL
- 消息摘要的计算
- 使用密码进行加密和解密
- SSL / TLS客户端和服务器测试
- 处理S / MIME签名或加密的邮件
- 时间戳记请求，生成和验证



#### openssl配置文件

centos的配置文件 `/etc/pki/tls/openssl.cnf`

```
[ CA_default ]
dir         = /etc/pki/CA       # Where everything is kept (CA有关的文件存放位置）
certs       = $dir/certs        # Where the issued certs are kept（签发的证书位置）
crl_dir     = $dir/crl          # Where the issued crl are kept（吊销证书存放位置）
database    = $dir/index.txt    # database index file.（生成证书索引数据库文件）
#unique_subject = no            # Set to 'no' to allow creation of
                                # several ctificates with same subject.
new_certs_dir   = $dir/newcerts     # default place for new certs.
certificate = $dir/cacert.pem   # The CA certificate（CA公钥位置）
serial      = $dir/serial       # The current serial number（指定颁发证书的序列号）
crlnumber   = $dir/crlnumber    # the current crl number
                    # must be commented out to leave a V1 CRL
crl     = $dir/crl.pem      # The current CRL
private_key = $dir/private/cakey.pem# The private key （CA私钥）
RANDFILE    = $dir/private/.rand    # private random number file
x509_extensions = usr_cert      # The extentions to add to the cert
policy      = policy_match
# For the CA policy  策略
[ policy_match ]
countryName     = match
stateOrProvinceName = match
organizationName    = match
organizationalUnitName  = optional
commonName      = supplied
emailAddress        = optional
# For the 'anything' policy
# At this point in time, you must list all acceptable 'object'
# types.
[ policy_anything ]
countryName     = optional
stateOrProvinceName = optional
localityName        = optional
organizationName    = optional
organizationalUnitName  = optional
commonName      = supplied
emailAddress    = optiona
```



#### 三种策略

* match（匹配）：要求申请填写的信息跟CA设置信息必须一致

* optional（可选）：可有可无，跟CA设置信息可不一致
* supplied（提供）：必须填写这项申请信息



#### 创建私有CA和申请、颁发证书文件

##### 创建所需的文件

```csharp
[root@centos7 ~]#cd /etc/pki/CA
[root@centos7 CA]#ls
certs  crl  newcerts  private
[root@centos7 CA]#touch index.txt   生成证书索引数据库文件(默认没有）
[root@centos7 CA]#echo 01 > serial  指定第一个颁发证书的序列号（默认也没有 这里01两位采用的是十六进制）
[root@centos7 CA]#ls
certs  crl  index.txt  newcerts  private  serial
```



##### 生成私钥文件

```
(umask 066;openssl genrsa -out private/cakey.pem 2048)
```



```
[root@centos7 CA]#(umask 066;openssl genrsa -out private/cakey.pem 2048)
Generating RSA private key, 2048 bit long modulus
................................+++
.........+++
e is 65537 (0x10001)
[root@centos7 CA]#ll private/
total 4
-rw------- 1 root root 1679 Jul 16 19:21 cakey.pem
```

```
umask 可以定义新建文件和新建文件夹的权限

umask	File     Permissions	     Directory Permissions
066	           rw——-	           rwx–x–x
067	           rw——-	           rwx–x—
070	           rw—-rw-	         rwx—rwx
071	           rw—-rw-	         rwx—rw-
```



##### 生成签名

```
openssl req -new -x509 -key private/cakey.pem -days 3650 -out cacert.pem
```



```
[root@CentOS7 CA]# openssl req -new -x509 -key private/cakey.pem -days 3650 -out cacert.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:hunan
Locality Name (eg, city) [Default City]:changsha
Organization Name (eg, company) [Default Company Ltd]:abc
Organizational Unit Name (eg, section) []:IT  
Common Name (eg, your name or your server's hostname) []:peterppan
Email Address []:peterppan@abc.com

选项说明：
-new：生成新证书签署请求
-x509：专用于CA生成自签证书
-key：生成请求时用到的私钥文件
-days n：证书的有效期限
-out /PATH/TO/SOMECERTFILE: 证书的保存路径
```



##### 颁发证书

###### 生成私钥

```
(umask 066;openssl genrsa -out /data/test.key 2048)
```



```
[root@CentOS7 CA]# (umask 066;openssl genrsa -out /data/test.key 2048)
Generating RSA private key, 2048 bit long modulus
..................................................+++
...............................+++
e is 65537 (0x10001)
```



###### 生成证书申请文件

```
openssl req -new -key /data/test.key -out /data/test.csr
```



```
[root@CentOS7 CA]# openssl req -new -key /data/test.key -out /data/test.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:hunan
Locality Name (eg, city) [Default City]:changsha
Organization Name (eg, company) [Default Company Ltd]:abc
Organizational Unit Name (eg, section) []:IT
Common Name (eg, your name or your server's hostname) []:peterpan / IP
Email Address []:peterpan@abc.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```



###### 将证书申请文件传输给CA

两台不同的主机可以使用scp命令传输



###### CA签署证书，并将证书颁发给请求者

```
openssl ca -in /data/test.csr -out certs/test.crt -days 100
```



```
[root@VM-242-16-centos /etc/pki/CA]# openssl ca -in /data/test.csr -out certs/test.crt -days 100
Using configuration from /etc/pki/tls/openssl.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Nov 21 11:25:49 2021 GMT
            Not After : Mar  1 11:25:49 2022 GMT
        Subject:
            countryName               = CN
            stateOrProvinceName       = HN
            organizationName          = tencent-test
            organizationalUnitName    = tencent-test
            commonName                = peterppan
            emailAddress              = peterppan@tencent.com
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                A3:9F:2C:15:0C:42:58:A9:9B:7E:F6:40:53:65:17:2F:95:1B:D2:97
            X509v3 Authority Key Identifier: 
                keyid:3A:12:3E:CB:E5:74:A9:AA:76:12:13:38:12:CC:18:14:07:F1:22:E1

Certificate is to be certified until Mar  1 11:25:49 2022 GMT (100 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated

注意：默认要求 countryName（国家），stateOrProvinceName（省），organizationName（公司）三项必须和CA一致,否则或报错

[root@VM-242-16-centos /etc/pki/CA]# openssl ca -in /data/test.csr -out certs/test.crt -days 100
Using configuration from /etc/pki/tls/openssl.cnf
Check that the request matches the signature
Signature ok
The organizationName field needed to be the same in the
CA certificate (tencent-test) and the request (aa)
```



###### 查看证书中的信息

```
openssl x509 -in certs/test.crt -noout -text
```

```
[root@VM-242-16-centos /etc/pki/CA]# openssl x509 -in certs/test.crt -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 1 (0x1)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, ST=hunan, L=changsha, O=tencent-test, OU=aa, CN=peterppan/emailAddress=peterppan@tencent.com
        Validity
            Not Before: Nov 21 11:25:49 2021 GMT
            Not After : Mar  1 11:25:49 2022 GMT
        Subject: C=CN, ST=HN, O=tencent-test, OU=tencent-test, CN=peterppan/emailAddress=peterppan@tencent.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:af:cc:89:d6:71:a8:24:ec:a2:30:53:3a:4c:8b:
                    b9:30:92:2e:dc:00:02:d9:45:85:1f:05:aa:16:a8:
                    5c:02:e4:63:16:cd:07:ae:e1:c9:af:7b:6d:a8:87:
                    09:9f:17:ed:64:8f:20:62:8f:32:de:2a:1d:cc:c5:
                    38:e3:72:b0:9f:13:5a:8d:2b:95:36:49:fb:98:70:
                    dc:5a:4e:48:19:ad:bd:b5:0d:61:49:ad:fc:4d:25:
                    d8:fe:ce:4c:f0:45:1d:75:61:ec:87:d1:5a:e8:d3:
                    41:97:a4:30:bb:53:60:7d:e1:ca:49:87:ac:0f:39:
                    93:20:7b:3a:b2:40:a8:6e:dc:10:8a:27:27:7a:c6:
                    39:00:d7:e7:9c:71:22:dd:58:0c:9e:66:05:7a:39:
                    38:5f:c2:51:88:9f:8d:e9:d2:e9:0a:bc:4b:23:26:
                    fb:c6:c2:61:c9:e8:28:e4:fa:ca:9e:4e:87:73:8b:
                    b8:a3:c7:4e:43:75:f3:5d:fd:8f:9a:75:73:c1:6f:
                    10:6d:90:13:3d:45:23:b6:84:48:b2:2d:cc:cf:14:
                    cb:09:13:81:b2:ce:78:eb:1f:d0:fb:1b:c0:21:f1:
                    7b:6a:ef:24:66:fb:a8:ac:90:dc:d6:66:58:1b:0f:
                    0f:11:e3:12:4c:47:03:d2:a6:4f:d6:e1:35:e7:61:
                    2a:a3
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                A3:9F:2C:15:0C:42:58:A9:9B:7E:F6:40:53:65:17:2F:95:1B:D2:97
            X509v3 Authority Key Identifier: 
                keyid:3A:12:3E:CB:E5:74:A9:AA:76:12:13:38:12:CC:18:14:07:F1:22:E1

    Signature Algorithm: sha256WithRSAEncryption
         60:e4:02:16:02:53:7e:44:ce:be:d3:cf:8d:c9:8e:52:09:d5:
         7f:05:9c:1c:be:71:5f:5b:a9:b2:8d:e4:45:0b:78:7d:f1:07:
         43:e4:04:13:d0:61:63:f8:bb:83:65:51:f4:50:7a:45:27:4e:
         2f:30:1c:e9:53:a9:37:5c:af:6e:d1:ff:bd:e8:1b:9d:38:9a:
         fe:8e:62:e3:16:30:e1:a4:8c:12:a6:3c:51:eb:a4:30:5b:82:
         2a:03:3d:e4:83:1f:42:9d:fd:0b:ef:c9:61:ef:fd:12:e7:8d:
         b1:18:ac:c3:5d:31:ab:a6:d3:67:7c:ae:2c:02:72:29:aa:e5:
         b5:4a:23:56:f8:a1:37:ee:cb:11:e8:61:6a:ae:3b:b4:7d:db:
         ab:58:91:44:51:1c:c7:d3:dd:b1:23:9e:fd:92:bf:8a:9f:a0:
         74:c9:84:0d:0a:63:22:ab:a0:43:5d:ff:10:d5:b9:5c:5e:19:
         f6:26:e5:7c:ff:7c:c7:f0:2e:3f:58:e9:32:23:84:89:f6:32:
         3d:90:c3:68:a5:68:05:b5:74:33:05:72:3f:c7:40:68:55:65:
         b8:de:7f:96:cd:a5:ac:04:8e:22:a6:56:ea:a3:7e:39:d4:40:
         c6:8f:6a:33:b4:4f:bc:fc:7a:2c:29:d5:bb:fe:bb:ef:cf:0d:
         f0:05:c6:51
```



###### 查看指定编号的证书状态

```
openssl ca -status 01
```



###### 聚合命令

```
cd /etc/pki/CA
-- 生成私钥
(umask 066;openssl genrsa -out private/cakey.pem 2048)
-- 生成私钥签名
openssl req -new -x509 -key private/cakey.pem -subj "/C=CN/ST=HN/O=tencent-test/OU=tencent-test/CN=9.134.242.16" -days 3650 -out cacert.pem
-- 生成公钥
(umask 066;openssl genrsa -out certs/test.key 2048)
-- 生成证书申请文件
openssl req -new -key certs/test.key -subj "/C=CN/ST=HN/O=tencent-test/OU=tencent-test2/CN=9.134.242.16" -out certs/test.csr
-- 生成证书
openssl ca -in certs/test.csr -out certs/test.crt -days 365

注意： 
	两个 -subj中的OU配置需要不一样，不然生成证书会报错
	failed to update database
	TXT_DB error number 2
	
	默认要求 countryName（国家），stateOrProvinceName（省），organizationName（公司）三项必须和CA一致
```



#### 吊销证书

##### 在客户端获取要吊销的证书的serial

```
[root@CentOS7 CA]# openssl x509 -in certs/test.crt -noout -serial -subject
serial=01
subject= /C=CN/ST=hunan/O=tencent-test/OU=tencent-test/CN=peterppan/emailAddress=peterppan@tencent.com
```



##### 在CA上，根据客户提交的serial与subject信息，对比检验是否与index.txt文件中的信息一致

```
[root@VM-242-16-centos /etc/pki/CA]# cat index.txt
V       220301112549Z           01      unknown /C=CN/ST=hunan/O=tencent-test/OU=tencent-test/CN=peterppan/emailAddress=peterppan@tencent.com
```



##### 吊销证书

```
[root@CentOS7 CA]# openssl ca -revoke newcerts/01.pem 
Using configuration from /etc/pki/tls/openssl.cnf
Revoking Certificate 01.
Data Base Updated
```



##### 指定第一个吊销证书的编号

```
[root@CentOS7 CA]# echo 01 > crlnumber

注意：第一次更新证书吊销列表前才需要执行
```



##### 更新证书吊销列表

```
[root@CentOS7 CA]# openssl ca -gencrl -out crl.pem
Using configuration from /etc/pki/tls/openssl.cnf
```



##### 查看crl文件

```
[root@CentOS7 CA]# openssl crl -in crl.pem -noout -text
```





#### 参考

[简书](https://www.jianshu.com/p/15b3d94c24f5)

[简书2](https://www.jianshu.com/p/15b3d94c24f5)