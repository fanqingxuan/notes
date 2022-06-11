### PHP
### 前言

用过Laravel的同学应该都见过类似如下的方法:
```php
Cache::get("user");
Route::get("/list");
DB::table("lgt_good")->where('lgid',2)->get();
```
看到没有不需要new实例，直接使用方法，是不是用起来特别方便。

###  正文

也许你认为是调用了类的静态方法，如果你是这么想的，那就大错特错了，可以通过IDE的代码追踪工具看到，每一个门面类都只有一个类似下面的代码，只有一个getFacadeAccessor()受保护的静态方法。

```php
class Route extends Facade
{
    /**
     * Get the registered name of the component.
     *
     * @return string
     */
    protected static function getFacadeAccessor()
    {
        return 'router';
    }
}
```

其实这种使用方式最关键的一点是实现了`__callStatic`静态方法，这是php内置的一个魔术方法，**当调用类不存在或者没有权限访问的静态方法时会调用**。聪明的同学应该看到了，门面类继承了一个抽象类Facade，可以看到里面确实实现了`__callStatic`静态方法。

```php
public static function __callStatic($method, $args)
    {
        $instance = static::getFacadeRoot();

        if (! $instance) {
            throw new RuntimeException('A facade root has not been set.');
        }

        return $instance->$method(...$args);
    }
```
到这里其实实现原理已经一目了然了，拿DB门面类举例，DB::table()调用时，门面类DB并不存在静态方法table()于是调用父类的`__callStatic`静态方法,`__callStatic`方法通过调用子类的getFacadeAccessor()方法获取实际类名，最终获取类的实例对象，然后调用相关方法。

```php
$instance->$method(...$args);
```

### 完整代码

下面是我提取的Laravel门面基类的所有关键代码

```php
//抽象类
abstract class Facade
{
    /**
     * The resolved object instances.
     *
     * @var array
     */
    protected static $resolvedInstance;

    /**
     * Get the root object behind the facade.
     *
     * @return mixed
     */
    public static function getFacadeRoot()
    {
        return static::resolveFacadeInstance(static::getFacadeAccessor());
    }

    /**
     * Handle dynamic, static calls to the object.
     *
     * @param  string  $method
     * @param  array  $args
     * @return mixed
     *
     * @throws \RuntimeException
     */
    public static function __callStatic($method, $args)
    {
        $instance = static::getFacadeRoot();

        if (! $instance) {
            throw new RuntimeException('A facade root has not been set.');
        }

        return $instance->$method(...$args);
    }

    /**
     * Resolve the facade root instance from the container.
     *
     * @param  object|string  $name
     * @return mixed
     */
    protected static function resolveFacadeInstance($name)
    {
        if (is_object($name)) {
            return $name;
        }

        if (isset(static::$resolvedInstance[$name])) {
            return static::$resolvedInstance[$name];
        }

        return new $name;
        
    }


    /**
     * Get the registered name of the component.
     *
     * @return string
     *
     * @throws \RuntimeException
     */
    protected static function getFacadeAccessor()
    {
        throw new RuntimeException('Facade does not implement getFacadeAccessor method.');
    }
}
```

DB门面类

```php
class DB extends Facade
{
    protected static function getFacadeAccessor()
    {
        return "Mysql";
    }
}
```

一个简单的Mysql实际类
```php
class Mysql 
{
    public function find($id)
    {
        print_r($id);
    }

    public function findAll($ids)
    {
       print_r($ids);
    }

    public function insert($data)
    {
        print_r($data);
    }
}

```
于是我们就可以像Laravel一样使用门面了

```php
DB::find(1);
DB::findAll([1,2,3]);
DB::insert(['name'=>'Json','pwd'=>3333]);
```