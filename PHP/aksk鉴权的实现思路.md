### PHP
### 什么是鉴权
鉴权是一种用于在网络通信中对试图访问服务的请求或用户进行权限认证的方法

### 鉴权使用场景
鉴权通常用于跨系统或者项目进行**重要数据**交互的场景

### 鉴权的作用
- 对请求进行身份验证
- 防止非法者劫持数据，对数据进行篡改
- 防止恶意攻击
- 防止不怀好意的人获取系统重要数据

### 什么是AKSK
AKSK是鉴权过程中的一对秘钥对，通常成对出现
AK即Access Key，用户的唯一标识
SK即Secret Key，用于加密认证的秘钥字符串，必须保密，不能泄露，鉴权过程中不在网络中传输，若SecretKey被恶意第三方窃取，可能导致非常严重的数据泄漏风险

### 鉴权过程
鉴权需要双方按照一定的规则进行数据交互传输，请求除了业务过程中必要的参数外，客户端在请求时还需要添加额外的参数以完成鉴权
- timestamp:当前请求时间戳
- signature:按照双方约定的算法，由客户端生成的签名串，加密算法中需要包含SK,通常用2-5个参数值进行加密
- accessKey:服务端提供给客户端的公钥，服务端用于查询对应的SK秘钥，然后按照相同的算法进行加密

### 服务端鉴权步骤
1. 根据业务对请求参数进行非空校验
2. 校验请求时间是否超时
3. 根据请求参数的公钥获取分配给该客户的秘钥，不存在则提示非法
3. 对请求参数拼接组装，按照双方约定的算法进行加密生成签名串
4. 判断客户端传递的签名串和服务端生成的签名串是否一致，不一致则提示非法
5. 校验通过，进行业务处理

### 举例说明
假如一个客户端要获取服务端一段时间内新增用户的信息，双方规定好了业务参数包括
startTime:开始时间，如2019-11-1
endTime:结束时间，若2019-11-31
type:查询类型，1用户信息列表 2企业信息列表
签名串signature的生成算法是:md5(startTime+md5(endTime)+SK+type)
请求方式是GET请求

那在请求时需要加上我们之前说的三项timestamp、signature、accessKey

服务端的鉴权代码应该类似是下面的样子
```php
    $arrInput = array(
        'startTime' =>  $_GET['startTime'],
        'endTime'   =>  $_GET['endTime'],
        'type'      =>  $_GET['type'],
        'timestamp' =>  $_GET['startTime'],
        'signature' =>  $_GET['startTime'],
        'accessKey' =>  $_GET['accessKey'],
    );
    $ret = array(
        'errCode'   =>  '',
        'data'      =>  array(),
        'errMsg'    =>  '',
    );
    if(strtotime($arrInput['startTime']>=strtotime($arrInput['endTime'])) {
        $ret['errCode'] = 3001;
        $ret['errMsg']  = '开始时间不能大于结束时间';
        echo json_encode($ret);
        die;
    }
    if($arrInput['type'] !=1 && $arrInput['type'] != 2) {
        $ret['errCode'] = 3001;
        $ret['errMsg']  = '查询类型不对';
        echo json_encode($ret);
        die;
    }
    //为了防止客户端与服务器时间不同步而导致的认证失败，引入5分钟的宽松系数
    if(time()-$arrInput['timestamp']>300) {
        $ret['errCode'] = 3001;
        $ret['errMsg']  = '请求超时';
        echo json_encode($ret);
        die;
    }
    $dbSecretKey = $db->get($arrInput['accessKey']);
    if(empty($dbSecretKey)) {
        $ret['errCode'] = 3002;
        $ret['errMsg']  = '非法攻击';
        echo json_encode($ret);
        die;
    }
    $signature = md5($arrInput['startTime'].md5($arrInput['endTime']).$dbSecretKey.$arrInput['type']);
    if($signature != $arrInput['signature']) {
        $ret['errCode'] = 3003;
        $ret['errMsg']  = '鉴权失败';
        echo json_encode($ret);
        die;
    }
    //业务处理，查询db返回数据
    $data = $db->getData($arrInput['startTime'],$arrInput['endTime'],$arrInput['type']);
    $ret['data']  = $data;
    echo json_encode($ret);
    die;
```
看下上面的代码，基本上就是一个鉴权认证的思路，可以理一下代码，如果请求被非法劫持，修改任何一个参数都不会获取到数据，如果不修改参数也只能5分钟内可以获取一段时间内数据