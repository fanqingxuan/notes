### PHP
### 前言

之前很少使用phpstorm进行项目开发,习惯了使用vscode、sublime,后来由于项目需要,sublime、vscode有些代码不能友好的提示出来，也不能很好的自动补全，多有不方便之处，最终还是安装了phpstorm，一个偶然的尝试,发现phpstorm竟然可以自动提示phpredis、memcached、yaf等的方法，phpstorm这个ide果真强大啊，通过ctrl+鼠标左键自动追溯跳转到函数或者方法定义之处。发现phpstorm集成内置了常用扩展的原型定义，在phpstorm安装目录/plugins/php/lib/php.jar包里面，于是通过函数跳转我发现了集成的phpredis的stub原来来源于[https://github.com/ukko/phpredis-phpdoc](https://github.com/ukko/phpredis-phpdoc).

### 插件下载
- 通过composer进行安装
```php
composer require ukko/phpredis-phpdoc
```

- 直接通过[github](https://github.com/ukko/phpredis-phpdoc)下载源码

### 使用方式
比较新的phpstorm已经集成了这个插件，不需要再设置什么,如果不自动提示，可手动设置

```shell
Menu "File" -> "Settings" -> "PHP" -> Select path to folder "phpredis-phpdoc"
```

### 插件作用

可以自动提示并完成phpredis的方法,并带有方法参数说明

![redisphp.png](http://www.fxjson.com/usr/uploads/2020/07/2632389623.png)

### 举例

```php
$redis = new Redis();
$redis->con<press Tab or press Ctrl+Space>
```
### 其它

这个插件的src/Redis.php文件里每一个方法都有方法注释，以及使用的example，大大方便了我们使用。更多细节和注意事项，大家可以通过作者的[github](https://github.com/ukko/phpredis-phpdoc)进行查看
```php
/**
* Set the string value in argument as value of the key.
*
* @since If you're using Redis >= 2.6.12, you can pass extended options as explained in example
*
* @param string       $key
* @param string|mixed $value string if not used serializer
* @param int|array    $timeout [optional] Calling setex() is preferred if you want a timeout.<br>
* Since 2.6.12 it also supports different flags inside an array. Example ['NX', 'EX' => 60]<br>
*  - EX seconds -- Set the specified expire time, in seconds.<br>
*  - PX milliseconds -- Set the specified expire time, in milliseconds.<br>
*  - PX milliseconds -- Set the specified expire time, in milliseconds.<br>
*  - NX -- Only set the key if it does not already exist.<br>
*  - XX -- Only set the key if it already exist.<br>
* <pre>
* // Simple key -> value set
* $redis->set('key', 'value');
*
* // Will redirect, and actually make an SETEX call
* $redis->set('key','value', 10);
*
* // Will set the key, if it doesn't exist, with a ttl of 10 seconds
* $redis->set('key', 'value', ['nx', 'ex' => 10]);
*
* // Will set a key, if it does exist, with a ttl of 1000 milliseconds
* $redis->set('key', 'value', ['xx', 'px' => 1000]);
* </pre>
*
* @return bool TRUE if the command is successful
*
* @link     https://redis.io/commands/set
*/
public function set($key, $value, $timeout = null)
{
}
```