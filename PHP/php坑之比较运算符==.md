### PHP
- 之前碰到过下面的细节坑
```php
var_dump("true" == 0);//返回true
var_dump("123abc" == 123);//返回true
```
这个很好解释，就是当用int类型跟字符串进行比较时，底层会阴影做转换，将字符串转成int类型进行比较，字符串转int类型的规则是:**取字符串前列的数字直到非数字的位置**，然后转成整数。
所以上面的例子"true"字符串第一个就是t，那就用0；"123abc"前面的数字的123，转成int类型123，也就成了123 == 123，返回true

- 今天在工作中又发现一个没遇到过的基础问题:
```php
var_dump("0000001" == "01");//返回true
```
太神奇了，怎么可能是true呢，两侧都是字符比较啊，没办法只能找官方手册，
官方在**[运算符](https://www.php.net/manual/zh/language.operators.comparison.php)**一节有这样的解释,如下:**当两个操作对象都是数字字符串， 或一个是数字另一个是数字字符串， 就会自动按照数值进行比较。 此规则也适用于 switch 语句。 当比较时用的是 === 或 !==， 则不会进行类型转换——因为不仅要对比数值，还要对比类型**。
根据这个解释，我们可以看到`"0000001" == "01"`两侧都是数字型字符串，所以按照数值比较，也就是左侧是数值1，右侧也是数值1，所以就相等了。

**注意:**这个问题在php8之前会有

