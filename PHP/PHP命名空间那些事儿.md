### PHP
##  概述
什么是命名空间？从广义上来说，命名空间是一种封装事物的方法。在很多地方都可以见到这种抽象概念。例如，在操作系统中目录用来将相关文件分组，对于目录中的文件来说，它就扮演了命名空间的角色。具体举个例子，文件 foo.txt 可以同时在目录/home/greg 和 /home/other 中存在，但在同一个目录中不能存在两个 foo.txt 文件。另外，在目录 /home/greg 外访问 foo.txt 文件时，我们必须将目录名以及目录分隔符放在文件名之前得到 /home/greg/foo.txt。这个原理应用到程序设计领域就是命名空间的概念。(引用自php.net)

命名空间主要用来解决两类问题：
* 用户编写的代码与PHP内部的或第三方的类、函数、常量、接口名字冲突
* 为很长的标识符名称创建一个别名的名称，提高源代码的可读性

PHP命名空间提供了一种将相关的类、函数、常量和接口组合到一起的途径，不同命名空间的类、函数、常量、接口相互隔离不会冲突，注意：PHP命名空间只能隔离类、函数、常量和接口，不包括全局变量。

接下来的两节将介绍下PHP命名空间的内部实现，主要从命名空间的定义及使用两个方面分析。

## 命名空间的定义
### 定义语法
命名空间通过关键字namespace 来声明，如果一个文件中包含命名空间，它必须在其它所有代码之前声明命名空间，除了declare关键字以外，也就是说除declare之外任何代码都不能在namespace之前声明。另外，命名空间并没有文件限制，可以在多个文件中声明同一个命名空间，也可以在同一文件中声明多个命名空间。
```php
namespace com\aa;

const MY_CONST = 1234;
function my_func(){ /* ... */ }
class my_class { /* ... */ }
```
另外也可以通过{}将类、函数、常量封装在一个命名空间下：
```php
namespace com\aa{
    const MY_CONST = 1234;
    function my_func(){ /* ... */ }
    class my_class { /* ... */ }
}
```
但是同一个文件中这两种定义方式不能混用，下面这样的定义将是非法的：
```php
namespace com\aa{
    /* ... */
}

namespace com\bb;
/* ... */
```
如果没有定义任何命名空间，所有的类、函数和常量的定义都是在全局空间，与 PHP 引入命名空间概念前一样。

### 内部说明
命名空间的实现实际比较简单，当声明了一个命名空间后，接下来编译类、函数和常量时会把类名、函数名和常量名统一加上命名空间的名称作为前缀存储，也就是说声明在命名空间中的类、函数和常量的实际名称是被修改过的，这样来看他们与普通的定义方式是没有区别的，只是这个前缀是内核帮我们自动添加的，例如：
```php
//ns_define.php
namespace com\aa;

const MY_CONST = 1234;
function my_func(){ /* ... */ }
class my_class { /* ... */ }
```
最终MY_CONST、my_func、my_class在EG(zend_constants)、EG(function_table)、EG(class_table)中的实际存储名称被修改为：com\aa\MY_CONST、com\aa\my_func、com\aa\my_class。

## 命名空间的使用
### 基本用法
我们知道了定义在命名空间中的类、函数和常量只是加上了namespace名称作为前缀，既然是这样那么在使用时加上同样的前缀是否就可以了呢？答案是肯定的，比如上面那个例子：在com\aa命名空间下定义了一个常量MY_CONST，那么就可以这么使用：
```php
include 'ns_define.php';

echo \com\aa\MY_CONST;
```
这种按照实际类名、函数名、常量名使用的方式很容易理解，与普通的类型没有差别，这种以"\"开头使用的名称称之为：**完全限定名称**，类似于绝对目录的概念，使用这种名称PHP会直接根据"\"之后的名称去对应的符号表中查找(namespace定义时前面是没有加"\"的，所以查找时也会去掉这个字符)。

