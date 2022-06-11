### PHP
我不是大牛，我只是代码的搬运工，上周末学习了下CodeIgniter4，首先看了下composer.json的依赖库，其中require里面有kint-php/kint，这是什么鬼，于是搜索了一番资料。

### 介绍
Kint是一个免费开源，用来替代系统内置的比如var_dump(),print_r(),debug_backtrace()等相关函数的调试利器,方便的很，谁用谁知道，不过当前像laravel、symfony等开源项目都有自己的调试规则，一般用不到，但是对自研项目还是有可取之处的。话不多说，直接看效果图
![1.jpg](http://39.103.150.24/usr/uploads/2020/07/3110350317.jpg)

### 安装
```php
composer require kint-php/kint
```

### 使用
```php
<?php

require_once './vendor/autoload.php';
Kint\Renderer\RichRenderer::$theme = 'aante-light.css';

//Kint::$enabled_mode = false;
Kint\Renderer\RichRenderer::$folder	= false;
d($_SERVER,$GLOBALS);
```
### 配置说明

- 设置主题

支持四种样式的主题original.css、solarized.css、solarized-dark.css、aante-light.css,默认是original.css,可以使用下面的方法修改主题

```php
Kint\Renderer\RichRenderer::$theme = 'aante-light.css';
```
- 展开输出调试
组件默认将调试输出折叠在浏览单底部，如果需要默认展开显示可设置如下

```php
Kint\Renderer\RichRenderer::$folder	= false;//true折叠，false不折叠
```

- 关闭输出
有时候，我们只想在开发环境进行调试输出，可以通过配置 Kint::$enabled_mode = false;来将相应的代码不输出

### 方法说明

```php

//输出
Kint::dump($_SERVER,$GLOBALS);
d($_SERVER,$GLOBALS);//Kint::dump的简写方式

//跟踪调试信息:
Kint::trace();
Kint::dump( 1 );//同Kint::trace()

//文本方式输出
s($_SERVER);

```

### 其它

更多使用上的用法可以参考[Kint官网](https://kint-php.github.io/kint)，我这里只是抛转引用，借用github上看到的一句话:想学习的人总会找到目录，我改成想学习的人总会找到入口。
