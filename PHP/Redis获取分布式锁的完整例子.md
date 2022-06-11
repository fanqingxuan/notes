### PHP
Redis的分布式锁网上有各种各样的实现方式，就像那句话:错误的方式各有不同，正确的方式只有一个。代码也一样，外层代码封装看着各有不同，其实底层锁的实现方式都是一个思路：

- 使用redis的set或者setnx命令上锁，set命令的话需要加参数NX、EX

- 通过redis提供的eval函数调用lua脚本，以实现解锁的原子性

### 完整源码

下面是加锁和解锁的全部源码

```php
<?php

class RedisLock {

	/**
     * The key of the lock.
     *
     * @var string
     */
	private $key;

	//redis value
	private $value;

	/**
     * The redis object
     *
     * @var Redis
     */
	private $redis;

	/**
     * The number of seconds the lock should be maintained.
     *
     * @var int
     */
	private $seconds;

	public function __construct(Redis $redis, $key, $seconds = 180) {

		$this->redis 	= $redis;
		$this->key   	= $key;
		$this->value 	= uniqid();
		$this->seconds 	= $seconds;

	}

	/**
     * Attempt to acquire the lock.
     *
     * @return bool
     */
	public function acquire() {
		return (bool)$this->redis->set($this->key, $this->value, ['NX', 'EX' => $this->seconds]);
	}

	/**
     * Release the lock.
     *
     * @return bool
     */
	public function release()
    {
        $script = '
            if redis.call("GET", KEYS[1]) == ARGV[1] then
                return redis.call("DEL", KEYS[1])
            else
                return 0
            end
        ';
        return (bool)$this->redis->eval($script, [$this->key, $this->value], 1);
    }
}
```

### 用法

```php
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

$lockObj = new RedisLock($redis,"test");

if($lockObj->acquire()) {
	echo 'do something';
	$lockObj->release();
}
```
