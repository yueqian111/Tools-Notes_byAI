# 暴力破解（Brute Force）各安全级别信息提取

#### 🔓 Low 级别

- **漏洞分析**：无防护，可直接爆破或SQL注入绕过

- **操作步骤**：

    1. Burp抓包，发送到Intruder

    2. 清除默认标记，标记`username`和`password`

    3. 选择Cluster Bomb模式，加载用户名字典（如`admin`）和密码字典

    4. 开始攻击，找到状态码200的有效组合

    5. 快速绕过：用户名输入`admin' or '1'='1`，密码任意，直接登录

---

#### 🔒 Medium 级别

- **漏洞分析**：对用户名做了`mysqli_real_escape_string`过滤，密码无严格限制，有基础防暴机制

- **操作步骤**：

    1. 抓包发现请求为POST方式，参数有`username`、`password`

    2. 用户名无法SQL注入，专注密码爆破

    3. 由于无token，直接用Burp爆破密码，使用常用密码字典

    4. 成功找到密码`password`

---

#### 🔐 High 级别

- **漏洞分析**：添加了Anti-CSRF Token，每次请求token不同，增加爆破难度

- **操作步骤**：

    1. 抓包发现多了`user_token`参数

    2. 在Burp中设置Grep - Extract提取token

    3. 设置Payload Processing自动更新token值

    4. 选择Pitchfork模式，同时处理密码和token

    5. 开始攻击，成功获取有效凭证

---

#### ❌ Impossible 级别

- **防护机制**：3次错误锁定15分钟，采用PDO参数化查询防SQL注入

- **绕过难度**：几乎无法绕过，仅作防御示范

---

要不要我帮你把这些步骤整理成一份**可直接复制的Burp Suite操作清单**，方便你在CTF练习中快速复现？
> （注：文档部分内容可能由 AI 生成）