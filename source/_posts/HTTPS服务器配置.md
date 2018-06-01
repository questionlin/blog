---
title: HTTPS服务器配置
date: 2018-06-01 15:43:38
tags: [http, 最佳实践]
---
### 一、SSL证书申请

1、确认需要申请证书的域名

2、生成私钥和csr文件  
在linux机器上执行以下命令生成私钥  
```
#openssl genrsa -out server.key 2048  
```
在linux机器上执行以下命令生成csr文件  
```
#openssl req -new -key server.key -out certreq.csr  
```
**以下黑色标识文字仅供参考，请根据商户自己实际情况进行填写**  
Country Name： **CN**                      //您所在国家的ISO标准代号，中国为CN  
State or Province Name：**guandong**       //您单位所在地省/自治区/直辖市  
Locality Name：**shenzhen**                 //您单位所在地的市/县/区  
Organization Name： **Tencent Technology (Shenzhen) Company Limited**                 //您单位/机构/企业合法的名称  
Organizational Unit Name： **R&D**         //部门名称  
Common Name： **www.example.com**     //通用名，例如：www.itrus.com.cn。此项必须与您访问提供SSL服务的服务器时所应用的域名完全匹配。  
Email Address：                          //您的邮件地址，不必输入，直接回车跳过  
"extra"attributes                        //以下信息不必输入，回车跳过直到命令执行完毕。  
执行上面的命令后，在当前目录下即可生成私钥文件**server.key**和**certreq.csr** csr文件

3、将生成的csr文件提交给第三方证书颁发机构申请对应域名的服务器证书，同时将私钥文件保存好，以免丢失。

4、证书申请后，证书颁发机构会提供服务器证书内容和两张中级CA证书，请按证书颁发机器说明生成服务器证书，此处假设服务器证书文件名称为**server.pem**

5、将生成的私钥文件**server.key**和服务器证书**server.pem**拷贝至服务器指定的目录即可进行HTTPS服务器配置

### 二、HTTPS服务器配置

1、 Nginx配置
```
server {
listen       443;   #指定ssl监听端口
server_name  www.example.com;
ssl on;    #开启ssl支持
ssl_certificate      /etc/nginx/server.pem;    #指定服务器证书路径
ssl_certificate_key  /etc/nginx/server.key;    #指定私钥证书路径
ssl_session_timeout  5m;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;     #指定SSL服务器端支持的协议版本
ssl_ciphers  ALL：!ADH：!EXPORT56：RC4+RSA：+HIGH：+MEDIUM：+LOW：+SSLv2：+EXP;    #指定加密算法
ssl_prefer_server_ciphers   on;    #在使用SSLv3和TLS协议时指定服务器的加密算法要优先于客户端的加密算法
#以下内容请按域名需要进行配置，此处仅供参考
location / {
return 444;
}
}
```
2、其它web服务器配置
请参考文档：http://www.itrus.cn/html/fuwuyuzhichi/fuwuqizhengshuanzhuangpeizhizhinan 《服务器证书配置指南》

三、相关事项

1、证书颁发机构  
推荐天威诚信，具体请见：http://www.itrus.com.cn

2、 参考文档  
http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_prefer_server_ciphers 《ngx_http_ssl_module》
http://nginx.org/cn/docs/http/configuring_https_servers.html 《nginx配置HTTPS服务器》
http://www.itrus.cn/html/fuwuyuzhichi/fuwuqizhengshuanzhuangpeizhizhinan 《服务器证书配置指南》

3、常见问题  
（1）证书受信任的问题  
部分国内签发的SSL证书，在Android上不受信任，推荐GeoTrust；  
（2）如果页面有动静分离，静态资源使用独立域名的话，也需要为该域名申请证书；  
（3）android低版本不支持SNI扩展，受此限制，一台服务器只能部署一个数字证书；


转自 https://pay.weixin.qq.com/wiki/doc/api/micropay.php?chapter=10_4