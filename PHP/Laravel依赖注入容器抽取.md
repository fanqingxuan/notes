### PHP
### 起因
Laravel是一个很优雅的框架，这一点不容置疑，laravel实现的很松耦合，有些组件库可以在自己的非laravel项目里用(只要你的项目使用composer)，比如illuminate/database这个orm，就是一个独立的repository，我们composer执行下面的命令就可以将laravel的数据库orm安装到自己的项目了
```
composer require illuminate/database
```
个人对Laravel的服务容器很感兴趣，想单独拿出来用，高兴的是官方也将Container容器单独抽离出一个仓库illuminate/container，我看了下这个仓库下的composer.json文件，依赖illuminate/contracts仓库，通过阅读Container类我发现其实只依赖illuminate/contracts下的container约定，其它的约定完全不需要，也就是说illuminate/contracts仓库下面的其它契约成了多余的代码，这对我来说完全受不了，容忍不了多余没有用的代码。

因此我就单独拉了一个仓库，代码完全来自illuminate/container仓库以及illuminate/contracts仓库下Illuminate/Contracts/Container命名空间下的代码，当然为了使用composer，所以我需要改下仓库，因此命名空间变更成了自己的命名空间，其它Laravel服务容器原有相关的方法100%可用；另外有些原有的功能在手册或者教程上并没有，通过阅读代码发现了已有的功能，也罗列出了用法

### 安装
```shell
composer require fanqingxuan/container
```
### demo
看下面的例子
```php
require_once './vendor/autoload.php';

use JsonContainer\Container;

$container = new Container;

class HomeController {

    private $userService;

    public function __construct(UserService $userService) {
        $this->userService = $userService;
    }

    public function say() {
        return $this->userService->getUser();
    }
}

interface UserService {

    public function getUser();
}

class UserServiceImpl implements UserService {

    private $userDao;

    public function __construct(UserDao $userDao) {
        $this->userDao = $userDao;
    }

    public function getUser() {
        return $this->userDao->getUser();
    }
}

class UserDao {

    public function getUser() {
        return "this is userdao/getuser";
    }
}

$container->bind(HomeController::class,HomeController::class);
$container->bind(UserService::class,UserServiceImpl::class);
$container->bind(UserDao::class,UserDao::class);

$homeController = $container->make(HomeController::class)->say();
print($homeController);
```
有没有发现，每一个类不再依赖具体的实现类，通过依赖注入容器实现控制反转

### 使用

- bind简单绑定

bind方法的第一个参数为要绑定的类 / 接口名，第二个参数是一个Closure

```php
require_once './vendor/autoload.php';

use JsonContainer\Container;

$container = new Container;

class Test {

}

$container->bind("test",function() {
    return new Test;//返回实例对象
});

$container->bind("hello",function() {
    return "hello world";//返回字符串
});
```

- singleton单例绑定

singleton方法将类或接口只解析一次绑定到容器中。一旦单例绑定被解析，相同的对象实例会在随后的调用中返回到容器中

```php
require_once './vendor/autoload.php';

use JsonContainer\Container;

$container = new Container;

class Test {

}

$container->singleton("test",function() {
    return new Test;
});
```
- instance实例绑定

绑定实例,将现有对象实例绑定到容器中。给定的实例会始终在随后的调用中返回到容器中

```php
require_once './vendor/autoload.php';

use JsonContainer\Container;

$container = new Container;

class Test {

}

$container->instance("test",new Test);
```
- 基本值绑定

当你有一个类不仅需要接受一个注入类，还需要注入一个基本值（比如整数）。你可以使用上下文绑定来轻松注入你的类需要的任何值

```php
require_once './vendor/autoload.php';

use JsonContainer\Container;

$container = new Container;

class HomeController {

    private $id;

    public function __construct($id) {
        $this->userService = $userService;
        $this->id = $id;
    }

    public function say() {
        return $this->id;
    }
}

$value = 32;
$container->when(HomeController::class)
          ->needs('$id')
          ->give($value);
```
- 绑定接口到实现

Laravel的服务容器有一个很强大的功能，就是支持绑定接口到给定的实现，这样的好处是某一天你可能有另一种实现，只需要修改绑定接口的实现代码就可用了，而业务代码不需要任何改动。例如，如果有个Cache接口 和一个 RedisCache实现。一旦我们写完了 Cache 接口的 RedisCache 实现，我们就可以在服务容器中注册它，像这样：

```php
$this->app->bind(
    Cache::class,
    RedisCache::class
);
```
意思是当一个类需要实现 Cache 时，应该注入 RedisCache,注入依赖项时，我们就可以使用注入Cache接口，而不需要注入具体实现，就像上面demo中的注入依赖项UserService接口，而不是具体实现

```php
class HomeController {

    private $userService;

    public function __construct(UserService $userService) {
        $this->userService = $userService;
    }

    public function say() {
        return $this->userService->getUser();
    }
}

interface UserService {

    public function getUser();
}

class UserServiceImpl implements UserService {

    private $userDao;

    public function __construct(UserDao $userDao) {
        $this->userDao = $userDao;
    }

    public function getUser() {
        return $this->userDao->getUser();
    }
}
```
- 按需绑定

