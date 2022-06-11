### PHP
### 概念

所谓限流，指的是在指定时间单个用户对某个路由资源的访问次数限制，该功能有两个使用场景，一个是在某些需要验证/认证的页面限制用户失败尝试次数，提高系统的安全性，另一个是避免非正常用户（比如爬虫）对路由的过度频繁访问，从而提高系统的可用性，此外，在流量高峰期还可以借助此功能进行有效的限流。

### 说明
特别喜欢laravel这个优秀的框架，当今大紫大红必有其原因，过于喜欢以至于总喜欢研究laravel框架引入的一些特性，遇到好的特性总想提取出来，以备自己的项目中使用，之所以想把好的特性提取出来一个最重要的原因是我觉得laravel太重量级了，我表示不喜欢复杂的东西，所以想把好的东西按功能一个个提取出来，将来按需使用需要的特性。今天又手抖提取了laravel中的限流组件，下面是源码全部代码，限流依赖于缓存组件，我提取的代码缓存组件是redis和memcache，所以你要想使用的话，麻烦检查服务器安装了redis或者memcache，php安装了phpredis扩展或者memcache扩展，**注意是memcache扩展而不是memcached扩展**

### 源码
```php
<?php

class Cache {

	/**
     * The cache store implementation.
     *
     */
	private $store;

	/**
     * Create a new cache cache instance.
     *
     * @return void
     */
	public function __construct($store) {
		$this->store = $store;
	}

	/**
     * Determine if an item exists in the cache.
     *
     * @param  string  $key
     * @return bool
     */
    public function has($key)
    {
        return ! is_null($this->get($key));
    }

	/**
     * Store an item in the cache if the key does not exist.
     *
     * @param  string  $key
     * @param  mixed  $value
     * @param  int|null  $seconds
     * @return bool
     */
    public function add($key, $value, $seconds = null)
    {
        if ($seconds !== null) {
            if ($seconds <= 0) {
                return false;
            }

            // If the store has an "add" method we will call the method on the store so it
            // has a chance to override this logic. Some drivers better support the way
            // this operation should work with a total "atomic" implementation of it.
            if (method_exists($this->store, 'add')) {
                return $this->store->add($key, $value, $seconds);
            }
        }

        // If the value did not exist in the cache, we will put the value in the cache
        // so it exists for subsequent requests. Then, we will return true so it is
        // easy to know if the value gets added. Otherwise, we will return false.
        if (is_null($this->get($key))) {
            return $this->put($key, $value, $seconds);
        }

        return false;
    }

	/**
     * Increment the value of an item in the cache.
     *
     * @param  string  $key
     * @param  mixed  $value
     * @return int|bool
     */
    public function increment($key, $value = 1)
    {
        return $this->store->increment($key, $value);
    }

	/**
     * Store an item in the cache.
     *
     * @param  string  $key
     * @param  mixed  $value
     * @param  int|null  $seconds
     * @return bool
     */
    public function put($key, $value, $seconds = null)
    {

        if ($seconds === null) {
            return $this->forever($key, $value);
        }

        if ($seconds <= 0) {
            return $this->forget($key);
        }

        $result = $this->store->put($key, $value, $seconds);

        return $result;
    }

    /**
     * Store an item in the cache indefinitely.
     *
     * @param  string  $key
     * @param  mixed  $value
     * @return bool
     */
    public function forever($key, $value)
    {
        $result = $this->store->forever($key, $value);

        return $result;
    }

	/**
     * Retrieve an item from the cache by key.
     *
     * @param  string  $key
     * @param  mixed  $default
     * @return mixed
     */
    public function get($key, $default = null)
    {
 
        $value = $this->store->get($key);

        if (is_null($value)) {
            $value = $default;
        }

        return $value;
    }

	/**
     * Remove an item from the cache.
     *
     * @param  string  $key
     * @return bool
     */
    public function forget($key)
    {
        return $this->store->forget($key);
    }
}


class RedisStore {

	private $redis;
	private $prefix = '';

	public function __construct(Redis $redis,$prefix = '') {
		$this->redis = $redis;
		$this->setPrefix($prefix);
	}
	/**
     * Retrieve an item from the cache by key.
     *
     * @param  string|array  $key
     * @return mixed
     */
    public function get($key)
    {
        $value = $this->redis->get($this->prefix.$key);

        return ! is_null($value) ? $this->unserialize($value) : null;
    }

    /**
     * Store an item in the cache for a given number of seconds.
     *
     * @param  string  $key
     * @param  mixed  $value
     * @param  int  $seconds
     * @return bool
     */
    public function put($key, $value, $seconds)
    {
        return (bool) $this->redis->setex(
            $this->prefix.$key, (int) max(1, $seconds), $this->serialize($value)
        );
    }

    /**
     * Store an item in the cache if the key doesn't exist.
     *
     * @param  string  $key
     * @param  mixed  $value
     * @param  int  $seconds
     * @return bool
     */
    public function add($key, $value, $seconds)
    {
        $lua = "return redis.call('exists',KEYS[1])<1 and redis.call('setex',KEYS[1],ARGV[2],ARGV[1])";

        return (bool) $this->redis->eval(
            $lua, [$this->prefix.$key,'expire', $this->serialize($value),(int) max(1, $seconds)],2
        );
    }

    /**
     * Increment the value of an item in the cache.
     *
     * @param  string  $key
     * @param  mixed  $value
     * @return int
     */
    public function increment($key, $value = 1)
    {
        return $this->redis->incrby($this->prefix.$key, $value);
    }

    /**
     * Store an item in the cache indefinitely.
     *
     * @param  string  $key
     * @param  mixed  $value
     * @return bool
     */
    public function forever($key, $value)
    {
        return (bool) $this->redis->set($this->prefix.$key, $this->serialize($value));
    }

    /**
     * Remove an item from the cache.
     *
     * @param  string  $key
     * @return bool
     */
    public function forget($key)
    {
        return (bool) $this->redis->del($this->prefix.$key);
    }

     /**
     * Set the cache key prefix.
     *
     * @param  string  $prefix
     * @return void
     */
    public function setPrefix($prefix)
    {
        $this->prefix = ! empty($prefix) ? $prefix.':' : '';
    }

    /**
     * Serialize the value.
     *
     * @param  mixed  $value
     * @return mixed
     */
    protected function serialize($value)
    {
        return is_numeric($value) && ! in_array($value, [INF, -INF]) && ! is_nan($value) ? $value : serialize($value);
    }

    /**
     * Unserialize the value.
     *
     * @param  mixed  $value
     * @return mixed
     */
    protected function unserialize($value)
    {
        return is_numeric($value) ? $value : unserialize($value);
    }
	
}

class RateLimiter
{

    /**
     * The cache store implementation.
     *
     */
    protected $cache;

    /**
     * Create a new rate limiter instance.
     *
     * @param  $cache
     * @return void
     */
    public function __construct(Cache $cache)
    {
        $this->cache = $cache;
    }

    /**
     * Determine if the given key has been "accessed" too many times.
     *
     * @param  string  $key
     * @param  int  $maxAttempts
     * @return bool
     */
    public function tooManyAttempts($key, $maxAttempts)
    {
        if ($this->attempts($key) >= $maxAttempts) {
            if ($this->cache->has($key.':timer')) {
                return true;
            }

            $this->resetAttempts($key);
        }

        return false;
    }

    /**
     * Increment the counter for a given key for a given decay time.
     *
     * @param  string  $key
     * @param  int  $decaySeconds
     * @return int
     */
    public function hit($key, $decaySeconds = 60)
    {
        $this->cache->add(
            $key.':timer', time()+$decaySeconds, $decaySeconds
        );

        $added = $this->cache->add($key, 0, $decaySeconds);

        $hits = (int) $this->cache->increment($key);

        if (! $added && $hits == 1) {
            $this->cache->put($key, 1, $decaySeconds);
        }

        return $hits;
    }

    /**
     * Get the number of attempts for the given key.
     *
     * @param  string  $key
     * @return mixed
     */
    public function attempts($key)
    {
        return $this->cache->get($key, 0);
    }

    /**
     * Reset the number of attempts for the given key.
     *
     * @param  string  $key
     * @return mixed
     */
    public function resetAttempts($key)
    {
        return $this->cache->forget($key);
    }

    /**
     * Get the number of retries left for the given key.
     *
     * @param  string  $key
     * @param  int  $maxAttempts
     * @return int
     */
    public function retriesLeft($key, $maxAttempts)
    {
        $attempts = $this->attempts($key);

        return $maxAttempts - $attempts;
    }

    /**
     * Clear the hits and lockout timer for the given key.
     *
     * @param  string  $key
     * @return void
     */
    public function clear($key)
    {
        $this->resetAttempts($key);

        $this->cache->forget($key.':timer');
    }

    /**
     * Get the number of seconds until the "key" is accessible again.
     *
     * @param  string  $key
     * @return int
     */
    public function availableIn($key)
    {
        return $this->cache->get($key.':timer') - time();
    }
}


$redis = new Redis;
$redis->connect('127.0.0.1', 6379);

$redisCache = new RedisStore($redis,'lock');
$cache = new Cache($redisCache);
/**
$memcache = new Memcache;
$memcache->connect('127.0.0.1', 11211);

$memcacheStore = new MemcacheStore($memcache,'lock');
$cache = new Cache($memcacheStore);
**/
$rateLimter = new RateLimiter($cache);

$key = 'hello';
$maxAttempts = 10;
$seconds = 60;

if($rateLimter->tooManyAttempts("hello",$maxAttempts)) {
	var_dump($rateLimter->availableIn($key).'秒后可用');
	throw new Exception("超次数了");
}

$rateLimter->hit($key, $seconds);

var_dump("剩余次数:".$rateLimter->retriesLeft($key,$maxAttempts));

```

### composer安装

```shell
composer require fanqingxuan/ratelimiter
```

### composer方式使用
```php

<?php

require './vendor/autoload.php';

use Json\RateLimiter\Cache;
use Json\RateLimiter\RedisStore;
use Json\RateLimiter\MemcacheStore;
use Json\RateLimiter\RateLimiter;


$redis = new Redis;
$redis->connect('127.0.0.1', 6379);

$redisCache = new RedisStore($redis,'lock');
$cache = new Cache($redisCache);
/**
$memcache = new Memcache;
$memcache->connect('127.0.0.1', 11211);

$memcacheStore = new MemcacheStore($memcache,'lock');
$cache = new Cache($memcacheStore);
**/
$rateLimter = new RateLimiter($cache);

$key = 'hello';
$maxAttempts = 10;
$seconds = 60;

if($rateLimter->tooManyAttempts("hello",$maxAttempts)) {
	var_dump($rateLimter->availableIn($key).'秒后可用');
	throw new Exception("超次数了");
}

$rateLimter->hit($key, $seconds);

var_dump("剩余次数:".$rateLimter->retriesLeft($key,$maxAttempts));


```
