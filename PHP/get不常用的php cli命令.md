### PHP
之前很少用php cli命令，因为毕竟php在web领域上用的多，最近接触到的php cli多了，感觉挺有意思，现在汇总到这里，作为笔记。

### php命令

- 查看ini文件位置
```php
php --ini
```

- 查看php加载的扩展、模块
```php
php -m
```

- text方式查看phpinfo()
```php
php -i
```

- 无需<?php，运行php代码
```php
php -r "phpinfo();"
php -r "echo 1+2;"
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
php -r "unlink('composer-setup.php');"
```

### php运行管道输出
```php
curl -Ss http://www.workerman.net/check.php | php

```
上面是curl获取curl的输出结果，作为管道php命令的运行内容

下面的方式是运行本地脚本，输出其实就是一个php格式的内容
test.php内容如下:
```php
<?php
header('HTTP/1.1 206 Partial Content');
header("Content-Type: application/octet-stream");

echo '<?php
$version_ok = $pcntl_loaded = $posix_loaded = false;
if(version_compare(phpversion(), "5.3.3", ">="))
{
  $version_ok = true;
}
if(in_array("pcntl", get_loaded_extensions()))
{
    $pcntl_loaded = true;
}
if(in_array("posix", get_loaded_extensions()))
{
    $posix_loaded = true;
}

function check($val)
{
    if($val)
    {
       return "\033[32;40m [OK] \033[0m\n";
    }
    else
    {
       return "\033[31;40m [fail] \033[0m\n";
    }
}

echo "PHP Version >= 5.3.3                 " . check($version_ok);

echo "Extension pcntl check                " . check($pcntl_loaded);

echo "Extension posix check                " . check($posix_loaded);

';

```
运行方式:
```php
php test.php | php
```