除了这种形式的名称之外，还有两种形式的名称：
* __非限定名称:__ 即没有加任何namespace前缀的普通名称，比如my_func()，使用这种名称时如果当前有命名空间则会被解析为：currentnamespace\my_func，如果当前没有命名空间则按照原始名称my_func解析
* __部分限定名称:__ 即包含namespace前缀，但不是以"\"开始的，比如：aa\my_func()，类似相对路径的概念，这种名称解析规则比较复杂，如果当前空间没有使用use导入任何namespace那么与非限定名称的解析规则相同，即如果当前有命名空间则会把解析为：currentnamespace\aa\my_func，否则解析为aa\my_func，使用use的情况后面再作说明

### use导入
使用一个命名空间中的类、函数、常量虽然可以通过完全限定名称的形式访问，但是这种方式需要在每一处使用的地方都加上完整的namespace名称，如果将来namespace名称变更了就需要所有使用的地方都改一遍，这将是很痛苦的一件事，为此，PHP提供了一种命名空间导入/别名的机制，可以通过use关键字将一个命名空间导入或者定义一个别名，然后在使用时就可以通过导入的namespace名称最后一个域或者别名访问，不需要使用完整的名称，比如：
```php
//ns_define.php
namespace aa\bb\cc\dd;

const MY_CONST = 1234;
```
可以采用如下几种方式使用：
```php
//方式1:
include 'ns_define.php';

use aa\bb\cc\dd;

echo dd\MY_CONST;
```
```php
//方式2:
include 'ns_define.php';

use aa\bb\cc;

echo cc\dd\MY_CONST;
```
```php
//方式3:
include 'ns_define.php';

use aa\bb\cc\dd as DD;

echo DD\MY_CONST;
```
```php
//方式4:
include 'ns_define.php';

use aa\bb\cc as CC;

echo CC\dd\MY_CONST;
```
这种机制的实现原理也比较简单：编译期间如果发现use语句 ，那么就将把这个use后的命名空间名称插入一个哈希表：FC(imports)，而哈希表的key就是定义的别名，如果没有定义别名则key使用按"\"分割的最后一节，比如方式2的情况将以cc作为key，即：FC(imports)["cc"] = "aa\bb\cc\dd"；接下来在使用类、函数和常量时会把名称按"\"分割，然后以第一节为key查找FC(imports)，如果找到了则将FC(imports)中保存的名称与使用时的名称拼接在一起，组成完整的名称。实际上这种机制是把完整的名称切割缩短然后缓存下来，使用时再拼接成完整的名称，也就是内核帮我们组装了名称，对内核而言，最终使用的都是包括完整namespace的名称。


use除了上面介绍的用法外还可以导入一个类，导入后再使用类就不需要加namespace了，例如：
```php
//ns_define.php
namespace aa\bb\cc\dd;

class my_class { /* ... */ }
```
```php
include 'ns_define.php';
//导入一个类
use aa\bb\cc\dd\my_class;
//直接使用
$obj = new my_class();
var_dump($obj);
```
use的这两种用法实现原理是一样的，都是在编译时通过查找FC(imports)实现的名称补全。从PHP 5.6起，use又提供了两种针对函数、常量的导入，可以通过`use function xxx`及`use const xxx`

简单总结下use的几种不同用法：
* __a.导入命名空间:__ 导入的名称保存在FC(imports)中，编译使用的语句时搜索此符号表进行补全
* __b.导入类:__ 导入的名称保存在FC(imports)中，与a不同的是不会根据"\"切割后的最后一节检索，而是直接使用类名查找
* __c.导入函数:__ 通过`use function`导入到FC(imports_function)，补全时先查找FC(imports_function)，如果没有找到则继续按照a的情况处理
* __d.导入常量:__ 通过`use const`导入到FC(imports_const)，补全时先查找FC(imports_const)，如果没有找到则继续按照a的情况处理

```php
use aa\bb;                  //导入namespace
use aa\bb\MY_CLASS;         //导入类
use function aa\bb\my_func; //导入函数
use const aa\bb\MY_CONST;   //导入常量
```

### 动态用法
前面介绍的这些命名空间的使用都是名称为CONST类型的情况，所有的处理都是在编译环节完成的，PHP是动态语言，能否动态使用命名空间呢？举个例子：
```php
$class_name = "\aa\bb\my_class";
$obj = new $class_name;
```
如果类似这样的用法只能用完全限定名称，也就是按照实际存储的名称使用，无法进行自动名称补全。