### PHP
写过几年php代码的同学应该见识了php的异常以及错误，有些exception我们可以通过代码捕获，有时候浏览器会报500错误，这时候我们直接的做法是看php的error log文件，一般都是出现了Fatal error错误，但是我们使用过框架的同学应该都知道了，Fatal error错误框架也都捕获并做了漂亮的突出美好，便于我们调试，感觉好高级的样子，怎么捕获的Fatal error错误呢，带着这种疑惑我们一起来学习下

### php中的error级别

```shell
Fatal Error:致命错误（脚本终止运行）
    E_ERROR         // 致命的运行错误，错误无法恢复，暂停执行脚本
    E_CORE_ERROR    // PHP启动时初始化过程中的致命错误
    E_COMPILE_ERROR // 编译时致命性错，就像由Zend脚本引擎生成了一个E_ERROR
    E_USER_ERROR    // 自定义错误消息。像用PHP函数trigger_error（错误类型设置为：E_USER_ERROR）

Parse Error：编译时解析错误，语法错误（脚本终止运行）
    E_PARSE  //编译时的语法解析错误

Warning Error：警告错误（仅给出提示信息，脚本不终止运行）
    E_WARNING         // 运行时警告 (非致命错误)。
    E_CORE_WARNING    // PHP初始化启动过程中发生的警告 (非致命错误) 。
    E_COMPILE_WARNING // 编译警告
    E_USER_WARNING    // 用户产生的警告信息

Notice Error：通知错误（仅给出通知信息，脚本不终止运行）
    E_NOTICE      // 运行时通知。表示脚本遇到可能会表现为错误的情况.
    E_USER_NOTICE // 用户产生的通知信息。
```

我们可以看到有E_ERROR | E_CORE_ERROR |  E_COMPILE_ERROR | E_USER_ERROR | E_PARSE这5中错误会导致程序终止并退出

### PHP异常处理的特技
其实也没什么，就是用了3个php提供的函数
- set_error_handler    这个函数用于设置一个用户自定义的错误处理函数

当程序出现错误的时候自动调用此方法，不过需要注意一下两点：第一，如果存在该方法，相应的error_reporting()就不能在使用了。所有的错误都会交给自定义的函数处理。第二，此方法不能处理以下级别的错误：E_ERROR、 E_PARSE、 E_CORE_ERROR、 E_CORE_WARNING、 E_COMPILE_ERROR、 E_COMPILE_WARNING，set_error_handler() 函数所在文件中产生的E_STRICT，该函数只能捕获系统产生的一些Warning、Notice级别的错误

```php
<?php

set_error_handler("errorHandler");

function errorHandler($type, $message, $file, $line) 
{
    var_dump('<b>set_error_handler: ' . $type . ':' . $message . ' in ' . $file . ' on ' . $line . ' line .</b><br />');
}

echo 1/0;
```
**注意:**
这个函数可以支持注册函数和实例方法两种方式

```php
<?php
 // 直接传函数名 NonClassFunction
 set_error_handler('function_name');

 // 传 class_name && function_name
 set_error_handler(array('class_name', 'method_name'));
?>
```
- register_shutdown_function()  这个函数用于注册一些自定义函数，这些函数会在PHP脚本执行结束前调用，比如脚本错误、die()、exit、异常、正常结束后都会调用被register_shutdown_function注册的自定义函数

这个函数很厉害，即使代码里有exit、die等暴力操作,只要我们在die之前通过register_shutdown_function注册了自定义函数，那自定义函数就可以在程序结束之前被调用。基于这个特点，我们可以借助于error_get_last()函数(这个函数可以拿到本次执行产生的所有错误)，捕获一些像Fatal Error、Parse Error等set_error_handler捕获不到的错误。

error_get_last()返回的信息：

```shell
　　[type]           - 错误类型
　　[message]        - 错误消息
　　[file]           - 发生错误所在的文件
　　[line]           - 发生错误所在的行
```