有时你可能有两个类使用了相同的接口，但你希望各自注入不同的实现。比如一个service可能希望memcache缓存，另一个service希望redis缓存

```php
require_once './vendor/autoload.php';

use JsonContainer\Container;

$container = new Container;


class Cache {

}

class Memcache extends Cache {

}

class Redis extends Cache {

}

class UserService {

    private $cache;

    public function __construct(Cache $cache) {
        $this->cache = $cache;
    }

    public function say() {
        print_r($this->cache);
    }
}

class OrderService {

    private $cache;

    public function __construct(Cache $cache) {
        $this->cache = $cache;
    }

    public function say() {
        print_r($this->cache);
    }
}

$container->when(UserService::class)
          ->needs(Cache::class)
          ->give(function () {
              return new Memcache;
          });

$container->when([OrderService::class, ImageService::class])
          ->needs(Cache::class)
          ->give(function () {
              return new Redis;
          });

print_r($container->make(UserService::class)->say());
print_r($container->make(OrderService::class)->say());
```
根据输出可以反省，UserService注入的是Memcache，而OrderService注入的是Redis

- 标记

有时候，你可能需要为容器中的服务分类，比如memcache，redis都属于cache，你可以使用 tag 方法给他们分配一个标签,一旦服务被打上标签，你就可以通过 tagged 方法轻松获取标签下的服务

```php
class Cache {

}

class Memcache extends Cache {

}

class Redis extends Cache {

}

$container->bind("memcache",function() {
    return new Memcache;
});

$container->bind("redis",function() {
    return new Redis;
});

$container->tag(['memcache','redis'],'cache');

$cachelist = $container->tagged('cache');

foreach ($cachelist as $cache) {
    print_r($cache);
}
```
- 扩展服务

extend 方法可以修改已注入到容器中的服务。比如，当一个服务被注入后，你可以添加额外的代码来修饰或者配置它。 extend 方法接受一个闭包，该闭包唯一的参数就是这个服务， 并返回修改过的服务

```php
class Cache {

}

class Memcache extends Cache {

}

class Redis extends Cache {

}

$container->bind("cache",function() {
    return new Memcache;
});

$container->extend("cache",function($serivce) {
    //$service是Memcache对象
    return new Redis;
});

print_r($container->make("cache"));
```
### 解析服务

使用 make 方法从容器中解析出类实例。 make 方法接收你想要解析的类或接口的名字

```php
class Cache {

}

class Memcache extends Cache {

}

$container->bind(Memcache::class,Memcache::class);

print_r($container->make(Memcache::class));
```
如果类依赖不能通过容器解析，你可以通过将它们作为关联数组作为 makeWith
、make 方法的参数注入：

```php
class Test {
    
}

class Cache {

    private $test;
    private $id;

    public function __construct(Test $test,$id) {
        $this->test = $test;
        $this->id = $id;
    }

    public function get() {
        return $this->id;
    }

}

$container->bind(Cache::class,Cache::class);

print_r($container->makeWith(Cache::class,['id'=>33])->get());
print_r($container->make(Cache::class,['id'=>33])->get());

```
### 容器事件

服务容器每次解析对象会触发一个事件，你可以使用 resolving 方法监听这个事件 :

```php
require_once './vendor/autoload.php';

use JsonContainer\Container;

$container = new Container;


class Test {
    
}

class Cache {

    private $test;

    public function __construct(Test $test) {
        $this->test = $test;
    }

}

$container->bind("test",Cache::class);
//解析所有服务都会触发这个事件
$container->resolving(function ($object, $container) {
    print_r($object);
    print_r($container);
});
//当解析cache时会触发这个事件
$container->resolving("cache", function ($object, $container) {
   print_r($object);
});

$container->make("test");

```
### PSR-11

Laravel服务容器实现了PSR-11接口，所以可以使用PSR-11接口提供的方法来获取服务

```php
class Test {
    
}

class Cache {

    private $test;

    public function __construct(Test $test) {
        $this->test = $test;
    }

}

$container->bind("cache",Cache::class);

print_r($container->get("cache"));
print_r($container->has('cache'));
```
### ArrayAccess

Laravel服务容器实现了ArrayAccess接口，因此可以像数组一样为容器注入服务，获取服务等操作

```php

class Cache {

    public function __construct() {
    }

}

$container->bind("cache",Cache::class);

print_r($container['cache']);
unset($container['cache']);
```
### 魔术方法

Laravel服务容器还实现了__get,__set魔术方法，因此我们可以用属性的方式进行服务注入和访问

```php
class Test {
    private $id;
    public function __construct($id) {
        $this->id = $id;
    }
}

class Cache {
    public function __construct(Test $test) {
    }
}

$container->bind("cache",Cache::class);
$container->Test = function () {
    return new Test(333);
};
var_dump($container->cache);
```
### Laravel服务容器的其它方法

- 获取容器单例

```php
$container = Container::getInstance();
```
- 清除容器中所有服务
```php
$container->flush();
```
- 检查服务是否存在
```php
$container->has("cache");
isset($container['cache'])
```
- 服务是否单例
```php
$container->isShared("cache")
```