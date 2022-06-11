### PHP
### 前言

这次要介绍的是一个Symfony组件，使用该组件可以通过不同的方式查找文件或者目录，比如根据名称、文件大小、修改时间等条件查找。

### 安装

```shell
composer require symfony/finder
```

### 使用

```php
require_once __DIR__.'/vendor/autoload.php';

use Symfony\Component\Finder\Finder;

$finder = new Finder();
// find all files in the current directory
$finder->files()->in(__DIR__);

// check if there are any search results
if ($finder->hasResults()) {
    // ...
}

foreach ($finder as $file) {
    $absoluteFilePath = $file->getRealPath();
    $fileNameWithExtension = $file->getRelativePathname();

    var_dump($fileNameWithExtension);
}
```

**注意:Finder实例对象不会重置内部状态，这也就意味着如果你想获取多个结果的话需要重新创建Finder类的实例对象**

### 方法说明

- in()

  in()方法确定查找目录，用法如下:

  ```php
  $finder->in(__DIR__);
  ```

  多个目录查找

  ```php
  $finder->in(['./a', './b']);
  
  // 跟上面等价
  $finder->in('./a')->in('./b');
  ```

  使用*通配符进行模糊匹配查找

  ```php
  $finder->files()->in('./a*');//查找当前目录下a开头的目录
  ```

  排除查找某些目录

  ```php
  $finder->in(__DIR__)->exclude('ruby');
  ```

  忽略没有权限的目录

  ```php
  $finder->ignoreUnreadableDirs()->in(__DIR__);
  ```

- files()&directories()

  默认情况下，查找会返回所有的目录和文件，如果想只返回目录可以使用directories(), 只返回符合条件的文件可以使用files()方法，如下

  ```php
  // find all files in the current directory
  $finder->files()->in('./a*')->files();
  
  foreach($finder as $file) {
      print_r($file->getRelativePathname());
  }
  ```

- followLinks()

  通过followLinks()方法，可以返回快捷方式，或者连接

  ```php
  $finder->files()->followLinks();
  ```

- name()

  根据名字进行查找

  ```php
  $finder->in('./')->files()->name('*.php');//查找当前目录下的php文件
  ```

  name()方法可以根据正则,glob，字符串形式的文件名匹配查找

  ```php
  $finder->files()->name('/\.php$/');
  ```

  name()也支持多种文件名方式的查找

  ```php
  $finder->files()->name(['*.php', '*.py']);//查找后缀是php或者py的文件
  
  //等价于上面的方式
  $finder->files()->name('*.php')->name('*.py');
  ```

  查找不包含某种模式的文件,如查找不是php后缀的所有文件

  ```php
  $finder->files()->notName('*.php');
  
  $finder->files()->notName(['*.rb', '*.py']);//不是php、py后缀的所有文件
  ```

- contains()

  根据文件内容查找,如查找文件内容包含hello的所有文件

  ```php
  $finder->files()->in('./a*')->files()->contains('hello');
  ```

  也可以根据正则匹配查找文件

  ```php
  $finder->files()->contains('/\s+hello/i');
  ```

  查找文件内容不包含hello的所有文件

  ```php
  $finder->files()->notContains('hello');
  ```

- path()

  查找匹配的文件或者目录

  ```php
  $finder->path('data')->name('*.xml');//查找data.xml或者data/*.xml
  ```

  同其它方法一样，也支持数组方式或者正则查找

  ```php
  $finder->path(['data', 'public']);
  $finder->path('/^public\/images/');
  ```

  不查找某文件和目录

  ```php
  $finder->notPath('public/image');
  ```

- size()

  根据文件大小查找文件

  ```php
  $finder->files()->size('< 1.5K');
  $finder->files()->size('>= 1M')->size('<= 2M');//根据区间查找
  ```

- date()

  根据修改时间查找

  ```php
  $finder->date('since yesterday');
  ```

- 对查找结果排序

  ```php
  $finder->sortByName();
  $finder->sortByType();
  $finder->sortByChangedTime();
  $finder->sortByModifiedTime();
  ```

  