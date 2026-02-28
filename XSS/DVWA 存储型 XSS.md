# DVWA 存储型 XSS 学习指南

### 🛡️ 存储型XSS（Stored XSS）学习指南

这是 **DVWA（Damn Vulnerable Web Application）** 中的存储型XSS练习页面，核心是让恶意脚本被服务器存储，并在其他用户访问时自动执行。下面是详细的学习步骤和利用方法：

---

## 1. 漏洞原理

存储型XSS（也叫持久型XSS）的特点是：

- 恶意脚本被**提交并存储到服务器数据库**中

- 当其他用户访问该页面时，服务器会将恶意脚本直接返回给浏览器执行

- 危害极大，可实现“一次注入，多次攻击”，影响所有访问者

在这个留言板场景中，`Name`和`Message`输入框就是典型的攻击入口。

---

## 2. 不同安全等级的利用方法

### 🔹 Low 等级（无防护，最易利用）

**目标**：直接注入脚本并触发弹窗。

1. 在 `Name` 输入框填写任意内容（如 `test`）

2. 在 `Message` 输入框中输入 payload：

    ```HTML
    
    <script>alert('Stored XSS')</script>
    ```

3. 点击 **Sign Guestbook** 提交

4. 页面刷新后，会弹出包含“Stored XSS”的对话框，说明脚本成功执行

进阶 payload：窃取当前用户的 Cookie

```HTML

<script>alert(document.cookie)</script>
```

提交后，弹窗会显示你的会话 Cookie，可用于会话劫持。

---

### 🔹 Medium 等级（基础过滤）

DVWA 在 Medium 等级会过滤 `<script>` 标签，需要用其他标签绕过。

**常用绕过 payload**：

- 利用图片加载错误触发：

    ```HTML
    
    <img src=x onerror=alert('XSS')>
    ```

- 利用 SVG 加载事件：

    ```HTML
    
    <svg onload=alert(1)>
    ```

- 双写 script 标签（如果过滤逻辑是简单替换）：

    ```HTML
    
    <scr<script>ipt>alert('XSS')</script>
    ```

提交后，同样会触发弹窗。

---

### 🔹 High 等级（严格防护）

High 等级会对输入进行更严格的过滤和转义，直接注入脚本难度很高。通常需要结合其他漏洞或利用 DOM 操作。

**思路**：

- 分析页面源码，寻找未被转义的输出点

- 尝试利用页面的 JavaScript 逻辑漏洞

- 若无法直接注入，可考虑结合其他漏洞（如 CSRF）触发 XSS

---

## 3. 防护措施

要防止存储型XSS，核心是**输入验证 + 输出编码**：

1. **输入验证**：

    - 对用户输入进行严格校验，只允许合法字符和格式

    - 限制输入长度，避免超长 payload

2. **输出编码**：

    - 在将用户输入渲染到页面时，进行 HTML 编码：

        - `<` → `&lt;`

        - `>` → `&gt;`

        - `&` → `&amp;`

        - `"` → `&quot;`

    - 使用现代前端框架（如 React、Vue），它们会自动对输出进行编码

3. **安全配置**：

    - 为 Cookie 设置 `HttpOnly` 属性，防止 XSS 窃取会话

    - 实施 **内容安全策略（CSP）**，限制页面可执行的脚本来源
> （注：文档部分内容可能由 AI 生成）