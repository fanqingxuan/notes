### PHP
直接上代码
```php
$list = array(
    array('id'=>1,'amount'=>1),
    array('id'=>2,'amount'=>2),
    array('id'=>3,'amount'=>30),
);
foreach($list as &$item) {
}

foreach($list as $item) {
    print_r($item);
}
```
我们期望的结果肯定是
```php
Array
(
    [id] => 1
    [amount] => 1
)
Array
(
    [id] => 2
    [amount] => 2
)
Array
(
    [id] => 3
    [amount] => 30
)

```
实际结果却是
```php
Array
(
    [id] => 1
    [amount] => 1
)
Array
(
    [id] => 2
    [amount] => 2
)
Array
(
    [id] => 2
    [amount] => 2
)

```
原因是循环使用了引用变量&$item
在第一次foreach循环中:
- 第一次循环 $item 是 $list[0] 的引用；
- 第二次循环 $item 是 $list[1] 的应用；
- 第三次循环 $item 是 $list[2] 的引用；
- 循环结束$item此时还在引用$list[2]

在第二次foreach循环中仍然使用了变量$item,循环会修改引用的变量的值
- 第一次循环结束 $item指向的元素变成了$list[0],也就是$list[2]改变成了$list[0]
- 第二次循环接受 $item指向的元素变成了$list[1],也就是$list[2]改变成了$list[1]
- 第三次取$list[2]跟$list[1]值一样(list的倒数第二个)

所以尽量不要在foreach循环上使用引用变量,如果必须要，则解决这个问题的方案有两种
- 在第一次foreach完毕后及时unset($item)取消item对别的元素的引用
- 别的循环上不要再使用引用变量，即不要再使用$item