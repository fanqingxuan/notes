### PHP
- 当key是数字时，array_merge不会覆盖、去重元素，键名以 0 开始进行重新索引
```
$arr = array(1,2,"5"=>9);
$arr1 = array(1,8,9);
var_dump("<pre>",array_merge($arr,$arr1)); //array(1,2,9,1,8,9)
```

- 当key是字符串时，array_merge对于相同的key后者会覆盖前者，没有重复的key会按顺序进行拼接
```
$arr = array("one"=>1,"two"=>2,"three"=>3);
$arr1 = array("one"=>11,"four"=>44);
var_dump("<pre>",array_merge($arr,$arr1));// array("one"=>11,"two"=>2,"three"=>3,"four"=>44);
```
- 两个数组直接相加，对于相同的key会保留前者，对于没有重复的key(key不分是数字还是字符串)会按顺序进行拼接
```
$arr = array("one"=>1,"two"=>2,"three"=>3);
$arr1 = array("one"=>11,"four"=>44);
var_dump("<pre>",$arr+$arr1);//array("one"=>1,"two"=>2,"three"=>3,"four"=>44);
```