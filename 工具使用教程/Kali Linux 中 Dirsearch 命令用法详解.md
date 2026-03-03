# Kali Linux 中 Dirsearch 命令用法详解

你想要掌握在Kali Linux中使用dirsearch工具进行Web目录扫描的命令用法，包括基础使用、常用参数和实际场景下的示例，我会一步步帮你梳理清楚。

### 一、dirsearch 基础介绍

dirsearch是Kali Linux中一款轻量级、高效的Web目录/文件扫描工具（基于Python开发），核心作用是探测目标网站的隐藏目录、敏感文件（如配置文件、备份文件、后台入口等），是渗透测试「信息收集阶段」的核心工具。Kali Linux默认预装了dirsearch，无需手动安装。

### 二、核心命令结构

dirsearch的基础命令格式非常简洁，核心是指定目标URL，再搭配可选参数：

```Bash

dirsearch -u <目标URL> [可选参数]
```

### 三、若未预装，手动安装方法

如果你的Kali中没有dirsearch，执行以下命令快速安装：

```Bash

# 1. 克隆官方仓库
git clone https://github.com/maurosoria/dirsearch.git
# 2. 进入工具目录
cd dirsearch
# 3. 安装依赖（Python3环境）
pip3 install -r requirements.txt
# 4. 也可直接用系统全局命令（安装后）：dirsearch
```

### 四、常用参数详解（新手必记）

|参数|作用|示例|
|---|---|---|
|`-u/--url`|指定扫描目标（必选）|`-u https://www.example.com`|
|`-e/--extensions`|指定扫描的文件扩展名（多值用逗号分隔）|`-e php,txt,zip,html`|
|`-w/--wordlist`|自定义字典文件路径|`-w /usr/share/wordlists/dirb/common.txt`|
|`-r/--recursive`|递归扫描发现的目录（深入子目录）|`-r`|
|`-t/--threads`|设置扫描线程数（默认10，建议10-30）|`-t 20`|
|`-i/--include-status`|只显示指定状态码的结果（如200=成功、301=重定向）|`-i 200,301`|
|`-x/--exclude-status`|排除指定状态码（如404=不存在、403=无权限）|`-x 404,403`|
|`-o/--output`|将扫描结果保存到文件|`-o /root/scan_result.txt`|
|`-q/--quiet`|静默模式，只显示有效结果（减少冗余）|`-q`|
|`--random-agent`|使用随机User-Agent（规避目标拦截）|`--random-agent`|
|`--proxy`|通过代理扫描（如Burp抓包）|`--proxy http://127.0.0.1:8080`|
### 五、实战示例（新手直接复制用）

#### 示例1：基础扫描（最常用）

扫描目标的常见目录/文件，默认扫描php、html等扩展名：

```Bash

dirsearch -u https://www.example.com
```

#### 示例2：指定扩展名扫描（精准定位）

只扫描目标的php、txt、sql后缀文件（适合找配置/备份文件）：

```Bash

dirsearch -u https://www.example.com -e php,txt,sql
```

#### 示例3：递归扫描+只显示有效结果

递归扫描所有子目录，仅输出状态码200的有效结果（静默模式）：

```Bash

dirsearch -u https://www.example.com -r -i 200 -q
```

#### 示例4：自定义字典+多线程

用dirb的经典字典，20线程扫描（提升效率，避免线程过高被封）：

```Bash

dirsearch -u https://www.example.com -w /usr/share/wordlists/dirb/common.txt -t 20
```

#### 示例5：扫描结果保存到文件

将扫描结果保存到本地，方便后续分析：

```Bash

dirsearch -u https://www.example.com -e php -o /root/example_scan.txt
```

#### 示例6：通过代理扫描（隐藏IP/抓包）

搭配Burp Suite代理扫描（本地8080端口），便于抓包分析或规避IP封禁：

```Bash

dirsearch -u https://www.example.com --proxy http://127.0.0.1:8080 --random-agent
```

### 六、重要注意事项

1. 线程数不要设置过大（建议≤30），否则容易被目标的WAF/防火墙拦截，导致扫描失败；

2. 仅可对**合法授权**的目标进行扫描，未经授权扫描他人网站可能违反法律法规；

3. 若扫描无结果，可尝试`--random-agent`或更换字典（Kali内置字典路径：`/usr/share/wordlists/`）。

### 总结

1. dirsearch核心用法是`dirsearch -u <目标URL> [参数]`，`-u`是必选参数，`-e`是最常用的可选参数；

2. 新手优先掌握：`-e`（指定扩展名）、`-r`（递归扫描）、`-i`（筛选有效结果）、`-o`（保存结果）；

3. 扫描时注意控制线程数、使用随机User-Agent，避免被目标拦截，且务必遵守法律规范。
> （注：文档部分内容可能由 AI 生成）