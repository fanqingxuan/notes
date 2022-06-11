### PHP
### 常规用法
switch最常用的方式是传递一个参数然后逐一跟case对比，类型下面的代码
```php
$os = PHP_OS;
switch($os) {
    case "LINUX":
        echo "LINUX";
        break;
    case "WINNT":
        echo "Windows";
        break;
    case "Mac":
        echo "Mac";
        break;
}
```
这里有一个需要注意的地方，就是PHP是弱类型语言的宽松比较,举例如下:
```php
$variable = 123;
switch($variable) {
    case "123hello":
        echo "123hello";
        break;
    case "123":
        echo "123";
		break;
}
```
理想情况下，我们希望的是输出`123`,但是却输出了`123hello`,解决这种方式比较简单，如下,将参数转成跟case一样的类型
```php
$variable = 123;
switch(strval($variable)) {
    case "123hello":
        echo "123hello";
        break;
    case "123":
        echo "123";
		break;
}
```

### 进阶用法
在阅读Codeigniter4代码的时候，看到有如下的用法，用true作为switch参数，每个case就相当于一个else if
```php
switch (true)
{
	case array_key_exists($name, $_ENV):
		return $_ENV[$name];
	case array_key_exists($name, $_SERVER):
		return $_SERVER[$name];
	default:
		$value = getenv($name);

		// switch getenv default to null
		return $value === false ? null : $value;
}
```
我表示自己在工作当中没有用过这样的方式，这样的好处是代码逻辑清晰，在分支比较多的时候，用switch代替if、else if，一目了然。

### switch的坑
先看代码:
```php
$var = 0;
switch($var) {
    case $var>=0:
        echo 0;
        break;
    case $var>10:
        echo 1;
        break;
    default:
        echo 2;
}
```
你觉得结果应该是输出什么呢?乍一看,似乎应该是进入第一个case，但是还有一个但是，
**switch匹配的是case中表达式的值,不能简单的把case当if用**
这里第一行case ($var >= 0),这个条件表达式的值为`true`,switch($var) 中传过来的是0,0和true 匹配,当然匹配不上。下面几行都是false，第二行 0 和false匹配，所以匹配走第二个case然后break。
所以最后应该输出： 1