demo示例:
```php
<?php

register_shutdown_function('shutdownHandler');

function shutdownHandler()
{
    if ($error = error_get_last()) {
        var_dump('<b>register_shutdown_function: Type:' . $error['type'] . ' Msg: ' . $error['message'] . ' in ' . $error['file'] . ' on line ' . $error['line'] . '</b>');
    }
}

echo 4++;
?>
```

如果你没敲什么的代码，认为这个demo没问题，就被带进坑里了，这个例子目的很明显是要抛出php语法错误，真正运行下代码会发现，并不能捕获错误,依然由php抛出错误

```shell
PHP Parse error:  syntax error, unexpected '++' (T_INC), expecting ',' or ';' in E:\project\demo\index.php on line 12
```

很奇怪，我们用laravel或者其它框架的时候，这种错误是可以捕获的，什么原因呢? 其实原因很简单，只在parse-time出错时是不会调用本函数的，只有在run-time出错的时候，才会调用register_shutdown_function函数。

框架中一般会有统一的入口index.php，然后每个类库文件都会通过include ** 的方式加载到index.php中，相当与所有的程序都会在index.php中聚集，同样，你写的具有语法错误的文件也会被引入到入口文件中，这样的话，调用框架，执行index.php，index.php本身并没有语法错误，也就不会产生parse-time错误，而是 include 文件出错了，是run-time的时候出错了，所以框架执行完之后就会触发register_shutdown_function()

现在可以试一下这个写法，这样就会触发register_shutdown_function()函数了：
```php
//a.php
<?php
echo 3++;
?>

//index.php
<?php

error_reporting(0);
register_shutdown_function('shutdownHandler');

function shutdownHandler()
{
    if ($error = error_get_last()) {
        var_dump('<b>register_shutdown_function: Type:' . $error['type'] . ' Msg: ' . $error['message'] . ' in ' . $error['file'] . ' on line ' . $error['line'] . '</b>');
    
    }
}

require ('a.php');
?>
```

- set_exception_handler() 设置默认的异常处理程序，用在没有用try/catch块来捕获的异常，也就是说不管你抛出的异常有没有人捕获，如果没有人捕获就会进入到该方法中，并且在回调函数调用后异常会中止。

```php
<?php

error_reporting(0);
set_exception_handler('exceptionHandler');

function exceptionHandler($exception)
{
    var_dump("<b>set_exception_handler: Exception: " . $exception->getMessage()  . '</b>');
}

throw new Exception("手动报错");

?>
```
### 综合技巧
根据上面的说明我们，可以知道set_exception_handler可以捕获异常；set_error_handler可以捕获一些错误，但不是所有错误；可以利用register_shutdown_function的最后被调用的函数这个特点，捕获set_error_handler捕获不到的错误；

因此我们可以综合使用，自定义异常以及输出了
```php
<?php

error_reporting(0);

class MyException {

    public function init() 
    {
        set_exception_handler([$this, 'exceptionHandler']);

        set_error_handler([$this, 'errorHandler']);

        register_shutdown_function([$this, 'shutdownHandler']);
    }
    
    public function exceptionHandler(Throwable $exception) 
    {
        print_r($exception->getMessage());
        print_r($exception->getLine());
        print_r($exception->getTrace());
    }
    
    public function errorHandler(int $severity, string $message, string $file = null, int $line = null)
    {
        if (! (error_reporting() & $severity))
        {
            return;
        }

        throw new ErrorException($message, 0, $severity, $file, $line);
    }
    
    public function shutdownHandler()
    {
        $error = error_get_last();

        if (! is_null($error))
        {
            // Fatal Error?
            if (in_array($error['type'], [E_ERROR, E_CORE_ERROR, E_COMPILE_ERROR, E_PARSE]))
            {
                $this->exceptionHandler(new ErrorException($error['message'], $error['type'], 0, $error['file'], $error['line']));
            }
        }
    }
}

(new MyException)->init();

test();
?>
```