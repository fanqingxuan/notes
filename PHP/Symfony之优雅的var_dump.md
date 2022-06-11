### PHP
大家对var_dump应该不陌生了，用于输出，一般都是调试的时候会用到，今天为大家介绍的是symfony组件VarDumper 。VarDumper是一个简单的，类似于 var_dump 的调试工具，输出界面优美，支持结构的折叠，可以用来替代 var_dump。

### 预览图
![1.jpg](http://www.fxjson.com/usr/uploads/2020/08/2840530327.jpg)

### 安装
```php
composer require symfony/var-dumper
```

### 使用
```php
<?php

require 'vendor/autoload.php';

$arr = [
    ['name'=>'范晓杰','age'=>31],
    ['name'=>'Json','age'=>30]
];

dump($arr,"12345");

class Test {
    
    private $name;
    private $age;
    private $address;
    
    public function __construct($name) {
        $this->name = $name;
    }
    
    public function say() {
        echo $this->name;
    }
    
}

$obj = new Test("test");
dd($obj);
dump("hello world");
```

### 介绍
上面的例子其实已经告诉大家怎么使用了，只需要两步
- 引入autoload.php文件
- dump或者dd函数直接使用就好，用法跟var_dump一样

值得注意的是,**dump函数不会终止程序，dd会终止下面的代码运行**
```php
<?php
require 'vendor/autoload.php';

$arr = [11,22,33,null,55];
dd($arr);//等价于dump($arr);exit(1);
echo "ok";

```

### 其它
VarDumper支持实例对象的输出
```php
<?php

require 'vendor/autoload.php';

class Test {
    
    public $name;
    public $age;
    public $db;
    public function say() {
        return "hello world";
    }
}

$test = new Test;
dump($test);

```