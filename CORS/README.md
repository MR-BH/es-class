# 浏览器跨域详解

## 准备工作
### Charles装包用

### 生成本地开发用的HTTPS证书

需要安装openssl
	
	// 生成证书秘钥
	openssl genrsa -out key.pem 2048
	// 生成证书
	openssl req -new -x509 -key key.pem -out cert.pem -days 1095

## 浏览器中的同源策略

### 同源的含义

* 协议相同
* 域名相同
* 端口相同

### 非同源限制范围

* Cookie、LocalStorage和IndexDB无法读取
* DOM无法获得
* AJAX请求不能发送


## Cookie详解

## JSONP详解

## CORS详解

### 简单请求

### 非简单请求

### 与CORS相关的首部字段

## fetch

## XHR

