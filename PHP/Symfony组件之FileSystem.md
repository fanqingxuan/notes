### PHP
今天要介绍的是一个操作文件系统的工具包---symfony/filesystem，由Symfony框架提供。

### 安装

```shell
 composer require symfony/filesystem
```

### 常用方法

- mkdir()

  递归的创建目录，第二个参数设置目录权限，默认777,如果目录已经存在，则会自动忽略`exists`

  ```php
  <?php
  
  require 'vendor/autoload.php';
  
  use Symfony\Component\Filesystem\Exception\IOExceptionInterface;
  use Symfony\Component\Filesystem\Filesystem;
  
  $filesystem = new Filesystem();
  
  try {
      $filesystem->mkdir('./test/a/'.random_int(0, 1000),755);
  } catch (IOExceptionInterface $exception) {
      echo "An error occurred while creating your directory at ".$exception->getPath();
  }
  ```

- exists()

  判断目录或者文件是否存在，可以传多个值，如果有任意一个不存在，则返回false，否则返回true

  ```php
  $filesystem->exists(['./test/aa']);
  ```

  

- copy()

  复制单个文件

  ```php
  $filesystem->copy('./test/a.txt','./a.txt',true);//强制将test目录下的a.txt复制到当前目录的a.txt文件，如果当前目录已经存在a.txt，则会覆盖
  
  $filesystem->copy('./test/a.txt','./a.txt');//复制test目录的a.txt文件到当前目录，如果当面目录a.txt不存在，或者test/a.txt文件修改时间比当前目录a.txt新，则复制成功
  ```

  

  

- touch()创建文件

  ```php
  $filesystem->touch('file.txt');
  ```

- chown()修改所有者

  ```php
  $filesystem->chown('file.txt','www');
  ```

- chmod()修改权限

  ```php
  $filesystem->chmod('file.txt',700);
  ```

- remove()

  删除文件、目录、快捷方式

  ```php
  $filesystem->remove(["file.txt","test"]);//删除file.txt文件以及test目录
  ```

- rename()

  重命名文件或者目录

  ```php
  $filesystem->touch('file.txt');
  $filesystem->rename("file.txt","hello.txt");//file.txt重命名为hello.txt
  ```

- symlink()

  创建快捷方式

  ```php
  $filesystem->touch('file.txt');
  $filesystem->symlink("file.txt","dd");//创建file.txt文件的快捷方式dd
  ```

- makePathRelative()

  返回相对目录，指的是第一个参数相对于第二个参数的目录

  ```php
  $filesystem->makePathRelative(
      '/var/lib/symfony/src/Symfony/',
      '/var/lib/symfony/src/Symfony/Component'
  );//
  
  // returns 'videos/'
  $filesystem->makePathRelative('/tmp/videos', '/tmp')
  ```

- mirror()

  复制目录内所有子目录以及文件

  ```php
  $filesystem->mirror('./vendor','./test');//将vendor目录内容备份到test目录
  ```

- isAbsolutePath()

  判断是否绝对路径

  ```php
  $filesystem->isAbsolutePath("/opt/lampp");
  ```

- tempnam()

  创建文件名唯一的临时文件,并返回文件名

  ```php
  // returns a path like : /tmp/prefix_wyjgtF
  $filesystem->tempnam('/tmp', 'prefix_');
  // returns a path like : /tmp/prefix_wyjgtF.png
  $filesystem->tempnam('/tmp', 'prefix_', '.png');
  ```

- dumpFile()

  保存文件内容

  ```php
  $filesystem->dumpFile('file.txt', 'Hello World');//file.txt内容变成了Hello World
  ```

- appendToFile()

  在原有文件末尾追加文件内容

  ```php
  $filesystem->appendToFile('file.txt', 'append hello world');
  ```

  