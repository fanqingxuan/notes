### PHP
### intval或者int整形转换

工作中遇到过后端跟前端交互时，前端传递了"null"或者"undefined"字符串，后端做了强制转整形，(int)参数,
结果(int)"undefined"值是0，很是惊讶，第一感觉应该是1才对啊，于是查了手册，才知道官方的解释：

- 如果参数是字符串，则返回字符串中第一个不是数字的字符之前的数字串所代表的整数值。
- 如果字符串第一个是‘-'，则从第二个开始算起。
- 如果参数是符点数，则返回他取整之后的值。
- 当然intval()返回的值在一个4字节所能表示的范围之内（-2147483648~2147483647），对于超过这个范围的值将用边界值代替

### 浮点数精度问题
```php
var_dump((0.1 + 0.7) * 10 === 0.8 * 10);//boolean false

var_dump((0.1 + 0.7) * 100 === 0.8 * 100);//boolean true

echo intval(0.58 * 100);//57
```
因为浮点数在底层存储的是一个近似值，并不是确切的值，所以运算结果也是一个近似值

**浮点数不要参与比较**

### 慎用用strpos或者array_search做if判断

- strpos是查找字符串首次出现的位置，如果没有查询到返回布尔值false，注意如果子字符串在开头，返回位置0
```php
var_dump(strpos("hello","h"));//返回0
```
- array_search函数在数组中搜索某个键值，并返回对应的键名,注意如果查找的键值是第一个，返回0
```php
var_dump(array_search("aa",['aa','bb','cc']));
//返回0
```

综上，当用if判断是否存在时不能使用
```php
if(strpos(xx,xx)) {
	//do something
}

if(array_search(xx,xx)) {
	//do something
}
```
需要使用恒等式
```php
if(strpos(xx,xx) !== false) {
	//do something
}

if(array_search(xx,xx) !== false) {
	//do something
}
```

### url请求参数中的0
业务中有时候需要前端传递bool参数，通常用0或者1代替bool值，这时候需要特别注意了，url中的参数值是一个字符串，使用$_GET或者$_POST获取的是一个"0"字符串，"0"直接if判断结果是true，因此使用if判断时要特别注意
```php
//错误
if($_GET['xx']) {

}
//正确
if($_GET['xx'] == 0) {

}
```