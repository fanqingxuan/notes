### PHP
### 起因
工作中因为业务的关系，需要查询从上一个月1号到当前时间的数据，于是使用了如下代码表示查询时间的起始时间
```php
date("Y-m-1",strtotime("-1 month"))
```
突然有一天测试同学在测试过程中发现数据不对，于是经过层层打断点发现，起始时间不是上一个月的1号，而是这一个月的一号，好奇怪，然后经过搜索资料，发现了亚一程鸟哥博客上的解释

### 正文
如果今天是2018-07-31 ,执行代码
```php
date("Y-m-d",strtotime("-1 month"))
```
输出的结果是2018-07-01

鸟哥给出的内部逻辑如下:

 - 先做-1 month, 那么当前是07-31, 减去一以后就是06-31.
 
- 再做日期规范化, 因为6月没有31号, 所以就好像2点60等于3点一样, 6月31就等于了7月1

可以通过如下几个测试验证确实如鸟哥所说
```php
var_dump(date("Y-m-d", strtotime("-1 month", strtotime("2017-03-31"))));
//输出2017-03-03
var_dump(date("Y-m-d", strtotime("+1 month", strtotime("2017-08-31"))));
//输出2017-10-01
var_dump(date("Y-m-d", strtotime("next month", strtotime("2017-01-31"))));
//输出2017-03-03
var_dump(date("Y-m-d", strtotime("last month", strtotime("2017-03-31"))));
//输出2017-03-03

```
#### 规避方式
从PHP5.3开始, date新增了一系列修正短语, 来明确这个问题, 那就是"first day of" 和 "last day of", 也就是你可以限定好不要让date自动"规范化"

```php
var_dump(date("Y-m-d", strtotime("last day of -1 month", strtotime("2017-03-31"))));
//输出2017-02-28
var_dump(date("Y-m-d", strtotime("first day of +1 month", strtotime("2017-08-31"))));
//输出2017-09-01
var_dump(date("Y-m-d", strtotime("first day of next month", strtotime("2017-01-31"))));
//输出2017-02-01
var_dump(date("Y-m-d", strtotime("last day of last month", strtotime("2017-03-31"))));
//输出2017-02-28
```