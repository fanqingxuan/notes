### PHP
## 加锁
### 正确加锁的实现方式
```php
<?
class Lock {
    
    public const SET_IF_NOT_EXIST = "NX";
    public const SET_EXPIRE_TIME = "EX";
    
    /**
     * [getLock 加锁]
     * @param  $redis      [redis对象]
     * @param  $lockKey    [redis key]
     * @param  $requestId  [value]
     * @param  $expireTime [锁的过期时间]
     */
    public static function getLock($redis, $lockKey, $requestId, $expireTime) {
        
        if($redis->set($lockKey, $requestId,[self::SET_IF_NOT_EXIST, self::SET_EXPIRE_TIME=>$expireTime])) {
            return true;
        }
        return false;
    }
}
```
可以看到getLock的实现其实就一行代码
```php
$redis->set($lockKey, $requestId,[self::SET_IF_NOT_EXIST, self::SET_EXPIRE_TIME=>$expireTime])
```
这个方法共有四个参数
- lockKey 锁的key，也就是redis中的key
- requestId 锁存储的value，一般用请求的唯一标识作为value，可以使用uuid库生成，用于解锁时使用，否则的话解锁时区分不了请求
- NX 意思是SET IF NOT EXIST，即当key不存在时，我们进行redis上锁操作；若key已经存在，则不做任何操作
- expireTime 给key设置一个过期时间

执行上面的set()方法就只会导致两种结果：
1. 当前没有锁（key不存在），那么就进行加锁操作，并对锁设置个有效期，同时requestId表示加锁的客户端
2. 已有锁存在，不做任何操作

仔细分析这个set方法，可以发现，这个锁的几个特点
- 互斥性
	set()加入了NX参数，可以保证如果已有key存在，则函数不会调用成功，也就是只有一个客户端能持有锁，满足互斥性
- 无死锁
对锁设置了过期时间，即使锁的持有者后续发生崩溃而没有解锁，锁也会因为到了过期时间而自动解锁（即key被删除）
- 可解锁
将value赋值为requestId，代表加锁的客户端请求标识，那么在客户端在解锁的时候就可以进行校验是否是同一个客户端

### 错误的实现方式
#### setnx组合expire命令
```php
	/**
     * [getLock 加锁]
     * @param  $redis      [redis对象]
     * @param  $lockKey    [redis key]
     * @param  $requestId  [value]
     * @param  $expireTime [锁的过期时间]
     */
    public static function getLock($redis, $lockKey, $requestId, $expireTime) {
        
        if($redis->setnx($lockKey, $requestId)) {
        	$redis->expire($lockKey,$expireTime);
            return true;
        }
        return false;
    }
```
乍一看好像和前面的set()方法结果一样，然而由于这是两条Redis命令，**不具有原子性**，如果程序在执行完setnx()之后突然崩溃，导致锁没有设置过期时间,那么将会发生死锁。

#### 利用客户端时间
```php
	/**
     * [getLock 加锁]
     * @param  $redis      [redis对象]
     * @param  $lockKey    [redis key]
     * @param  $expireTime [锁的过期时间]
     */
    public static function getLock($redis, $lockKey, $expireTime) {
        
        $expireStr = time() + $expireTime;
        if($redis->setnx($lockKey, $expireStr)) {
            return true;
        }
        $redis_value = $redis->get($lockKey);
        if($redis_value && $redis_value<time()) {
        	$old_value = $redis->getset($lockKey,$expireStr);
        	if($old_value && $old_value == $redis_value) {
        		return true;
        	}
        }
        return false;
    }
```
实现过程如下
1. 通过setnx()方法尝试加锁，如果当前锁不存在，返回加锁成功。
2. 如果锁已经存在则获取锁的过期时间，和当前时间比较，如果锁已经过期，则设置新的过期时间，并且返回加锁成功。

这个实现的问题在于
- 依赖于服务器的本地时间，如果分布式，则要求各个机器的时间必须一致
- 若锁过期，多个请求同时执行getset，则只有一个获取锁成功，但是获取锁的这个请求的过期时间可能被其它客户端覆盖
- 锁没有用户标识，任何人都可以解锁

## 解锁
### 正确解锁的实现方式
```php
	/**
     * [getLock 加锁]
     * @param  $redis      [redis对象]
     * @param  $lockKey    [redis key]
     * @param  $requestId  [value]
     */
    public static function releaseLock($redis,$lockKey, $requestId ) {

    	$script = "if redis.call('get', '$lockKey') == '$requestId' then return redis.call('del', '$lockKey') else return 0 end";
        return $redis->eval($script);
    }
```
使用了redis->eval方法来解锁，redis天然集成了lua，eval方法是将lua代码交给redis服务器端执行。
```php
$script = "if redis.call('get', '$lockKey') == '$requestId' then return redis.call('del', '$lockKey') else return 0 end";
```
首先获取锁lockKey对应的value值，检查是否与requestId相等，如果相等则删除锁（解锁）,使用eval的目的是就是实现原子操作

### 错误解锁方式
#### 暴力删除
```php
public static function releaseLock($redis,$lockKey ) {
    return $redis->del($lockKey);
}
```
没有区分客户端，无论是不是自己锁定的，强制解锁了,任何客户端都可以随时进行解锁，即使这把锁不是自己占用的

#### 迷惑性错误
```php
public static function releaseLock($redis,$lockKey,$requestId ) {
	if($redis->get($lockKey) == $requestId) {
		return $redis->del($lockKey);
	}
}
```
这个看上似乎没什么问题，其实仔细分析，还是有点儿问题的，就是没有实现原子性。如果客户端A加锁，一段时间之后客户端A解锁，在A执行del()之前，锁突然过期了，此时客户端B尝试加锁成功，然后客户端A再执行del()方法，则将客户端B的锁给解除了

**关于redis锁的问题，实际应用场景中还有很多方法，各种各样的实现，各种各样的坑，值得我们去分析和研究，只要我们记住始终用原子操作，就不会有太大问题**