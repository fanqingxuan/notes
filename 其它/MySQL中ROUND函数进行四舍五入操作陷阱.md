### 其它
### 问题

sql语句中使用内置的round函数可能不会得到预期结果，案例如下:
- 创建测试表
表中字段有不同类型的数据类型，方便测试不同数据类型的现象
```sql
CREATE TABLE test(
	id INT PRIMARY KEY AUTO_INCREMENT,
	amount_1 FLOAT DEFAULT NULL,
	amount_2 DOUBLE DEFAULT NULL,
	amount_3 VARCHAR(10) DEFAULT NULL,
	amount_4 VARCHAR(10) DEFAULT NULL,
	amount_5 DECIMAL(20,4) DEFAULT NULL
);
```
- 写入测试数据
```sql
INSERT INTO test VALUES (NULL,3.245,3.245,'3.245','3.245',3.245);
INSERT INTO test VALUES (NULL,3.265,3.265,'3.265','3.265',3.265);
INSERT INTO test VALUES (NULL,3.5,3.5,'3.5','3.5',3.5);
```
- 查看数据
```sql
select * from test;
```

| id  |  amount_1 |amount_2|amount_3|amount_4|amount_5|
| ------------ | ------------ | ------------ | ------------ | ------------ | ------------ |
|  1 |    3.245 |    3.245 | 3.245    | 3.245    |   3.2450 |
|  2 |    3.265 |    3.265 | 3.265    | 3.265    |   3.2650 |
|  3 |      3.5 |      3.5 | 3.5      | 3.5      |   3.5000 |
- 使用round函数，查看数据
```sql
SELECT ROUND(amount_1,2),ROUND(amount_2,2),ROUND(amount_3,2),ROUND(amount_4,2),ROUND(amount_5,2) FROM test;
```

| ROUND(amount_1,2) | ROUND(amount_2,2) | ROUND(amount_3,2) | ROUND(amount_4,2) | ROUND(amount_5,2) |
| ------------ | ------------ | ------------ | ------------ | ------------ | ------------ |
|              3.24 |              3.24 |              3.24 |              3.24 |              3.25 |
|              3.27 |              3.26 |              3.26 |              3.26 |              3.27 |
|              3.50 |              3.50 |              3.50 |              3.50 |              3.50 |

我们可以很明显的看到，使用round后，并不能得到预期的结果，比如3.245保留两位小数后，有的结果是3.24，有的结果是3.25，但是我们预期的结果应该是3.25，这是什么原因呢？

### 官方解释
mysql官方文档中关于ROUND函数的部分，其中包含下面两条规则

- For exact-value numbers, ROUND() uses the “round half up” rule**（对于精确的数值， ROUND 函数使用四舍五入）**
- For approximate-value numbers, the result depends on the C library. On many systems, this means that ROUND() uses the “round to nearest even” rule: A value with any fractional part is rounded to the nearest even integer **(对于近似值，则依赖于底层的C函数库，在很多系统中 ROUND 函数会使用“取最近的偶数”的规则）**

根据这个解释，float、double类型是近似值，所以round保留小数位后，末尾应该是偶数，但是从上面的实际结果来看，float类型的3.265round之后是3.27，推翻了这个解释，于是我也不知道怎么解释了

decimal类型，mysql当做精确的数据，所以始终按照`四舍五入`保留小数位处理

### 解决方式
我们预期的结果是`四舍五入`，那有没有什么方式把float、varchar、char、double类型当做decimal类型呢？答案是肯定的，我们自己可以通过网络寻找答案，会好几种方式进行类型转换。
- CAST函数
```sql
SELECT ROUND(CAST(amount_1 AS DECIMAL(20,4)),2),ROUND(CAST(amount_2 AS DECIMAL(20,4)),2),ROUND(CAST(amount_3 AS DECIMAL(20,4)),2),ROUND(CAST(amount_4 AS DECIMAL(20,4)),2),ROUND(CAST(amount_5 AS DECIMAL(20,4)),2) FROM test;
```
- CONVERT函数
```sql
SELECT ROUND(CONVERT(amount_1 , DECIMAL(20,4)),2),ROUND(CONVERT(amount_2 , DECIMAL(20,4)),2),ROUND(CONVERT(amount_3 , DECIMAL(20,4)),2),ROUND(CONVERT(amount_4 , DECIMAL(20,4)),2),ROUND(CONVERT(amount_5 , DECIMAL(20,4)),2) FROM test;
```
- 如果业务逻辑允许，可以修改表的字段类型为decimal类型，这样代码层面不需要做任何修改

### 进阶
解决方式有了，但是实际项目中，并不一定是刚开始就发现了这个坑，可能项目中已经有大量的地方使用了round函数，另一方面代码修改的话，到处都会有`convert(field,DECIMAL(20,4))`或者`cast(field as DECIMAL(20,4))`这样重复的代码,修改起来的话也更容易出错。有没有更好的方式呢？答案是肯定的。

mysql为我们提供了可以创建自定义函数的通道，所以我们可以通过创建**自定义函数**(如何创建自定义函数，大家可以自行查阅资料)来替换mysql内置的round函数，然后代码批量替换round函数来达到我们的目的。
那上面的例子举例，我们创建自定义my_round函数
```sql
DELIMITER $$
CREATE FUNCTION MY_ROUND(X VARCHAR(30),D INT)
RETURNS VARCHAR(30) DETERMINISTIC
BEGIN
 RETURN(ROUND(CAST(X AS DECIMAL(20,4)),D));
END$$
DELIMITER ;
```
代码层面批量替换之后，sql语句为
```sql
SELECT MY_ROUND(amount_1,2),MY_ROUND(amount_2,2),MY_ROUND(amount_3,2),MY_ROUND(amount_4,2),MY_ROUND(amount_5,2) FROM test;
```
只需要将`round`函数的关键字批量替换为`MY_ROUND`。

| MY_ROUND(amount_1,2) | MY_ROUND(amount_2,2) | MY_ROUND(amount_3,2) | MY_ROUND(amount_4,2) | MY_ROUND(amount_5,2) |
| ------------ | ------------ | ------------ | ------------ | ------------ | ------------ |
| 3.25                 | 3.25                 | 3.25                 | 3.25                 | 3.25                 |
| 3.27                 | 3.27                 | 3.27                 | 3.27                 | 3.27                 |
| 3.50                 | 3.50                 | 3.50                 | 3.50                 | 3.50                 |

以上就是所有内容了。