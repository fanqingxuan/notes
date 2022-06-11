### PHP
parse_ini_file函数是php内置的一个解析ini文件的函数，解析成功时以关联数组 array 返回结果，失败时返回 FALSE

###常规用法

```php
//config.ini
one = 1
five = 5
animal = BIRD

;这是注释
list[] = 1
list[] = 2
list[] = 3

debug = false
openLock=on

```
```php
$arr = parse_ini_file('./config.ini');
print_r($arr);
//输出Array ( [one] => 1 [five] => 5 [animal] => BIRD [list] => Array ( [0] => 1 [1] => 2 [2] => 3 ) [debug] => 1 [openLock] => 1 )
```
**注意**
- 如果 ini 文件中的值包含任何非字母数字的字符，需要将其括在双引号中（"）

- 有些保留字不能作为 ini 文件中的键名，包括：null，yes，no，true 和 false。值为 null，no,off 和 false 等效于 ""，值为 yes,on, true 等效于 "1"。字符 {}|&~![()" 也不能用在键名的任何地方，而且这些字符在选项值中有着特殊的意义。

### 进一步用法

- 常量可以在 ini 文件中被解析，因此如果在运行 parse_ini_file() 之前定义了常量作为 ini 的值，将会被集成到结果中去

```php
define('BIRD','this is bird');
$arr = parse_ini_file('./config.ini');
print_r($arr);
```
可以看到config.ini的BIRD的值被解析成this is bird字符串了

- 支持第二个参数，获取多维数组

```php
//config.ini

[base]

url = 127.0.0.1
[local] 
url = localhost

[product]

url = http://www.fxjson.com
```

```php
<?php

$arr = parse_ini_file('./config.ini',true);

print_r($arr);

```
输出结果

```php
Array
(
    [base] => Array
        (
            [url] => 127.0.0.1
        )

    [local] => Array
        (
            [url] => localhost
        )

    [product] => Array
        (
            [url] => http://www.fxjson.com
        )

)
```

### 活学活用
根据上面的基本用法，我们可以用ini作为配置文件，实现配置的继承和差异化,使用过yaf的大佬应该都知道application.ini的配置继承，我们也可以实现类似yaf解析配置的效果
```php
//config.ini
[base]
host=localhost
user=root
pass=root123
database=default
debug=on

[development:base]
database=fxjson

[product : base]
database=wcshop
debug = off
```
```php
<?php

/**
 * @param string $filename
 * @return array
 */
function parse_ini_file_extended($filename) {
    $p_ini = parse_ini_file($filename, true);
    $config = array();
    foreach($p_ini as $namespace => $properties){
        list($name, $extends) = explode(':', $namespace);
        $name = trim($name);
        $extends = trim($extends);
        // create namespace if necessary
        if(!isset($config[$name])) $config[$name] = array();
        // inherit base namespace
        if(isset($p_ini[$extends])){
            foreach($p_ini[$extends] as $prop => $val)
                $config[$name][$prop] = $val;
        }
        // overwrite / set current namespace values
        foreach($properties as $prop => $val)
        $config[$name][$prop] = $val;
    }
    return $config;
}

print_r(parse_ini_file_extended('./config.ini'));
?>
```
最后得到结果
```php

Array
(
    [base] => Array
        (
            [host] => localhost
            [user] => root
            [pass] => root123
            [database] => default
            [debug] => 1
        )

    [development] => Array
        (
            [host] => localhost
            [user] => root
            [pass] => root123
            [database] => fxjson
            [debug] => 1
        )

    [product] => Array
        (
            [host] => localhost
            [user] => root
            [pass] => root123
            [database] => wcshop
            [debug] => 
        )

)
```
我们看到了，product作为我们的生产配置继承了基础设置，也可以重写修改成自己的配置

### 总结

巧用官方提供的内置函数，也能干很多有意思的事情，多查手册，怀有好奇心，一定会有收获