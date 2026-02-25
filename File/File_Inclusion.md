# 模块5：文件包含 LFI（第6周）笔记

---

## 一、核心原理 💡

本地文件包含（Local File Inclusion, LFI）是指Web应用在处理文件包含逻辑时，直接将用户可控的参数作为文件名，且未进行任何过滤或校验，导致攻击者可以构造恶意参数，让应用读取或执行服务器上的任意本地文件。

---

## 二、必学内容 📌

### 1. 读取系统敏感文件

- **利用方式**：通过构造URL参数，让应用包含系统中的敏感文件。

- **示例**：

    ```Plain Text
    
    ?page=/etc/passwd
    ```

- **说明**：`/etc/passwd` 是Linux系统中存储用户基本信息的文件，成功读取后可获取服务器用户列表。

### 2. PHP伪协议读取源码

- **利用方式**：使用PHP内置的 `php://filter` 伪协议，读取PHP文件的原始源码，避免其被服务器解析执行。

- **示例**：

    ```Plain Text
    
    ?page=php://filter/convert.base64-encode/resource=index.php
    ```

- **说明**：

    - `convert.base64-encode`：将文件内容以Base64编码输出，避免浏览器直接解析PHP代码。

    - `resource=index.php`：指定要读取的目标文件。

    - 读取后，将Base64编码内容解码即可得到原始PHP源码。

---

## 三、实操：DVWA File Inclusion Low 级别

### 环境准备

- 启动DVWA环境，将安全级别设置为 **Low**。

- 进入 `File Inclusion` 模块。

### 步骤1：读取系统文件

1. 在URL中构造参数：

    ```Plain Text
    
    http://dvwa_ip/vulnerabilities/fi/?page=/etc/passwd
    ```

2. 提交后，页面将直接显示 `/etc/passwd` 文件的内容。

### 步骤2：使用PHP伪协议读取源码

1. 构造URL读取当前页面源码：

    ```Plain Text
    
    http://dvwa_ip/vulnerabilities/fi/?page=php://filter/convert.base64-encode/resource=index.php
    ```

2. 页面会返回一段Base64编码的字符串。

3. 将该字符串进行Base64解码，即可得到 `index.php` 的原始PHP代码。

---

## 四、关键注意事项 ⚠️

- **路径穿越**：当应用限制了文件目录时，可使用 `../` 进行路径穿越，如 `?page=../../etc/passwd`。

- **文件包含与文件上传结合**：若存在文件上传漏洞，可上传包含恶意代码的文件，再通过文件包含执行该代码，形成RCE（远程代码执行）。

- **防护措施**：严格过滤用户输入，限制可包含的文件目录和类型，避免直接使用用户可控参数作为文件名。

---
> （注：文档部分内容可能由 AI 生成）