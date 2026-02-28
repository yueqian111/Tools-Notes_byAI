# DVWA 验证码学习指南：从环境修复到漏洞详解

针对你在 DVWA 靶场中遇到的 **Insecure CAPTCHA（不安全验证码）** 模块报错及学习需求，这份指南将分为两部分：首先解决 `API key missing` 的环境配置问题，然后分级别详解漏洞原理、利用方法（结合 Burp Suite）及源码分析，完全适配你作为信息安全大一学生的学习节奏。

---

### 第一部分：环境修复（解决 API Key 缺失）

页面提示的核心原因是未配置 Google reCAPTCHA 密钥，而国内环境无法正常访问该服务。**无需科学上网**，按以下步骤即可快速修复，进入漏洞学习环节：

#### 方法：直接填写测试密钥（推荐）

编辑 DVWA 的配置文件，填入公开可用的测试密钥，即可正常加载页面。

1. **打开配置文件**：在 Kali 虚拟机中，通过命令打开配置文件：

    ```Bash
    
    vim /var/www/html/dvwa/config/config.inc.php
    ```

2. **修改密钥配置**：找到以下两行代码，将空值替换为测试密钥：

    ```PHP
    
    // 原代码
    // $_DVWA['recaptcha_public_key'] = '';
    // $_DVWA['recaptcha_private_key'] = '';
    
    // 替换为测试密钥（直接复制即可）
    $_DVWA['recaptcha_public_key'] = '6LdK7xITAAzzAAJQTfL7fu6I-0aPl8KHHieAT_yJg';
    $_DVWA['recaptcha_private_key'] = '6LdK7xITAzzAAL_uw9YXVUOPoIHPZLfw2K1n5NVQ';
    ```

3. **保存并刷新**：在 Vim 中输入 `:wq` 保存，刷新浏览器页面，报错消失，显示密码修改表单。

---

### 第二部分：漏洞核心学习（Insecure CAPTCHA）

该漏洞的核心并非验证码本身被破解，而是**业务逻辑设计缺陷**——服务器仅通过客户端传递的参数判断是否通过验证，而非在关键步骤重新校验，导致攻击者可直接绕过验证码环节。

#### 通用准备步骤

1. 登录 DVWA，将安全级别设为 `Low`（右上角 `DVWA Security`）。

2. 打开 Burp Suite，配置浏览器代理，开启拦截功能（Intercept is on）。

---

#### 级别一：Low（基础逻辑绕过）

##### 1. 漏洞原理

源码将改密操作分为两步：`step=1`（验证验证码）和 `step=2`（执行改密）。但 `step=2` 未再次验证验证码，仅检查 `step` 参数值，攻击者可直接构造 `step=2` 的请求绕过验证。

##### 2. 利用步骤（Burp Suite 实操）

1. 在靶场页面输入新密码（如 `123456`）和确认密码，点击 `Change`。

2. Burp Suite 拦截到请求，发送至 **Repeater**（右键 → Send to Repeater）。

3. **修改请求参数**：将 `step=1` 改为 `step=2`，删除无关的验证码参数（如 `g-recaptcha-response`）。

    ```HTTP
    
    // 修改前
    POST /dvwa/vulnerabilities/captcha/ HTTP/1.1
    ...
    password_new=123456&password_conf=123456&step=1&Change=Change
    
    // 修改后
    POST /dvwa/vulnerabilities/captcha/ HTTP/1.1
    ...
    password_new=123456&password_conf=123456&step=2&Change=Change
    ```

4. 点击 Repeater 中的 `Send`，响应包显示 `Password Changed`，绕过成功。

##### 3. 源码关键片段

```PHP

// 仅检查 step=2，未验证验证码是否真的通过
if( isset( $_POST['Change'] ) && ( $_POST['step'] == '2' ) ) {
    // 直接执行改密逻辑
}
```

---

#### 级别二：Medium（新增参数校验的绕过）

##### 1. 漏洞原理

在 Low 级别基础上，新增了 `passed_captcha=true` 参数校验，要求客户端证明“已通过验证码”。但该参数仍由客户端控制，攻击者可手动添加。

##### 2. 利用步骤

1. 同上，输入密码后抓包，发送至 Repeater。

2. **修改两个参数**：`step=1` → `step=2`，并添加 `passed_captcha=true`。

    ```HTTP
    
    password_new=123456&password_conf=123456&step=2&passed_captcha=true&Change=Change
    ```

3. 发送请求，改密成功。

##### 3. 源码关键片段

```PHP

if( isset( $_POST['Change'] ) && ( $_POST['step'] == '2' ) ) {
    // 仅检查参数是否存在，未验证真实性
    if( !$_POST['passed_captcha'] ) {
        $html .= "<pre><br />You have not passed the CAPTCHA.</pre>";
        return;
    }
    // 执行改密
}
```

---

#### 级别三：High（结合客户端校验的绕过）

##### 1. 漏洞原理

要求验证码响应值为固定字符串 `hidd3n_valu3`，且 User-Agent 为 `reCAPTCHA`。虽增加了校验条件，但参数仍可伪造。

##### 2. 利用步骤

1. 抓包后发送至 Repeater。

2. **修改参数+请求头**：

    - 请求体：添加 `g-recaptcha-response=hidd3n_valu3`，`step=2`，`passed_captcha=true`。

    - 请求头：添加 `User-Agent: reCAPTCHA`。

3. 发送请求，绕过成功。

---

#### 级别四：Impossible（安全实现）

##### 1. 安全措施

- 强制验证当前密码（身份校验）。

- 使用 CSRF Token 防止伪造请求。

- 后端严格校验 reCAPTCHA 响应的真实性。

##### 2. 防御启示

1. **关键操作双重校验**：敏感操作（改密、转账）必须重新验证身份，而非仅依赖前端参数。

2. **服务端独立校验**：验证码的验证逻辑必须在服务端完成，且结果由服务端存储（如 Session），不依赖客户端传递的 `passed_captcha` 等参数。

3. **使用成熟方案**：优先采用 Google reCAPTCHA、阿里云验证码等成熟服务，避免自行实现。
> （注：文档部分内容可能由 AI 生成）