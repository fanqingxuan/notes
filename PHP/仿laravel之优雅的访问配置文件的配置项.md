### PHP
使用过laravel框架的朋友大概都使用过laravel中的config辅助函数，例如config("app.name"),多维数组总能转化成config("key.key1.key2")用点号的形式访问，看着挺舒服，今天我们就仿照laravel实现下

### 定义一个Config类，如下
```php
class Config
{
    private $data;//用来存储数组

    public function __construct($path)
    {
        $this->data = array();
        $this->handle($path);
    }

    public function handle($path)
    {
        $itemlist = glob("{$path}/*.php");//匹配路径下的php文件
        foreach ($itemlist as $filename) {
            $prepend = basename($filename, '.php');
			//将二维数组拼成点号的一维数组
            $item = $this->dot(require_once $filename, $prepend.'.');
            $this->data = array_merge($this->data, $item);
        }
    }
	//二维数组转成点号的一维数组实现
    private function dot($array, $prepend = '')
    {
        $results = array();
        foreach ($array as $key => $value) {
            if (is_array($value) && ! empty($value)) {
                $results = array_merge($results, static::dot($value, $prepend.$key.'.'));
            } else {
                $results[$prepend.$key] = $value;
            }
        }

        return $results;
    }
	
	//对外的访问函数
    public function get($name, $default = null)
    {
        return isset($this->data[$name])?$this->data[$name]:$default;
    }

    public function all()
    {
        return $this->data;
    }
}
```
### 实例化
```php
$config = new Config('./config');//指定配置文件所在目录
```
### 使用
- 获取所有配置项
```php
$config->all();
```
- 获取单个配置项值
```php
$config->get('app.hello', 'default value');
```
第一个参数是配置项名称(文件名.配置名称)，第二个是默认值(如果配置项不存在，则返回默认值)

举例如下:

1. 访问app文件中的name属性,不存在则返回默认值
```
$config->get('app.name','Json博客');
```
2. 访问database文件中的host配置
```
$config->get('database.host');
```
3. 访问app文件中list配置的第一个值
```
$config->get('app.list.0')
```
### 完整实例
```php
$config = new Config('./config');
var_dump($config->get('app.name','Json博客'));
var_dump($config->get('database.host'));
var_dump($config->get('app.list.0'));
```
重点是Config类的实现