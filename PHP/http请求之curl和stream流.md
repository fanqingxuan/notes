### PHP
### Curl
- get请求
```php
function get($url) {
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    $header = array(
        'Content-Type: application/json',
    );
    curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
    //设置获取的信息以文件流的形式返回，而不是直接输出。  
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  
    if(substr($url, 0, 5) == 'https') {
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
    }
    $output = curl_exec($ch);
    
    if($output === false) {
        $error = curl_error($ch);
        throw new Exception("Curl request error:".$error);
        
    } 
    curl_close($ch);
    return $output;
}
```
- post请求
```php
function post($url, $data, $content_type='') {

    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    
    if($content_type == "json"){
        $contentType = "application/json";
        $post_string = json_encode($data);
    }else{
        $contentType = "application/x-www-form-urlencoded";
        $post_string = http_build_query($data);
    }
    $header = array(
        'Content-Type: '.$contentType,
        'Content-Length: ' . strlen($post_string),
    );
    curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
    //设置获取的信息以文件流的形式返回，而不是直接输出。  
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  
    curl_setopt($ch, CURLOPT_POST, 1);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $post_string);
    if(substr($url, 0, 5) == 'https') {
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
    }
    $output = curl_exec($ch);
    
    if($output === false) {
        $error = curl_error($ch);
        throw new Exception("Curl request error:".$error);
        
    } 
    curl_close($ch);
    return $output;
}
```
注意post请求有两个选项
```php
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, $post_string);
```
**需要特别注意的是，如果你的请求有https请求，最好加下面的代码，表示不验证https证书，不加的话,https请求可能报错**
```php
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE); // https请求 不验证证书和hosts
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);
```

### stream流
```php

function post($url, $data, $content_type ='application/x-www-form-urlencoded') {
    if($content_type == "json"){
        $contentType = "application/json";
        $postdata = json_encode($data);
    }else{
        $contentType = "application/x-www-form-urlencoded";
        $postdata = http_build_query($data);
    }

    $opts = array(
        'http' => array(
            'method' => "POST",
            'header' => "Connection: close\r\n".
                "Content-type: $contentType\r\n".
                "Content-Length: ".strlen($postdata)."\r\n",
            'content' => $postdata,
            'timeout' => 180
        )
        ,'ssl' => array(
            'verify_peer' => false,
            'verify_peer_name' => false,
            'allow_self_signed' => true
        )
    );
    $context = stream_context_create($opts);
    return file_get_contents($url, false,$context);
}
var_dump(post("http://web.demo.com:8899/test.php",['name'=>'hello world'],'json'));

```
注意这里的ssl设置，之前遇到的问题是我们php版本有5.5.3升级到5.6.40,结果发送邮件以及http请求都请求不出去了，查看底层才知道是因为ssl版本的问题，所以加了ssl不验证ssl证书，问题解决了

### 其它
工作中用的最多的就是使用curl发送http请求了，尝试追踪过[guzzle/guzzle](https://github.com/guzzle/guzzle)这个库的源码，她的底层也是用的上面两种方式，不过还有其它请求方式，有兴趣的同学可以去研究下，比如下面几种方式:
- fsockopen
- fopen
- file_get_contents 
