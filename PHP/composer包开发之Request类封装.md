### PHP
自己封装的一个Request类

### 安装
```code
composer require fanqingxuan/request
```
### 使用
```php

require "vendor/autoload.php";

use Json\Request;

$request = new Request;

var_dump($request->isGet());
```

### 封装的方法

- get($name,$defaultValue)   --从REQUEST中获取值，可指定name不存在的默认值
- getClientAddress() --获取客户端ip
- getHeaders() --获取请求头信息
- getHttpHost() --获取域名
- getHTTPReferer() --获取从哪个地址跳转过来的
- getJsonRawBody() --获取非multipart/form-data形式json格式数据
- getMethod() --获取请求方法，包括GET、POST、PUT、DELETE、PATCH等
- getPort() --获取端口
- getPost($name,$defaultValue) --获取POST请求中name的值，不存在则返回defaultValue的值
- getPut($name,$defaultValue) --获取PUT请求中name的值，不存在则返回defaultVlaue的值
- getQuery($name,$defaultValue) --获取GET请求name的值，不存在则返回defalutValue的值
- getRawBody() --获取原始输入流
- getScheme() --获取请求协议
- getServer($name) --获取$_SERVER中name键对应的值
- getServerAddress() --获取服务端ip
- getURI($onlyPath=false) --获取URI,传true不返回get参数部分
- getUserAgent() --获取用户代理
- has($name) --判断$_REQUEST中是否存在某key
- hasPost($name) --判断$_POST中是否有某key
- hasPut($name) --判断PUT请求中是否有某key
- hasQuery($name) --判断$_GET中是否有某key
- isAjax() --判断是否是ajax请求
- isConnect() --判断是否connect请求
- isDelete() --判断是否delete请求
- isGet() --判断是否get请求
- isHead() --判断是否head请求
- isMethod($mehtod) --是否是某种请求，例如isMethod('GET')
- isPatch() --判断是否Patch请求
- isOptions() --判断是否Options请求
- isPost() --判断是否POST请求
- isValidHttpMethod($method) --判断请求method是否http有效的method
- hasFiles() --请求是否包含文件
- getUploadedFiles() --获取上传文件