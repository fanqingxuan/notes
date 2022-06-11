### PHP
Pimple是一个简单的php版本依赖注入容器，官网地址是[https://pimple.symfony.com/](https://pimple.symfony.com/)，当前pimple最新版本是3.0，我们用3.0进行使用讲解
### 安装
- composer
```shell
  composer require pimple/pimple ~3.0
```
- c extension
Pimple容器也提供了c扩展形式的安装方式，性能应该比php原生的要好
```shell
$ git clone https://github.com/silexphp/Pimple
$ cd Pimple/ext/pimple
$ phpize
$ ./configure
$ make
$ make install
```

### 使用
- 创建容器实例

	```shell
	<?php

	require_once(__DIR__ . '/vendor/autoload.php');

	use Pimple\Container;

	$container = new Container();
	```
- 容器注入服务
	服务是一个对象实例，作为庞大系统的一部分,比如数据连接、缓存中间件、模板引擎、邮件等都可以成为一项服务。
	Pimple中通过匿名函数返回实例对象的方式来注册服务，如下代码:

	```php
	$container['test'] = function() {
		return new StdClass;
	};

	class Cache {
	}

	$container['cache'] = function() {
		return new Cache;
	};
	```
	请注意，匿名函数中可以访问当前容器实例，允许引用其他服务作为参数，由于只有在获取对象时才创建对象，因此定义的顺序并不重要。例子代码如下:

	```php
	class Cache {
		private $test;
		public function __construct($test) {
			$this->test = $test;
		}
	}
	//使用container容器作为参数，获取容器中的test服务传递给Cache类的构造函数
	$container['cache'] = function($container) {
		return new Cache($container['test']);
	};
	$container['test'] = function() {
		return new StdClass;
	};
	```
- 获取服务
获取服务很简单，如下

	```php
	//获取cache实例对象
	$cache = $container['cache'];
```
- 工厂服务
默认情况下，Pimple获取相同服务时，返回的是同一个实例，也就是实现的单例模式。演示代码如下:

	```php
	class Cache {
		public function __construct() {
			echo '实例化了一次';
		}
	}

	$container['cache'] = function() {
		return new Cache();
	};

	$container['cache'];
	$container['cache'];
	$container['cache'];
	```
	界面输出

	```shell
	实例化了一次
	```
	证明只实例化了一次，也就是同一个实例。如果想每次获取服务返回不同的实例，则需要使用factory方法包装匿名函数,代码如下:

	```php
	class Cache {
		public function __construct() {
			echo '实例化了一次';
		}
	}

	$container['cache'] = $container->factory(function() {
	  return new Cache;  
	});

	$container['cache'];
	$container['cache'];
	$container['cache'];
	```
	界面输出

	```shell
	实例化了一次实例化了一次实例化了一次
	```
	证明实例化了3次，是不同的实例

- 容器注入非对象简单数据

	```php
	$container['dbname'] = 'demo';
	$container['app_name'] = 'Json';
	```

- 修改已经注入容器的服务
修改已经注入容器的服务，可以使用extend方法

	```php
	class Cache {
		public function __construct() {
			echo '实例化了一次';
		}
	}

	$container['cache'] = function() {
	  return new Cache;  
	};

	$container->extend('cache',function($cache,$container){
		$cache->name = 'Json';
		return $cache;
	});
	```
	extend方法的第一个参数是注入容器的服务名称，第二个参数是一个匿名函数，匿名函数第一个参数是当前服务实例，匿名函数第二个参数是容器实例，注意匿名函数需要返回修改后的实例

- 服务提供者
使用过laravel开源框架的应该都知道服务提供者是什么，怎么用，Pimple跟laravel的使用方式类似,但是需要实现Pimple\ServiceProviderInterface接口

	```php
	class CacheProvider implements Pimple\ServiceProviderInterface {

		public function register(Container $container) {
			$container['cache'] = function() {
				return new Cache;
			};
		}
	}

	class Cache {
		public function __construct() {
			echo '实例化了一次';
		}
	}
	$container->register(new CacheProvider);
	```