---
layout:     post                    # 使用的布局（不需要改）
title:      Apache NiFi启用SSL/TLS              # 标题 
subtitle:   Apache NiFi启用SSL/TLS-客户端证书 #副标题
date:       2018-09-11              # 时间
author:     LJ                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Apache NiFi
    - TLS
---
# <center>Apache NiFi启用SSL/TLS </center >      

| 系统 | Apache NIFI(单机) | Java Version | 
| ------ | ------ | ------ | 
| MacOS 10.13.4 | 1.7.1 | 1.8.0_181 | 

## 创建自签名证书
1. 从https://nifi.apache.org/download.html下载对应的NiFi工具包
2. 将其解压缩到一个目录。   
<code>/usr/local/Cellar/nifi-toolkit-1.7.1</code>
3. 进入该目录   
<code>/usr/local/Cellar/nifi-toolkit-1.7.1/bin</code>
4. 切换到该目录并使用tls-toolkit生成证书。    
<code>cd /usr/local/Cellar/nifi-toolkit-1.7.1/bin/</code>  
这将生成一个客户端证书和密码文件以及服务器密钥库和信任库(如果在不同的服务器上运行username和NIFI，请使用NiFi节点的主机名，以防止出现证书问题) :   
-C 指定将生成的客户端证书的DN(区分名) ;CN=Common Name 为用户名或服务器名 ; OU=Organization Unit为组织单元  
-o 指定用于输出的目录  
<code>./tls-toolkit.sh standalone -n "localhost" -C "CN=username, OU=NIFI" -o targetdocumet</code>   
目录结构如下：   
![SSL](/img/2018/09/11/SSL04.jpg)
5. 用生成的nifi.properties文件替换nifi/conf下的nifi.properties，将证书文件放到nifi-bin/conf下  
![SSL](/img/2018/09/11/SSL05.jpg)
下面为新nifi.properties文件内容：     
``` 
        # web properties #
        nifi.web.war.directory=./lib
        nifi.web.http.host=
        nifi.web.http.port=
        nifi.web.http.network.interface.default=
        nifi.web.https.host=localhost
        nifi.web.https.port=9443     #为访问端口
        nifi.web.https.network.interface.default=
        nifi.web.jetty.working.directory=./work/jetty
        nifi.web.jetty.threads=200
        nifi.web.max.header.size=16 KB
        nifi.web.proxy.context.path=
        nifi.web.proxy.host=

        # security properties #
        nifi.sensitive.props.key=
        nifi.sensitive.props.key.protected=
        nifi.sensitive.props.algorithm=PBEWITHMD5AND256BITAES-CBC-OPENSSL
        nifi.sensitive.props.provider=BC
        nifi.sensitive.props.additional.keys=

        nifi.security.keystore=./conf/keystore.jks
        nifi.security.keystoreType=jks
        nifi.security.keystorePasswd=gEZ4tIzBaYnytmLeSD0gXWzP20XVctX3Sm55ozL4MNk
        nifi.security.keyPasswd=gEZ4tIzBaYnytmLeSD0gXWzP20XVctX3Sm55ozL4MNk
        nifi.security.truststore=./conf/truststore.jks
        nifi.security.truststoreType=jks
        nifi.security.truststorePasswd=Oa3Sy6uAr5WgzkKq2DsK38Uce88XeT2dWOtu4p7g4W8
        nifi.security.needClientAuth=
        nifi.security.user.authorizer=managed-authorizer
        nifi.security.user.login.identity.provider=
        nifi.security.ocsp.responder.url=
        nifi.security.ocsp.responder.certificate=
```
6. 编辑/usr/local/Cellar/nifi/1.7.1/libexec/conf/authorizers.xml 文件以添加初始管理员标识。此条目需要与您在步骤4中用于生成证书的短语相匹配。(CN=username, OU=NIFI  之间要有空格)
```
    <authorizer>
        <identifier>file-provider</identifier>
        <class>org.apache.nifi.authorization.FileAuthorizer</class>
        <property name="Authorizations File">./conf/authorizations.xml</property>
        <property name="Users File">./conf/users.xml</property>
        <property name="Initial Admin Identity">CN=username, OU=NIFI</property>
        <property name="Legacy Authorized Users File"></property>

    <!--    <property name="Node Identity 1">CN=localhost, OU=nifi</property>  -->
    </authorizer>
```

## 在Mac上导入客户端证书  
1. 将您在步骤4中创建的.p12文件（/usr/local/Cellar/nifi-toolkit-1.7.1/bin/targetdocument/CN=username_OU=NIFI.p12）复制到Mac的用户目录。
2. 在启动台打开钥匙串访问。
3. 如下图所示导入证书，并填写秘钥（cat CN\=username_OU\=NIFI.password ）。设置为始终信任。
![SSL](/img/2018/09/11/SSL2-01.jpg)   
![SSL](/img/2018/09/11/SSL2-03.jpg)


## 在SSL下访问NiFi  
1. 启动nifi start，访问 https://localhost:9443/nifi   (查看启动日志：tail -f /usr/local/Cellar/nifi/1.7.1/libexec/logs/nifi-app.log)   
2. 选择证书。
![SSL](/img/2018/09/11/SSL03-01.jpg)
![SSL](/img/2018/09/11/SSL4-01.jpg)