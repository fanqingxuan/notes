### PHP
- array_key_exists() 会检查键值的存在. 这个函数会返回TRUE，只要键值存在，即使值为NULL
- isset()会同时检查键和值，只有当健存在，且对应的值不为NULL的时候才会返回TURE

```php
$arr = array(
    "one"   => 1,
    "two"   =>  null,
);
var_dump(array_key_exists("one",$arr),array_key_exists("two",$arr),isset($arr["one"]),isset($arr['two']));

//bool(true) bool(true) bool(true) bool(false)
```