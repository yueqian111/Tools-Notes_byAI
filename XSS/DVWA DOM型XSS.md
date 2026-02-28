# DVWA DOM型XSS（Low级别）学习笔记

## 一、场景与观察

在DVWA的DOM Based Cross Site Scripting（Low级别）页面中：

- 页面提供了一个语言选择下拉框（如French、English等）。

- 当选择不同语言并点击“Select”后，URL栏中的`default`参数会同步变化，例如：

    ```Plain Text
    
    http://172.16.16.246/dvwa/vulnerabilities/xss_d/?default=French
    ```

- 这说明`default`参数是**用户可控**的，且被前端脚本读取并渲染到页面中，是潜在的XSS注入点。

---

## 二、攻击步骤

1. **构造恶意Payload**

利用`default`参数直接传入XSS代码，构造如下URL：

```Plain Text

http://172.16.16.246/dvwa/vulnerabilities/xss_d/?default=<script>alert(1)</script>
```

1. **触发XSS执行**

访问构造的URL后，页面会弹出`alert(1)`的弹窗，证明XSS攻击成功。

---

## 三、原理分析

- **Low级别防护缺失**：该级别下，前端脚本直接将URL中的`default`参数值作为DOM内容插入，没有对特殊字符（如`<`、`>`、`script`标签）进行任何过滤或转义。

- **DOM型XSS本质**：恶意代码并非由服务器端返回，而是由客户端JavaScript直接解析并执行，属于客户端层面的漏洞。

---

## 四、防御思路

1. **输出转义**：将用户可控数据插入DOM前，对特殊字符（`<`、`>`、`&`、`"`、`'`等）进行HTML实体转义，避免被解析为代码。

2. **避免危险API**：不使用`document.write`、`innerHTML`等直接解析HTML的方式插入用户输入，优先使用`textContent`等安全API。

3. **输入验证**：对`default`这类参数进行严格的白名单验证，只允许预期的合法值（如语言代码），拒绝其他字符。
> （注：文档部分内容可能由 AI 生成）