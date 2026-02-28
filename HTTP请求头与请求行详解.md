# Burp Suite 抓包核心 - HTTP请求头与请求行详解

适用场景：CTF Web题分析、HTTP协议学习、Burp Suite抓包实操

整理时间：2026年2月

备注：信息安全专业备赛CTF/蓝桥杯专用笔记

## 目录

1. [请求行核心组成](#请求行核心组成)

2. [常见请求头字段说明](#常见请求头字段说明)

3. [CTF实操小提示](#ctf实操小提示)

## 一、请求行核心组成

请求行是HTTP请求的第一行，包含客户端对服务器的核心操作指令，格式为：`Method URL HTTP版本`。

|字段|核心作用|常见值/示例|
|---|---|---|
|**Method（请求方法）**|指示对服务器资源的操作类型|GET、POST、PUT、DELETE、HEAD、OPTIONS|
|**URL/路径**|目标资源的完整地址，包含路径和查询参数|`/index.php?id=1`|
|**HTTP版本**|标识使用的HTTP协议版本，决定通信规则|HTTP/1.1（最常用）、HTTP/2|
## 二、常见请求头字段说明

请求头字段紧跟请求行，以`键值对`形式存在，用于传递客户端信息、请求偏好、会话状态等关键数据，是Burp Suite抓包分析的核心重点。

|字段名|核心作用|示例/关键值|
|---|---|---|
|**Host**|指定目标服务器的域名和端口号，服务器据此定位目标主机|`Host: www.example.com:8080`|
|**User-Agent**|传递客户端软硬件信息（系统、浏览器、引擎），服务器用于识别客户端类型|`User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/121.0.0.0`|
|**Accept**|声明客户端可接收的响应内容类型|`Accept: text/html, application/json`|
|**Accept-Language**|声明客户端首选的语言类型|`Accept-Language: zh-CN,zh;q=0.9`|
|**Accept-Encoding**|声明客户端支持的内容压缩方式，减少传输体积|`Accept-Encoding: gzip, deflate`|
|**Connection**|控制请求的连接行为，决定连接是否保持|`Connection: keep-alive`（长连接）、`close`（短连接）|
|**Cookie**|存储客户端会话信息，用于身份验证、状态保持（CTF高频考点）|`Cookie: session=abc123; uid=10086`|
|**Referer**|指示请求的来源页面URL，用于防盗链、溯源|`Referer: https://www.example.com/login`|
|**Content-Type**|POST/PUT请求专用，声明请求体的数据类型（CTF传参核心）|`application/x-www-form-urlencoded`、`multipart/form-data`|
|**Content-Length**|声明请求体的字节长度，服务器据此接收完整请求体|`Content-Length: 20`|
## 三、CTF实操小提示

1. **Method篡改**：部分题目通过修改请求方法（如GET改POST、POST改PUT）可绕过访问限制。

2. **Cookie伪造**：缺失或篡改Cookie字段常是身份绕过题的突破口。

3. **Referer校验**：防盗链题目可通过伪造Referer字段绕过服务器校验。

4. **Content-Type适配**：文件上传题需将Content-Type改为对应文件类型（如`image/jpeg`）。
> （注：文档部分内容可能由 AI 生成）