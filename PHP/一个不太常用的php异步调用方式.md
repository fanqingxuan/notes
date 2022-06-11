### PHP
php异步调用基本上都是结合消息队列来实现的，但是有时候现实是很骨感的，服务端没有安装消息队列服务，那这时候怎么实现异步呢；还有一个策略是结合存储中间件+crontab轮询，但是这难免会有时间差的影响，不能实时触发，笔者就遇到了这种困境，项目早期没有消息队列的引入，又要实时触发，要优化当前请求的效率，怎么办呢，就引入了今天的一个不太常用的异步方式。

### 异步好处

- 解耦代码
- 减少客户端的响应时间，提高吞吐量

### 代码
```php
<?php
pclose(popen("/usr/bin/php /home/web/task.php &",'r'));
```
### 代码说明

对的你没有看错,用popen+pclose函数就可以实现异步执行代码了

- popen
1. popen()函数通过创建一个管道，调用fork()产生一个子进程
2. popen(command,mode)有两个参数，第一个是执行的命令，第二个是连接模式

- pclose
关闭由popen()打开的管道

*注意popen中command中的异步是有条件的，需要在command后面加上“&”，表示后台执行，这样才不会对PHP造成阻塞*

### 缺点

- popen只能在本机执行
- command不能传递大数据量的参数
- 高并发时会创建很多进程
- 根据服务器自身情况会出现fork出的进程执行不完就被kill掉

### 其它

使用popen、pclose函数前需要检查php.ini里面disable_functions参数有没有禁用掉这些函数,有的服务器配置考虑到系统安全性是禁用这些函数的
```shell
disable_functions => passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server => passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server
```
如果禁用的话，使用函数会出现错误
```shell
PHP Warning:  popen() has been disabled for security reasons
PHP Warning:  pclose() expects parameter 1 to be resource, null given in 
```
