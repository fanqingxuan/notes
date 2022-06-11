### PHP
- json_encode一个从整数0开始的连续数组，返回的是一个数组结构，否则返回对象结构 
- 另:空数组返回的结构也是类似js的数组结构

```
$arr = array(1,2,3);
echo json_encode($arr);//输出[1,2,3]
echo "<br/>";

$arr = array(1,2,'3'=>3);
echo json_encode($arr);//输出{"0":1,"1":2,"3":3}
echo "<br/>";

echo json_encode(array());//输出[]
echo "<br/>";

$obj = new StdClass();
echo json_encode($obj);//输出{}

echo "<br/>";
echo json_encode(array(),JSON_FORCE_OBJECT);//输出{}
echo "<br/>";
```