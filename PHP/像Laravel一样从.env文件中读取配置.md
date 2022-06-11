### PHP
熟悉laravel的朋友应该对env函数不陌生了吧，config文件夹里面的每个文件应该都会用到，今天为大家推荐的就是laravel项目从.env文件里面读取配置的库,[vlucas/phpdotenv](https://github.com/vlucas/phpdotenv)。大家可以看到laravel/framework项目的composer.json文件有依赖这个库。大家可以在github上搜到这个库。

### 用途

从.env文件里面读取配置到getenv(), $_ENV and $_SERVER。

### 安装
```shell
composer require vlucas/phpdotenv
```

### 使用
- 创建.env文件

```shell
DB_HOST=127.0.0.1
DB_NAME=demo
DB_USER=root
DB_PASS=root
DB_PORT=3306
```

- 代码

所有的配置变量都可以通过$_ENV,$_SERVER超全局变量读取。
```php
<?php

require 'vendor/autoload.php';

$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();

var_dump($_ENV['DB_HOST'],$_SERVER['DB_HOST']);
```

### 其它
- 自定义配置文件

使用第二个参数设置从其它文件读取配置
```php
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__, '.env.local');
$dotenv->load();
```
- 使用getenv函数读取配置

要使用getenv函数读取配置，需要使用Dotenv::createUnsafeImmutable方法，如下代码

```php
<?php

require 'vendor/autoload.php';

$dotenv = Dotenv\Dotenv::createUnsafeImmutable(__DIR__);
$dotenv->load();

var_dump($_ENV['DB_HOST'],$_SERVER['DB_HOST'],getenv('DB_HOST'));
```

- 配置里面使用另一个配置变量

配置变量使用另一个配置变量的值，需要使用${}将变量包起来，如下,DB_URL使用DB_HOST、DB_USER、DB_PASS变量

```shell
DB_HOST=127.0.0.1
DB_NAME=demo
DB_USER=root
DB_PASS=root
DB_PORT=3306

DB_URL = "${DB_HOST}/${DB_USER}:${DB_PASS}"
```

- 验证变量
该库提供了对配置文件中变量的必要验证方法
 
	-  验证配置变量必填
 ```php
 $dotenv->required('DATABASE_DSN');
 ```
 
 - 验证多个变量必填
 ```php
 $dotenv->required(['DB_HOST','DB_URL1']);
 ```
 - 验证存在且不为空
 ```php
 $dotenv->required('DATABASE_DSN')->notEmpty()
 ```
 - 验证是整数
 ```php
 $dotenv->ifPresent('DB_PORT')->isInteger();
 ```

phpdotenv还有更多用法，大家可以点击[这里](https://github.com/vlucas/phpdotenv)
