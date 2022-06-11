### PHP
限流最常用于对 API 进行速率限制，或限制用户针对表单进行的尝试次数，以帮助防止暴力攻击。

CI4限流类不会自发地做任何的请求速率限制或对请求进行限流，但却是限流功能得以实现的关键

CI4限流跟Laravel还是不太一样的，Laravel限流其实是熔断，一旦到达限流的阈值，则时间段内一直阻断后面的处理，但是CI4的限流模式是限流常用的方式，采用令牌桶算法，时间段内可以限制流量，实现降级，不会暴力的完全阻断

### 代码
抽取的限流代码如下，限流数据存储依赖于缓存，我只抽取了Redis方式相关的代码，如果你需要用其它的存储方式的话，存储类实现CacheInterface接口即可

```php
<?php

/**
 * Cache interface
 */
interface CacheInterface {

	/**
	 * Attempts to fetch an item from the cache store.
	 *
	 * @param string $key Cache item name
	 *
	 * @return mixed
	 */
	public function get($key);

	//--------------------------------------------------------------------

	/**
	 * Saves an item to the cache store.
	 *
	 * @param string  $key   Cache item name
	 * @param mixed   $value The data to save
	 * @param integer $ttl   Time To Live, in seconds (default 60)
	 *
	 * @return mixed
	 */
	public function save($key, $value, $ttl = 60);
}

/**
 * Redis cache handler
 */
class RedisHandler implements CacheInterface {

	/**
	 * Redis connection
	 *
	 * @var Redis
	 */
	protected $redis;

	/**
	 * Constructor
	 */
	public function __construct($host = '127.0.0.1',$port = 6379) {
		$this->redis = new Redis;
		$this->redis->connect($host,$port);
	}

		/**
	 * Attempts to fetch an item from the cache store.
	 *
	 * @param string $key Cache item name
	 *
	 * @return mixed
	 */
	public function get($key)
	{

		return $this->redis->get($key);
	}


	/**
	 * Saves an item to the cache store.
	 *
	 * @param string  $key   Cache item name
	 * @param mixed   $value The data to save
	 * @param integer $ttl   Time To Live, in seconds (default 60)
	 *
	 * @return mixed
	 */
	public function save($key, $value, $ttl = 60)
	{
		return $this->redis->set($key, $value, $ttl);
	}
}

class RateLimiter
{

	/**
	 * Container for throttle counters.
	 *
	 */
	protected $cache;

	/**
	 * The prefix applied to all keys to
	 * minimize potential conflicts.
	 *
	 * @var string
	 */
	protected $prefix = 'ratelimiter_';


	//--------------------------------------------------------------------

	/**
	 * Constructor.
	 *
	 * @param  type $cache
	 * @throws type
	 */
	public function __construct(CacheInterface $cache)
	{
		$this->cache = $cache;
	}

	//--------------------------------------------------------------------

	/**
	 * Restricts the number of requests made by a single IP address within
	 * a set number of seconds.
	 *
	 * Example:
	 *
	 *  if (! $throttler->check($request->ipAddress(), 60, MINUTE))
	 * {
	 *      die('You submitted over 60 requests within a minute.');
	 * }
	 *
	 * @param string  $key      The name to use as the "bucket" name.
	 * @param integer $capacity The number of requests the "bucket" can hold
	 * @param integer $seconds  The time it takes the "bucket" to completely refill
	 * @param integer $cost     The number of tokens this action uses.
	 *
	 * @return   boolean
	 * @internal param int $maxRequests
	 */
	public function check($key, $capacity, $seconds = 60, $cost = 1)
	{
		$tokenName = $this->prefix . $key;

		$nowTime = time();
		// Check to see if the bucket has even been created yet.
		if (($tokens = $this->cache->get($tokenName)) === null)
		{
			// If it hasn't been created, then we'll set it to the maximum
			// capacity - 1, and save it to the cache.
			$this->cache->save($tokenName, $capacity - $cost, $seconds);
			$this->cache->save($tokenName . 'Time', $nowTime, $seconds);

			return true;
		}

		// If $tokens > 0, then we need to replenish the bucket
		// based on how long it's been since the last update.
		$throttleTime = $this->cache->get($tokenName . 'Time');
		$elapsed      = time() - $throttleTime;

		// Number of tokens to add back per second
		$rate = $capacity / $seconds;

		// Add tokens based up on number per second that
		// should be refilled, then checked against capacity
		// to be sure the bucket didn't overflow.
		$tokens += $rate * $elapsed;
		$tokens  = $tokens > $capacity ? $capacity : $tokens;

		// If $tokens > 0, then we are safe to perform the action, but
		// we need to decrement the number of available tokens.
		if ($tokens > 0)
		{
			$this->cache->save($tokenName, $tokens - $cost, $seconds);
			$this->cache->save($tokenName . 'Time', time(), $seconds);
			return true;
		}
		return false;
	}

}
```

### 使用方法

```php
$cache = new RedisHandler;

$rateLimiter = new RateLimiter($cache);

if(!$rateLimiter->check($_SERVER['REMOTE_ADDR'],30)) {
	echo '限流了';
} else {
	echo '正常访问';
}
```
例子的意思是，限制每个ip的QPS为30/min，因为使用的是令牌桶算法，所以并不是绝对限制每分钟接收30个请求，意思是当达到30个请求每分钟时，会进行降级处理，仍然可以请求，只不过单位请求量降低了

### 限流方法说明

限流方法只有一个check方法，参数一共有4个
- $key (string) – 储存桶的名称
- $capacity (int) – 储存桶中持有的令牌数量
- $seconds (int) – 储存桶完全填满的秒数
- $cost (int) – 此操作将会花费的令牌数量，默认一个

如果可以正常执行则函数返回 TRUE，否则返回 FALSE

调用该 check() 方法时，你要告诉它存储桶的大小， 可以容纳多少令牌以及时间间隔。在默认情况下，每个 check() 的调用请求将会使用1个可用令牌。

假设我们希望限制每个客户端每分钟120请求，则参考代码如下

```php
$cache = new RedisHandler;

$rateLimiter = new RateLimiter($cache);

if(!$rateLimiter->check($_SERVER['REMOTE_ADDR'],120,60)) {
	echo '限流了';
} else {
	echo '正常访问';
}
```
### 算法说明
- 当前key是第一次请求，则校验通过，同时存储当前请求时间和剩余允许请求次数
- 非第一次请求
	1. 获取上次请求时间
	2. 计算当前时间与上次请求时间差=当前时间-上次请求时间
	3. 计算速率=函数参数传递的容量次数(令牌数)/参数传递的时间
	4. 计算产生的新令牌=速率*时间差
	5. 合计令牌数=桶内剩余令牌数目+产生的新令牌
	6. 重置桶内令牌数:若合计令牌数>桶允许的令牌数，则桶内令牌数为桶允许的令牌数，否则为5计算出的合计令牌数
	7. 校验重置后桶内令牌数是否>0，大于0则校验通过，存储当前时间和桶内令牌剩余数，否则校验失败，即认为应该限流了

