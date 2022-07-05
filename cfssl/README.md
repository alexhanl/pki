# 证书管理

## 最佳实践 
- 构建企业内部 Root CA 和 Intermediate CA，通过 Intermediate CA 签发所有的证书。
- 通过 Let's Encrypt 获取官方 CA 签发的证书，前提是必须要有自有域名。 
- 通过 Windows 域管理自动在客户端（比如操作系统、浏览器）中添加信任 Root CA 的证书。 
- 其他不方便自动加入的证书信任的客户端，建议应用采用 Let's Encrypt 签发的官方证书

## 使用 cfssl 证书工具进行企业证书管理 —— Cheat Sheet

### 下载和安装 cfssl 证书工具
https://github.com/cloudflare/cfssl/releases/tag/v1.6.1

需要下载 cfssl、cfssl-certinfo、cfssljson

### 创建 Root CA
1. 准备 root-ca-csr.json
2. 创建 Root CA 的 key 和证书：
```
cfssl gencert -initca root-ca-csr.json | cfssljson -bare root-ca 
```

### 创建 Intermediate CA - dev 部门的 Intermediate CA
1. 准备 dev-intermediate-ca-csr.json
2. 创建 Intermediate CA 的 key 和 证书
```
cfssl genkey -initca dev-intermediate-ca-csr.json | cfssljson -bare dev-intermediate-ca
```
3. 通过 Root CA 对 Intermedia CA 的证书进行签名
```
cfssl sign -ca=root-ca.pem -ca-key=root-ca-key.pem -config=config.json -profile=intermediate_ca dev-intermediate-ca.csr | cfssljson -bare dev-intermediate-ca
```

### 申请 web.dev.hanliang.link 证书
1. 准备 web-dev-hanliang-link-csr.json
2. 通过 Intermediate CA 签发 web.dev.hanliang.link 的 key 和证书
```
cfssl gencert -ca=dev-intermediate-ca.pem -ca-key=dev-intermediate-ca-key.pem -config=config.json -profile=server dev-hanliang-link-csr.json | cfssljson -bare dev-hanliang-link
```
3. 由于在客户端中只添加了 Root CA 的信任，但是当前信任关系中需要包含 Intermediate CA，所需要构建表示信任链的证书链
```
cat dev-hanliang-link.pem dev-intermediate-ca.pem root-ca.pem > dev-hanliang-link-chain.pem
```

### 使用 web.dev.hanliang.link 证书
1. 配置 Nginx Server 或者其他网站的 443 端口和证书配置，按照模板配置即可
2. 在客户端浏览器或者操作系统（chrome）导入根证书 root-ca.pem，这时我们应该能够看到网站打开时，不再提示证书问题，在浏览器网址栏的左边出现一个锁的图标，表示网站是可信任的。 

### Revoke 证书 （TODO）
1. Revoke 证书
2. 通过 OCSP responder 去验证证书状态：Revoking certificates and running OCSP responder：https://propellered.com/posts/cfssl_revoking_certs_ocsp_reponder/


## Troubelshooting
1. 查看证书是否正确：使用 https://csr.chinassl.net/decoder-ssl.html。 注意证书如果已经经过 Base 64 Encoding，则需要首先 decode。Decode之后的证书格式应该是 -----BEGIN CERTIFICATE----- 开头的文本。 


## 参考资料
1. 李国强的博客：[企业CA系列一 搭建企业根CA和中间CA](
https://www.toutiao.com/article/6844782478880670211/?app=news_article&timestamp=1655970987&use_new_style=1&req_id=202206231556260101351620280F1487D1&group_id=6844782478880670211&share_token=16AE4B8A-6FB2-487B-8C21-94D7866CAF32&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_ios&utm_campaign=client_share&wxshare_count=1&source=m_redirect&wid=1655971046761)

2. [How to use cfssl to create self signed certificates](https://rob-blackbourn.medium.com/how-to-use-cfssl-to-create-self-signed-certificates-d55f76ba5781)

3. [CFSSL使用方法 以及 vSphere 证书替换](https://www.ethanzhang.xyz/cfssl%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95/)



