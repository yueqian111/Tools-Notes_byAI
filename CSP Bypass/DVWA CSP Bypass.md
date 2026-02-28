# DVWA CSP Bypass

结合你作为信息安全专业大一学生的背景，本教程按**Low/Medium/High/Impossible**四个安全级别，提供**原理+可直接执行的实操步骤+源码分析**，同时补充CTF竞赛核心考点，适配你搭建靶场、备赛CTF的需求。

## 前置核心知识

CSP（Content Security Policy）是浏览器端的安全机制，通过**白名单**限制网页可加载/执行的资源（核心是`script-src`指令控制JS来源），用于缓解XSS。本模块的核心是利用**CSP配置缺陷**绕过限制，执行恶意JS。

---

## 一、Low级别：第三方可信域滥用

### 1. 核心原理

服务器配置的CSP头过度信任第三方公开平台（如Pastebin），允许加载这些域的脚本；且用户输入的URL会直接拼接到`<script src="这里">`中，无过滤。

### 2. 实操步骤（可直接执行）

1. 打开DVWA，切换安全级别为`Low`，进入`CSP Bypass`模块。

2. 访问第三方公开脚本托管平台：`https://digi.ninja/dvwa/`（官方提供的测试脚本，无需自己上传）。

3. 选择现成的恶意脚本地址：`https://digi.ninja/dvwa/alert.js`（内容为`alert(document.cookie)`）。

4. 将该URL填入输入框，点击`Include`，页面会弹出包含Cookie的弹窗，绕过成功。

### 3. 源码分析（`source/low.php`）

```PHP

// CSP头：信任自身+多个第三方域（包括digi.ninja）
$headerCSP = "Content-Security-Policy: script-src 'self' https://pastebin.com hastebin.com ... https://digi.ninja ;";
header($headerCSP);

// 用户输入直接拼接到<script>标签的src属性，无过滤
if (isset($_POST['include'])) {
    $page['body'] .= "<script src='".$_POST['include']."'></script>";
}
```

### 4. CTF考点

**第三方域白名单滥用**：若目标CSP信任`cdnjs.cloudflare.com`等公共CDN，可搜索该CDN上的恶意脚本或利用其JSONP接口绕过。

---

## 二、Medium级别：Nonce配置失效+Unsafe-inline

### 1. 核心原理

服务器虽配置了`nonce`（随机令牌），但同时保留了`'unsafe-inline'`（允许内联脚本），且用户输入直接拼接到页面中，导致`nonce`失效，内联脚本可直接执行。

### 2. 实操步骤（可直接执行）

1. 切换DVWA安全级别为`Medium`，进入`CSP Bypass`模块。

2. 构造包含**官方固定nonce**的内联脚本Payload：

    ```HTML
    
    <script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert('Medium级别CSP绕过成功')</script>
    ```

    （注：该nonce的Base64解码为`Never going to give you up`，是固定值）

3. 将Payload填入输入框，点击`Include`，弹窗出现即绕过成功。

### 3. 源码分析（`source/medium.php`）

```PHP

// 关键缺陷：同时存在'unsafe-inline'和固定nonce，且禁用了浏览器XSS保护
$headerCSP = "Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=';";
header($headerCSP);
header("X-XSS-Protection: 0");

// 用户输入直接拼接到页面，无任何过滤
if (isset($_POST['include'])) {
    $page['body'] .= " " . $_POST['include'] . "";
}
```

### 4. CTF考点

**Nonce硬编码/可预测**：若目标nonce固定、可被猜测或从其他接口泄露，可直接携带nonce构造内联脚本。

---

## 三、High级别：同源JSONP回调注入

### 1. 核心原理

CSP仅允许同源脚本（`script-src 'self'`），但页面加载了同源的`high.js`，该JS会动态创建`<script>`标签请求`jsonp.php`的JSONP接口；而`jsonp.php`未校验`callback`参数，会将其直接反射到响应中，导致恶意函数被执行。

### 2. 实操步骤（可直接执行）

1. 切换DVWA安全级别为`High`，进入`CSP Bypass`模块（页面无输入框，只有按钮）。

2. 打开浏览器开发者工具（F12）→`Network`→`Fetch/XHR`，点击页面的`solve`按钮，捕获到请求：`jsonp.php?callback=solveSum`。

3. 构造恶意`callback`参数，修改请求为：

    ```Plain Text
    
    http://你的DVWA地址/vulnerabilities/csp/source/jsonp.php?callback=alert(document.cookie)
    ```

4. 直接访问该URL，或用Burp Suite拦截请求修改`callback`值，页面会弹出Cookie弹窗，绕过成功。

### 3. 源码分析

- `source/high.php`：CSP仅允许同源脚本

    ```PHP
    
    $headerCSP = "Content-Security-Policy: script-src 'self';";
    header($headerCSP);
    $page['body'] .= '<script src="source/high.js"></script>';
    ```

- `jsonp.php`：核心缺陷——`callback`参数无校验，直接反射

    ```PHP
    
    header("Content-Type: application/javascript");
    $callback = $_GET['callback'];
    echo $callback . "(1+2+3+4+5);"; // 直接拼接用户输入，执行恶意函数
    ```

### 4. CTF考点

**同源JSONP接口滥用**：是CTF CSP绕过的高频考点，需寻找目标的JSONP接口（如`/api/jsonp?callback=xxx`），构造`callback=恶意代码`实现绕过。

---

## 四、Impossible级别：安全参考实现

### 1. 核心原理

保留了`script-src 'self'`的严格CSP，同时**硬编码JSONP的回调函数名**，拒绝用户可控的`callback`参数，彻底阻断注入路径。

### 2. 源码分析（`jsonp_impossible.php`）

```PHP

header("Content-Type: application/javascript");
$allowedCallbacks = ['calculateSum']; // 白名单仅允许固定函数
$callback = $_GET['callback'] ?? '';

// 仅允许白名单内的函数，无用户可控空间
if (in_array($callback, $allowedCallbacks)) {
    echo $callback . "(".json_encode(["answer" => 15]).")";
} else {
    echo "calculateSum(".json_encode(["answer" => 15]).")";
}
```

### 3. 防御启示（CTF/实战必备）

1. 禁用`'unsafe-inline'`和`'unsafe-eval'`；

2. 避免信任第三方公开域，仅指定业务必需的源；

3. JSONP接口必须校验`callback`参数（白名单/正则匹配）；

4. 使用**动态随机nonce**（每次请求生成）或**脚本哈希**替代固定nonce。
> （注：文档部分内容可能由 AI 生成）