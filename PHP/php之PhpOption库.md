### PHP
### 前言

查看laravel框架的config目录的配置文件时，看到了env这个函数，于是根据代码提示追踪了下，无意间发现了phpoption/phpoption这个代码仓库，这个库具体有什么作用，我也说不清楚，感觉有点儿类似于空对象模式，但又有些不一样。

### 介绍

这个库在github可以搜索到，大家可以自行搜索，github介绍说这个库是一个可选项类型，什么意思呢我不是特别理解。我总结的这个库主要是用来提供默认值或者默认结构，减少冗余代码。

- 提供默认值

laravel也是采用的这种方式，主要是给config配置项提供默认值，大家应该都知道了env()函数，从.env文件里面读取配置，如果.env文件里面不存在这个变量，则取env函数的第二个值作为默认值。逻辑代码如下:

```php
/**
 * Gets the value of an config file.
 *
 * @param  string  $key
 * @param  mixed  $default
 * @return mixed
 */
function env($key,$default) {
    
   //模拟配置文件
   $configFromFile = [
        'DB_HOST'   =>  'localhost',
        'DB_USER'   =>  'json'
   ];
   
   return Option::fromValue($configFromFile[$key])->getOrCall(function() use ($default) {
        return $default; 
   });
}
$arr = [
    'DB_HOST'   =>  env("DB_HOST",'127.0.0.1'),
    'DB_USER'   =>  env('DB_USER','root'),
    'DB_PORT'   =>  env('DB_PORT',3306)
];

var_dump($arr);
```
最后结果输出

```php
array(3) {
  ["DB_HOST"]=>
  string(9) "localhost"
  ["DB_USER"]=>
  string(4) "json"
  ["DB_PORT"]=>
  int(3306)
}
```
这里我用$configFromFile模拟从.env文件里面读取的配置，laravel其实是通过vlucas/phpdotenv库读取.env文件的，之前我有文章已经介绍过这个库。我们可以看到$configFromFile里面没有DB_PORT，则读取的是默认值3306.

- 返回默认结构

相信用过laravel或者其它框架的同学都知道，一般model或者repository返回的都是实体entity，我们在业务层经常见过类似下面的代码。
```php
	$user = $userRepository->find(5);
	$data = [
		'name'   => $user?$user->name:'',
		'age'	=> $user?$user->age:'',
	];
```
我们需要判断兼容$user为空的情况。而PhpOption在这里也有适用的场景，我们看修改后的完整代码。

```php
require 'vendor/autoload.php';

use PhpOption\Option;

class UserRepository 
{
    public function find($id)
    {
        $user = null;
        //模拟查到的数据存在和不存在的情况，这里真实情况下一般是db里面查询记录结构，映射成entity
        if($id>10) {
            $user = new UserEntity($id,"Json",32);
        }
        return Option::fromValue($user);
        
    }
}

class UserEntity 
{
    public $id;
    public $name;
    public $age;
    
    public function __construct($id='', $name='', $age='') 
    {
        $this->id = $id;
        $this->name = $name;
        $this->age = $age;
    }
}
```
使用方法
```php
$userRepository = new UserRepository;

$user = $userRepository->find(8)->getOrCall(function() {
    return new UserEntity;
});

$data = [
    'name'   => $user->name,
    'age'	 => $user->age,
];
var_dump($data);
```
可以看到从不再需要$user?$user->name:""类似的兼容代码了，看上去是不是很舒服。

### 安装
```shell
composer require phpoption/phpoption
```

### 其它

上面的例子代码使用了PhpOption库中的fromValue方法和getOrCall方法，这个库还有其它一些方法，大家可以通过**[github](https://github.com/schmittjoh/php-option)**进行查看，这里就不一一介绍了。
