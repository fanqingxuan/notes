### PHP
### 案例

前两天跟金蝶财务系统对接凭证，就涉及了api调用，当我call接口成功后对方会返回金蝶系统生成的凭证Id，在撤销凭证接口时会使用这个Id，生成凭证成功后的结果如下:
```bash
{"data":[{"dindex":0,"number":"0002","orgnumber":"BU-00002","success":true,"id":1195649395178081280}],"success":true,"errorCode":null,"message":null}
```
我现在要解析这个json字符串，获取id存储起来，以备撤销凭证接口使用

当我使用`json_decode`之后竟然并没有获取到我想要的格式，代码如下

```php
$result_str = '{"data":[{"dindex":0,"number":"0002","orgnumber":"BU-00002","success":true,"id":1195649395178081280}],"success":true,"errorCode":null,"message":null}';

$result = json_decode($result_str,true);

var_dump($result);
```

获取到的结果是:
```php
array(4) {
  ["data"]=>
  array(1) {
    [0]=>
    array(5) {
      ["dindex"]=>
      int(0)
      ["number"]=>
      string(4) "0002"
      ["orgnumber"]=>
      string(8) "BU-00002"
      ["success"]=>
      bool(true)
      ["id"]=>
      float(1.1956493951781E+18)
    }
  }
  ["success"]=>
  bool(true)
  ["errorCode"]=>
  NULL
  ["message"]=>
  NULL
}
```
id奇迹般的解析成了float类型，成了科学计数法格式，这并不是我想要的结果，我想获取id的结果为1195649395178081280。

### 原因

之所以出现将json字符串中的1195649395178081280解析成php的float类型，是因为该值已经超过了php数据类型int长度PHP_INT_MAX，所以会自动转换成float类型

### 解决方式
怎么处理这种情况呢，php又没有long类型，好吧，既然用到了json_decode函数，那我看看手册上这个函数有没有其它参数解决这种问题。结果一看，**[官方](https://www.php.net/manual/zh/function.json-decode.php)**还是挺牛掰的，竟然有专门的处理方式.

第四个可选参数JSON_BIGINT_AS_STRING可以完美解决这个问题,那好吧，对json_decode做一个小小的修改，代码如下:
```php

array(4) {
  ["data"]=>
  array(1) {
    [0]=>
    array(5) {
      ["dindex"]=>
      int(0)
      ["number"]=>
      string(4) "0002"
      ["orgnumber"]=>
      string(8) "BU-00002"
      ["success"]=>
      bool(true)
      ["id"]=>
      string(19) "1195649395178081280"
    }
  }
  ["success"]=>
  bool(true)
  ["errorCode"]=>
  NULL
  ["message"]=>
  NULL
}
```
id成了自己想要的数值，只不过类型变成了string，但是对于php弱类型并没有多大影响。最终问题得到解决。

**注意**:官方手册有说明，在php5.4.0版本之后json_decode才加入了JSON_BIGINT_AS_STRING可选项


