### PHP
ArrayAccess是PHP提供的一个接口，其作用是**使得你的对象可以像数组一样被访问**

什么意思呢？我们直接看使用方法
```php
<?php
class Container implements ArrayAccess {
        
    private $elements;
    
    public function offsetExists($offset){
        return isset($this->elements[$offset]);
    }
    
    public function offsetSet($offset, $value){
        $this->elements[$offset] = $value;
    }
    
    public function offsetGet($offset){
        return $this->elements[$offset];
    }
    
    public function offsetUnset($offset){
        unset($this->elements[$offset]);
    }
}
$di = new Container();
$di['name'] = 'hello world';

var_dump($di['name'],isset($di['name']));//true true

unset($di['name']);
var_dump($di['name'],isset($di['name']));//null false

?>
```
看到没有我们可以像访问数组一样访问对象，其实很简单,
ArrayAccess接口为我们提供了4个抽象方法，只有我们实现了这几个抽象方法就可以了
```php
ArrayAccess {
/* 方法 */
	abstract public offsetExists ( mixed $offset ) : boolean
	
	abstract public offsetGet ( mixed $offset ) : mixed
	
	abstract public offsetSet ( mixed $offset , mixed $value ) : void
	
	abstract public offsetUnset ( mixed $offset ) : void
}
```

- $di[xxx] 对应调用offsetGet方法

- $di[xxx] = 'yyy' 对应调用offsetSet方法

- isset($di[xxx]) 对应调用offsetExists方法

- unset($di[xxx]) 对应调用offsetUnset方法
