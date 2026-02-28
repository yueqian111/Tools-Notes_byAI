# DVWA CSRF Low 级别漏洞分析与复现

---

## 一、漏洞概述

CSRF（Cross-Site Request Forgery，跨站请求伪造）是一种攻击方式，攻击者诱导受害者在已登录目标网站的状态下，执行非预期的操作（如修改密码、转账、发布内容等）。在DVWA的CSRF Low级别中，网站修改密码功能未采取任何防护措施，攻击者可轻易构造恶意请求，诱导受害者执行。

---

## 二、攻击复现步骤

1. **观察正常修改密码的请求**

    - 进入DVWA的CSRF模块，输入新密码（如`123456`）和确认密码，点击“Change”。

    - 观察URL栏参数：

        ```Plain Text
        
        http://127.0.0.1/dvwa/vulnerabilities/csrf/?password_new=123456&password_conf=123456&Change=Change#
        ```

    - 可见修改密码的操作通过**GET请求**传递参数，参数包括`password_new`（新密码）、`password_conf`（确认密码）和`Change`（提交标识）。

2. **抓包确认请求方式**

    - 使用Burp Suite等工具抓包，发现请求为GET方式，参数直接暴露在URL中：

        ```HTTP
        
        GET /dvwa/vulnerabilities/csrf/?password_new=123456&password_conf=123456&Change=Change HTTP/1.1
        Host: 127.0.0.1
        ...
        Cookie: PHPSESSID=s7o7abq7dok4h3nkm8d0vsv373; security=low
        ```

3. **构造恶意URL**

    - 攻击者可修改URL中的密码参数，构造恶意链接，例如将密码改为`456789`：

        ```Plain Text
        
        http://127.0.0.1/dvwa/vulnerabilities/csrf/?password_new=456789&password_conf=456789&Change=Change#
        ```

4. **诱导受害者执行攻击**

    - 当受害者（已登录DVWA）点击该恶意链接时，浏览器会自动携带登录Cookie发起请求。

    - 网站验证两次密码一致后，执行密码修改操作，页面显示“Password Changed.”，攻击成功。

---

## 三、源码分析

Low级别CSRF的PHP源码如下：

```PHP

<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["__mysqli_ston"]) && is_object($GLOBALS["__mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["__mysqli_ston"],  $pass_new ) : $pass_new);
        $pass_new = md5( $pass_new );

        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["__mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["__mysqli_ston"])) ? mysqli_error($GLOBALS["__mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["__mysqli_ston"]))) ? false : $___mysqli_res);
}

?>
```

**关键问题**：

- 仅通过`$_GET`获取参数，未校验请求来源（如Referer）。

- 仅验证两次密码是否一致，无CSRF Token等防护机制。

- 直接执行SQL更新操作，无二次验证（如原密码、验证码）。

---

## 四、漏洞危害

- **账户接管**：攻击者可诱导已登录的管理员点击恶意链接，直接修改管理员密码，接管网站后台。

- **数据篡改**：若存在其他敏感操作（如修改用户权限、删除数据），攻击者可构造对应请求，执行非预期操作。

- **信任滥用**：利用受害者与网站的信任关系，无需窃取密码即可执行操作，隐蔽性强。

---

## 五、防御建议

针对CSRF漏洞，可采取以下防护措施：

1. **使用CSRF Token**：在表单中嵌入随机Token，服务器验证Token有效性后再执行操作。

2. **校验Referer/Origin**：验证请求来源是否为可信域名，防止跨站请求。

3. **添加二次验证**：敏感操作（如修改密码）需验证原密码或输入验证码。

4. **避免GET请求执行敏感操作**：将修改密码等操作改为POST请求，降低攻击便利性。

5. **设置SameSite Cookie**：限制Cookie在跨站请求中自动携带，如`SameSite=Strict`或`SameSite=Lax`。
> （注：文档部分内容可能由 AI 生成）