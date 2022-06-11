### PHP
写了一个简单的console类，linux命令行下可以输出7种不同颜色的字体,有命令行需求的时候可以直接拿来用。代码如下
```php
class Console {

    private static $modesList = [
        [
            'name'  =>  "log",   
            'color' =>  "white",  
            'font'  =>   37,
        ],
        [
            'name'  =>  "error", 
            'color' =>  "red",   
            'font'  =>  31
        ],
        [
            'name'  =>  "debug", 
            'color' =>  "cyan",   
            'font'  =>  36
        ],
        [
            'name'  =>  "info", 
            'color' =>  "green",   
            'font'  =>  32
        ],
        [
            'name'  =>  "warn", 
            'color' =>  "yellow",   
            'font'  =>  33
        ],
        [
            'name'  =>  "trace", 
            'color' =>  "blue",   
            'font'  =>  34
        ],
        [
            'name'  =>  "fatal", 
            'color' =>  "magenta",   
            'font'  =>  35
        ]
    ];

    private static function isWindows() {
        return strtolower(substr(PHP_OS, 0, 3)) === 'win';
    }

    private static function isCli() {
        return preg_match("/cli/i", php_sapi_name()) ? true : false;
    }

    public static function __callStatic($method,$arguments) {
        $colorDict = array_column(self::$modesList,'font','color');
        $levelDict = array_column(self::$modesList,'font','name');
        if(!isset($colorDict[$method]) && !isset($levelDict[$method])) {
            throw new Exception("方法不存在");
        }
        $message = is_array($arguments)?implode(',',$arguments):$arguments;
        if(self::isWindows() || !self::isCli()) {
            echo $message;
        } else {
            if(isset($colorDict[$method])) {
                echo sprintf("\033[0;%sm%s\033[0m",''. $colorDict[$method],$message);
            } else {
                echo sprintf("\033[0;%sm%s\033[0m",''. $levelDict[$method],$message);
            }
        }
        if(self::isCli()) {
            echo "\n";
        }
    }

    public static function success($message) {
        self::green($message);
    }

    public static function fail($message) {
        self::red($message);
    }

}
```

### 使用方式
```php
Console::info("this is info test");
Console::debug("this is debug test");
Console::warn("this is warn test");
Console::error("this is error test");
Console::trace("this is trace test");
Console::log("this is log test");
Console::fatal("this is fatal test");

Console::success("this is success test");
Console::fail("this is fail test");

Console::white("this is white test");
Console::red("this is red test");
Console::green("this is green test");
Console::yellow("this is yellow test");
Console::blue("this is blue test");
Console::magenta("this is magenta test");
Console::cyan("this is cyan test");
```

### 效果图
![console.png](http://www.fxjson.com/usr/uploads/2021/03/2498297245.png)