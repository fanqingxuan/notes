### PHP
### 介绍
![11.jpg](http://www.fxjson.com/usr/uploads/2020/07/2050173822.jpg)

使用过laravel框架的同学对这个友好的错误提示页面应该不陌生，其实这个也是有开源库的，laravel正是集成了这个库，这个库名字叫whoops,你可以在github上面很轻松的搜索到它。

### 安装

```shell
composer require filp/whoops
```

### demo代码

```php
<?php

require_once(__DIR__ . '/vendor/autoload.php');

use Whoops\Run;
use Whoops\Handler\PrettyPageHandler;

$whoops = new Run;
$whoops->pushHandler(new PrettyPageHandler);  
$whoops->register();

require_once './demo.php';

```
其中demo.php文件演示代码比较简单，只有一行

```php
<?php
array()+33;
```

### 其它

使用就这么简单，如果你想定制化输出内容，或者添加更多信息，可以参考[github](https://github.com/filp/whoops),我这只是抛转引玉，还是那句:想学的人总能找到资源

