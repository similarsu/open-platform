# 概述
`该项目基于spring boot 2.0，使用gradle作为构建工具。`

# 一、ssl
## 1.1、ssl生成
### 1.1.1、CA证书

#### 创建私钥(ca.key)
```bash
openssl genrsa -out ca/ca.key
```
#### 创建请求(ca.csr)
```bash
openssl req -new -out ca/ca.csr -key ca/ca.key -subj '/C=CN/ST=ZhengJiang/L=WenZhou/O=SimilarSu CA Corp'
```
#### 自签署证书(ca.crt)
```bash
openssl x509 -req -in ca/ca.csr -out ca/ca.crt -signkey ca/ca.key -days 3650 -extensions v3_ca
```
#### 将.crt 文件导入到JKS文件(ca.jks)
```bash
keytool -keystore ca/ca.jks -keypass cacajks -storepass cacajks -alias ca -import -trustcacerts -file ca/ca.crt
```

### 1.1.2、server端证书

#### 创建私钥（server.key）
```bash
openssl genrsa -out server/server.key
```
#### 创建请求(server.csr)
```bash
openssl req -new -out server/server.csr -key server/server.key -subj '/C=CN/ST=Zhengjiang/L=WenZhou/O=SimilarSu Server Corp/OU=dev/CN=localhost'
```
#### 签署证书(server.crt)
```bash
openssl x509 -req -in server/server.csr -out server/server.crt -signkey server/server.key -CA ca/ca.crt -CAkey ca/ca.key -CAcreateserial -days 3650  -extensions v3_ca
```
#### 导出.p12格式(server.p12)
```bash
openssl pkcs12 -export -in server/server.crt -inkey server/server.key -out server/server.p12 -passout pass:serverp12
```
#### 将.p12 文件导入到JKS文件(server.keystore)
```bash
keytool -importkeystore -v -srckeystore  server/server.p12 -srcstoretype pkcs12 -srcstorepass serverp12 -destkeystore server/server.jks -deststoretype jks -deststorepass serverp12
```
`注意：srcstorepass与deststorepass需要相同`

### 1.1.3、client端证书

#### 创建私钥（client.key）
```bash
openssl genrsa -out client/client.key
```
#### 创建请求(client.csr)
```bash
openssl req -new -out client/client.csr -key client/client.key -subj '/C=CN/ST=Zhengjiang/L=WenZhou/O=SimilarSu Server Corp/OU=dev/CN=client'
```
#### 签署证书(client.crt)
```bash
openssl x509 -req -in client/client.csr -out client/client.crt -signkey client/client.key -CA ca/ca.crt -CAkey ca/ca.key -CAcreateserial -days 3650  -extensions v3_ca
```
#### 导出.p12格式(client.p12)
```bash
openssl pkcs12 -export -in client/client.crt -inkey client/client.key -out client/client.p12 -passout pass:clientp12
```
## 1.2、ssl配置
### 1.2.1、单向
#### 服务端
```yml
server:
  ssl:
    key-store: classpath:ssl/server/server.jks
    key-password: serverp12
```
#### 客户端
```
使用ca.crt(非必要)
```
### 1.2.2、双向
#### 服务端
```yml
server:
  ssl:
    key-store: classpath:ssl/server/server.jks
    key-password: serverp12
    trust-store: classpath:ssl/ca/ca.jks
    trust-store-password: cacajks
    client-auth: need
```
#### 客户端
```
使用ca.crt(非必要)
需倒入client.p12
```