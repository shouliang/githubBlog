title: SQL注入
date: 2017-02-23 14:45:23
tags: [安全]
---
###  如何理解SQL注入（攻击）？
1. SQL注入是一种将SQL代码添加到输入参数中，传递到服务器解析并执行的一种攻击手法。
2. SQL注入攻击是输入参数未经过滤，然后直接拼接到SQL语句当中解析，执行达到预想之外的一种行为，称之为SQL注入攻击。

### SQL注入是怎么产生的？
1. WEB开发人员无法保证所有的输入都已经过滤
2. 攻击者利用发送给SQL服务器的输入参数构造可执行的SQL代码（可加入到get请求、post请求、http头信息、cookie中）
3. 数据库未做相应的安全配置

### 如何进行SQL注入攻击

要想发动sql注入攻击，就要知道正在使用的系统数据库，不然就没法提取重要的数据。
首先从Web应用技术上就给我们提供了判断的线索：
- ASP和.NET：Microsoft SQL Server
- PHP：MySQL、PostgreSQL
- Java：Oracle、MySQL

Web容器也给我们提供了线索，比如安装IIS作为服务器平台，后台数据及很有可能是Microsoft SQL Server，而允许Apache和PHP的Linux服务器就很有可能使用开源的数据库，比如MySQL和PostgreSQL。

##### 基于错误识别数据库
大多数情况下，要了解后台是什么数据库，只需要看一条详细的错误信息即可。比如判断我们事例中使用的数据库，我们加个单引号。
```sql
error:You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''' at line 1
```
从错误信息中，我们就可以发现是MySQL。

```sql
Microsoft OLE DB Provider for ODBC Drivers 错误 '80040e14'
 [Microsoft][ODBC SQL Server Driver][SQL Server]Line 1:
```
上面错误信息可以发现是Microsoft SQL Server，如果错误信息开头是ORA，就可以判断数据库是Oracle，很简单，道理都是一样的，就不一一列举了。

###### UINON语句提取数据
UNION操作符可以合并两条或多条SELECT语句的查询结果，基本语法如下：
```sql
select column-1 column-2 from table-1
UNION
select column-1 column-2 from table-2
```
###### 枚举数据库
我们只以MySQL数据库为例了，枚举数据库并提取数据遵循一种层次化的方法，首先我们提取数据库名称，然后提取表，再到列，最后才是数据本身。要想获取远程数据库的表、列，就要访问专门保存描述各种数据库结构的表。通常将这些结构描述信息成为元数据。在MySQL中，这些表都保存在information_schema数据库中
###### 第一步：提取数据库
在MySQL中，数据库名存放在information_schema数据库下schemata表schema_name字段中
```sql
id=1 union select null,schema_name,null from information_schema.schemata
```
######  第二步：提取表名
在MySQL中，表名存放在information_schema数据库下tables表table_name字段中
```sql
?id=1 union select null,table_name,null from information_schema.tables where table_schema='ichunqiu'
```

######  第三步：提取字段名
在MySQL中，字段名存放在information_schema数据库下columns表column_name字段中
```sql
select null,column_name,null from information_schema.columns where table_name='users' and table_schema=''
```

###### 第四步：提取数据
```sql
select username,password,null from users
```
### SQL盲注
#### 布尔型注入
例如：在参数后面加上or 1=1，可返回所有数据，因为 or 1=1永远为真

#### 联合查询注入
在参数后面加上：
```sql
UNION select column-1 column-2 from table-2
```
#### 多语句注入
在参数后面加上：
```sql
;drop table a; select * from tableb;
```
#### 字符串注册
'#'：'#'后所有的字符串都会被当成注释来处理用户名输入：user'#（单引号闭合user左边的单引号），密码随意输入，如：111，然后点击提交按钮。等价于SQL语句：
```sql
SELECT * FROM user WHERE username = 'user'#'ADN password = '111'
```

### 如何预防SQL注入
1. 严格检查输入变量的类型和格式
    对于整数参数，加判断条件：不能为空、参数类型必须为数字
    对于字符串参数，可以使用正则表达式进行过滤：如：必须为[0-9a-zA-Z]范围内的字符串
2. 对URL进行编码
3. 对进入数据库的特殊字符（'"\尖括号&*;等）进行转义处理，或编码转换
4. 所有的查询语句建议使用数据库提供的参数化查询接口，参数化的语句使用参数而不是将用户输入变量嵌入到SQL语句中，即不要直接拼接SQL语句
5. 避免网站打印出SQL错误信息，比如类型错误、字段不匹配等，把代码里的SQL语句暴露出来，以防止攻击者利用这些错误信息进行SQL注入。
6. 利用sql的预编译机制
    把sql语句的模板（变量采用占位符进行占位）发送给数据库服务器，数据库服务器对sql语句的模板进行编译，编译之后根据语句的优化分析对相应的索引进行优化，在最终绑定参数时把相应的参数传送给数据库服务器，直接进行执行，节省了sql查询时间，以及数据库服务器的资源，达到一次编译、多次执行的目的，除此之外，还可以防止SQL注入。具体是怎样防止SQL注入的呢？实际上当将绑定的参数传到数据库服务器，服务器对参数进行编译，即填充到相应的占位符的过程中，做了转义操作。

### 参考
https://blog.csdn.net/github_36032947/article/details/78442189
https://www.jianshu.com/p/ba35a7e1c67d
https://paper.seebug.org/15/
https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/09.4.md


