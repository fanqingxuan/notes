### PHP
今天看Laravel源码中的composer时，发现了`illuminate/macroable`,然后就了解了下，包代码只有一个文件，源码如下:
```php
<?php

namespace Illuminate\Support\Traits;

use BadMethodCallException;
use Closure;
use ReflectionClass;
use ReflectionMethod;

trait Macroable
{
    /**
     * The registered string macros.
     *
     * @var array
     */
    protected static $macros = [];

    /**
     * Register a custom macro.
     *
     * @param  string  $name
     * @param  object|callable  $macro
     * @return void
     */
    public static function macro($name, $macro)
    {
        static::$macros[$name] = $macro;
    }

    /**
     * Mix another object into the class.
     *
     * @param  object  $mixin
     * @param  bool  $replace
     * @return void
     *
     * @throws \ReflectionException
     */
    public static function mixin($mixin, $replace = true)
    {
        $methods = (new ReflectionClass($mixin))->getMethods(
            ReflectionMethod::IS_PUBLIC | ReflectionMethod::IS_PROTECTED
        );
        var_dump($methods);
        foreach ($methods as $method) {
            if ($replace || ! static::hasMacro($method->name)) {
                $method->setAccessible(true);
                static::macro($method->name, $method->invoke($mixin));
            }
        }
    }

    /**
     * Checks if macro is registered.
     *
     * @param  string  $name
     * @return bool
     */
    public static function hasMacro($name)
    {
        return isset(static::$macros[$name]);
    }

    /**
     * Flush the existing macros.
     *
     * @return void
     */
    public static function flushMacros()
    {
        static::$macros = [];
    }

    /**
     * Dynamically handle calls to the class.
     *
     * @param  string  $method
     * @param  array  $parameters
     * @return mixed
     *
     * @throws \BadMethodCallException
     */
    public static function __callStatic($method, $parameters)
    {
        if (! static::hasMacro($method)) {
            throw new BadMethodCallException(sprintf(
                'Method %s::%s does not exist.', static::class, $method
            ));
        }

        $macro = static::$macros[$method];

        if ($macro instanceof Closure) {
            $macro = $macro->bindTo(null, static::class);
        }

        return $macro(...$parameters);
    }

    /**
     * Dynamically handle calls to the class.
     *
     * @param  string  $method
     * @param  array  $parameters
     * @return mixed
     *
     * @throws \BadMethodCallException
     */
    public function __call($method, $parameters)
    {
        if (! static::hasMacro($method)) {
            throw new BadMethodCallException(sprintf(
                'Method %s::%s does not exist.', static::class, $method
            ));
        }

        $macro = static::$macros[$method];

        if ($macro instanceof Closure) {
            $macro = $macro->bindTo($this, static::class);
        }

        return $macro(...$parameters);
    }
}

```
使用例子如下:
```php
require __DIR__.'/vendor/autoload.php';

use Illuminate\Support\Traits\Macroable;

class Hello {
    use Macroable;
    
    public $name = "hello";
}

Hello::macro("sayHi",function($name) {
    echo "hello,".$name;
});

Hello::sayHi("Json");

$hello = new Hello;

$hello->sayHi("杰哥");


Hello::macro("Say",function() {
    echo "hello,".$this->name;
});

$hello->Say();
```
macroable包就是定义了一个trait，然后使用了该trait的类，可以通过代码动态挂载函数到类或者实例上面，然后可以像访问静态方法或者实例方法一样调用方法了，如上面的`Hello::sayHi`和`$hello->Say`。

本质上还是利用了php本身的`__call`和`__callStatic`魔术方法结合`static`静态变量实现的，本人越来越不喜欢魔术方法了，因为编辑器难以提示，IDE追踪起来困难，但是代码却可以完好跑起来，太神奇了，心里总是不踏实。

但是不可否认的是`macroable`包，让代码看起来更优雅，更面向对象了，让开发者写代码更舒服了