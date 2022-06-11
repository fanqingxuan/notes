### PHP
### 数组

#### 解构赋值[7.1]

- 可以 解构数组，将多个元素提取到单独的变量中

  ```php
  $array = [11, 22, 33];
  // 使用list语法:
  list($a, $b, $c) = $array;
  // 或速记语法:
  [$a, $b, $c] = $array;
  ```

- 跳过元素解构

  ```php
  $array = [11, 22, 33];
  [, , $c] = $array;
  ```

- 基于键的解构

  ```php
  $array = [
      'a' => 1,
      'b' => 2,
      'c' => 3,
  ];
  ['c' => $c, 'a' => $a] = $array;
  ```

#### 展开操作符`...`[5.6]

- 数组可以展开到函数中

  ```php
  function test($a,$b) {
      var_dump($a,$b);
  }
  $arr = [11, 22];
  test(...$arr);
  ```

- 函数可以使用`...`算符自动收集其余变量

  ```php
  function test($a,...$arr) {
      var_dump($a,$arr);
  }
  test(1,2,3,4,5);
  ```

  剩余参数甚至可以类型提示

  ```php
  function test($a,int ...$arr) {
      var_dump($a,$arr);
  }
  test(1,2,3,4,5);
  ```

- 具有数值键的数组也可以展开到新数组中[7.4]

  ```php
  $a = [11,22];
  $b = [33,44];
  var_dump([...$a,...$b]);
  ```

- 使用字符串键展开数组[8.1]

  ```php
  $a = ['one'=>1,'two'=>2];
  $b = [33,'one'=>11];
  var_dump([...$a,...$b]);
  ```

### 闭包

#### 箭头函数[7.4]

短闭包可自动访问外部作用域，并且只允许自动返回单个表达式

```php
$arr = [1,3,4,5,6,7];
$result = array_filter($arr,fn($value)=>$value%2==0);
var_dump($result);
```

#### 可调用对象[8.1]

通过传递`...`作为参数到调用的callable 来生成一个闭包

```php
function test($a,$b) {
    var_dump($a,$b);
}
$foo = test(...);
$foo(b:'bb',a:'aa');
```

### 修饰

#### 类名[8.0]

从 PHP 8 开始，您也可以在对象上使用`::class`

```php
class Test {
}
$test = new Test;
var_dump(Test::class,$test::class);
```

#### 数字值[7.4]

使用 `_` 运算符设置数值的格式

```php
$number = 100_20;
```

#### 尾随逗号[8.0]

几个地方允许尾随逗号:

- 数组
- 函数调用
- 函数定义
- 闭包的 `use` 语句

### Enums(枚举)[8.1]

#### 声明枚举

枚举内置于语言中

```PHP
enum Status
{
    case DRAFT;
    case PUBLISHED;
    case ARCHIVED;
}
var_dump(Status::DRAFT);
```

#### 枚举方法

枚举可以包含方法，也可以具有每个case的字符串或整数值

```php
enum Status: int
{
    case DRAFT = 1;
    case PUBLISHED = 2;
    case ARCHIVED = 3;

    public function color(): string
    {
        return match($this)
        {
            Status::DRAFT => 'grey',
            Status::PUBLISHED => 'green',
            Status::ARCHIVED => 'red',
        };
    }
}

$status = Status::PUBLISHED;
var_dump($status->color());
```

### 异常[8.0]

引发异常现在是一个表达式，这意味着可以在更多位置抛出，例如短闭包或null并合运算符处

```php
$error = fn($message) => throw new Error($message);
$input = $data['input'] ?? throw new Exception('Input not set');
```

### match[8.0]

相比switch， match会直接返回值

```php
$result = match($input) {
        "true" => 1,
        "false" => 0,
        "null" => NULL,
};
```

### null

#### null合并[7.0]

PHP 7 新增加的 NULL 合并运算符（??）是用于执行isset()检测的三元运算的快捷方式。

NULL 合并运算符会判断变量是否存在且值不为NULL，如果是，它就会返回自身的值，否则返回它的第二个操作数

```php
var_dump(isset($a)?$a:'bb');
var_dump($a??'bb');
```

可以使用 null 合并*赋值*运算符在原始变量为 `null`时将值写入原始变量[7.4]

```php
$temp = $a??='hello';
var_dump($temp,$a);
```

#### Nullsafe运算符[8.0]

可能返回`null`的链式方法

```php
$object
?->methodA()
?->methodB()
?->methodC();
```

### 命名参数[8.0]

按名称而不是位置传递参数

```php
function test($name,$age) {
    var_dump($name,$age);
}
test(age:32,name:"json");
```

命名参数还支持数组展开

```php
function test($name,$age) {
    var_dump($name,$age);
}
$arr = [
    'name' => 'json',
    'age'  => 32,
];
test(...$arr);
```

### 构造函数属性提升[8.0]

将构造函数参数加上`public`, `protected` 或 `private`参数，以使其得到提升

```php
class Test {
    public function __construct(
        public string $name,
        private int $age
    )
    {
        
    }
}
```

仍然可以添加构造函数体，并合并提升属性和非提升属性

```php
class Test {
    private $age;
    public function __construct(
        public string $name,
        int $age
    )
    {
        $this->age = $age;
    }
}
```

### 在初始化器中使用new关键字[8.1]

PHP 8.1 允许您在函数或者方法定义中使用 new 关键字作为默认参数

```php
class Test {
    public function say($cls = new stdClass) {
        var_dump($cls);
    }
}
```

### 只读属性[8.1]

- 只读属性必须与类型化属性结合使用
- 只读属性不能有默认值，除非它们是提升的属性
- 一旦设置了只读属性，它就不能再更改了

```php
class Test {
    public readonly string $name;//无默认值
    public function __construct(
        $name,
        public readonly int $age = 34,// 提升的属性
    )
    {
        $this->name = $name;
    }
}
```

