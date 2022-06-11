### PHP
我们经常用Redis的List结构去做一个轻量级的消息队列

### 常规做法
一般来说大家都是使用lpush和rpop、rpush和lpop这两对命令实现，具体来说就是用lpush或者rpush向队列里面写数据，然后写一个脚本调用rpop或者lpop弹出消息进行业务处理，最后执行定时任务每间隔一段时间调用一下脚本进行消息的pop处理
### 问题
上面的做法对于实时性要求不高的业务是没有问题的，但是如果要求实时进行消息处理，就有问题了，因为定时任务是每隔一段时间进行处理的，在这一段时间内产生的消息是不能得到及时处理，
需要在定时时间到来的时候才会处理，这时候怎么处理呢，其实redis已经帮我们提供了这样的命令
### 新方案
- brpop
  brpop命令移出并获取列表的右侧第一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止  
- blpop
blpop命令移出并获取列表的左侧第一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
**注意** brpop,blpop都是阻塞命令，也就是说brpop是rpop的阻塞版本，blpop是lpop的阻塞版本，当list中没有数据时,命令会阻塞等待
综上，我们可以用brpop或者blpop来实现实时消费的目的，demo代码如下

```php
<?php
$redis = new Redis();

$redis->connect('127.0.0.1',6379);

while($data = $redis->brpop('test',3600)) {
    var_dump($data);
}
```
上面的例子是，获取key名是test的list的数据，没有数据的话会阻塞3600秒，可是在实际应用中发现，大约1分钟左右脚本就报错并终止了

```shell
Fatal error: Uncaught exception 'RedisException' with message 'read error on connection' in /opt/lnmp/scm/a.php:6
Stack trace:
#0 /opt/lnmp/scm/a.php(6): Redis->brPop('test', 3600)
#1 {main}
  thrown in /opt/lnmp/scm/a.php on line 6

```
查询资料发现：**php的redis扩展是基于php的socket方式实现的，如果php本身配置了socket read超时时间，那么超时会报错退出**
查看php.ini,发现是php.ini文件中的socket默认超时实际配置项导致：

```
default_socket_timeout = 60
```

找到了问题，就好解决了，我们可以通过两种方式修改这个参数
- 修改php.ini文件，一般不建议
- 通过ini_set函数修改，推荐这种方式，只对当前脚本有效，不对其他脚本产生影响
因此上面的例子可以写成下面的方式，真正实现阻塞3600秒:

```php
ini_set('default_socket_timeout', -1); //不超时

$redis = new Redis();

$redis->connect('127.0.0.1',6379);

while($data = $redis->brpop('test',3600)) {
    var_dump($data);
}
```
如果需要无限阻塞，只需要将

```php
$data = $redis->brpop('test',3600)
```
改成

```php
$data = $redis->brpop('test',0)
```