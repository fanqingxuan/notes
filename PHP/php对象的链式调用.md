### PHP
链式调用给使用者非常好的体验,很多优秀的框架中都有提供这样的接口,使用过Laravel框架的应该都使用过类似下面的代码：

```php
DB::table('lgt_good')->select(['lgid'],'name')->where('lgid>44')->get()->toArray();
```

其实这个很简单，借助$this关键字返回实例对象，我们写个demo代码如下:

```php
<?php

class Db {
    
    private static $_instance = null;
    private $_where = array();
    private $_fields = null;
    private $_table = null;
    
    private function __construct(){}
    
    private function __clone(){}
    
    //单例模式
    public static function getInstance() {
        if (!self::$_instance instanceof self) {
            self::$_instance = new self();
        }
        return self::$_instance;
    }
    
    public function table($table) {
        $this->_table = $table;
        return $this;
    }
    
    public function where($where) {
        $this->_where = $where;
        return $this;
    }
    
    public function fields($fields) {
        $this->_fields = $fields;
        return $this;
    }
    
    public function getAll() {
        $sql = 'SELECT ';
        if(is_array($this->_fields)) {
            $sql .= implode(',',$this->_fields);
        } else {
            $sql .= $this->_fields;
        }
        $sql .= ' FROM '.$this->_table;
        $sql .= ' WHERE '.$this->_where;
        return $sql;
    }
}
```
结合单例模式，进行类的实例化,使用方式如下:
```php
Db::getInstance()
    ->table('users')
    ->fields(['id','username','create_time'])
    ->where("age>23 and salary>4000")
    ->getAll();
```