---
layout: Yubikey&Canokey PIV 配置
title: Yubikey&Canokey PIV 配置
categories:
  - - 折腾记录
tags:
  - yubikey
  - canokeys
date: 2024-11-20 11:40:23
---
# 准备

-  Yubikey & Canokey
-  [yubico-piv-tool]([Releases](https://developers.yubico.com/yubico-piv-tool/Releases/)) & [ykman]([Releases](https://developers.yubico.com/yubikey-manager/Releases/))
-  [WinCryptSSHAgent (Windows)]([buptczq/WinCryptSSHAgent: Using a Yubikey for SSH Authentication on Windows Seamlessly](https://github.com/buptczq/WinCryptSSHAgent))
# 生成私钥及证书
## CA 私钥和证书

```shell
openssl genpkey -algorithm RSA -out ca_private_key.pem -pkeyopt rsa_keygen_bits:2048
openssl req -new -x509 -key ca_private_key.pem -out ca_cert.pem -days 3650 -subj "/C=CN/ST=Guangdong/L=Guangzhou/O=XXX/OU=XXX/CN=taoidle"
```
## RSA 私钥和CSR

```shell
openssl genpkey -algorithm RSA -out rsa_private_key.pem -pkeyopt rsa_keygen_bits:2048
openssl req -new -key rsa_private_key.pem -out csr.pem -subj "/C=CN/ST=Guangdong/L=Guangzhou/O=XXX/OU=XXX/CN=taoidle"
```
## 使用CA签发CSR

```shell
openssl x509 -req -in csr.pem -CA ca_cert.pem -CAkey ca_private_key.pem -CAcreateserial -out signed_cert.pem -days 3650
```
## 证书验证

### CSR验证

```shell
openssl req -in csr.pem -noout -text
```
### 验证生成证书

```shell
openssl x509 -in signed_cert.pem -noout -text
```
### 证书链验证

```shell
openssl verify -CAfile ca_cert.pem signed_cert.pem
```
### 自签证书链验证

```shell
openssl verify -CAfile signed_cert.pem signed_cert.pem
```

- `ca_private_key.pem`：CA 私钥。
- `ca_cert.pem`：自签名 CA 证书。
- `rsa_private_key.pem`： RSA 私钥。
- `csr.pem`：证书签名请求。
- `signed_cert.pem`：CA 签发的证书。

# Yubikey PIV
## yubico-piv-tool
### 导入私钥

```shell
yubico-piv-tool -a import-key -s 9a -i rsa_private_key.pem
```
### 导入证书

```shell
yubico-piv-tool -a import-key -s 9a -i signed_cert.pem
```
## ykman

- 把 rsa_private_key.pem、signed_cert.pem cat出来，对应到私钥、证书，输入后留空一行然后ctrl+c取消后输入pin
### 导入私钥

```shell
ykman piv keys import 9a -
```
### 导入证书

```shell
ykman piv certificates import 9a -
```
# Canokey PIV
## yubico-piv-tool
### 导入私钥

```shell
yubico-piv-tool -r canokey -a import-key -s 9a -i rsa_private_key.pem
```
### 导入证书

```shell
yubico-piv-tool -r canokey -a import-key -s 9a -i signed_cert.pem
```
## ykman

- 把 rsa_private_key.pem、signed_cert.pem cat出来，对应到私钥、证书，输入后留空一行然后ctrl+c取消后输入pin
### 导入私钥

```shell
ykman -r canokey piv keys import 9a -
```
### 导入证书

```shell
ykman -r canokey piv certificates import 9a -
```

# SSH自动登录
## Windows
 下载 WinCryptSSHAgent，启动后右键托盘图标 ，`show public key`。将公钥放到服务器.ssh/authorized_keys