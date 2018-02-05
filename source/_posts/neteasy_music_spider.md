title: 网易云音乐爬虫 PHP版本
date: 2017-07-31 10:07
categories:
- notes
tags:
- netease
- php
---

# 网易云音乐爬虫 - 爬取评论 - PHP

#### 1、拼接url
```php
/**
* 生成接口url
*/
function crypt_api($song_id, $offset){
    $url = "http://music.163.com/weapi/v1/resource/comments/R_SO_4_{$song_id}/?csrf_token=";
    $first_param = "{rid:\"\", offset:\"{$offset}\", total:\"true\", limit:\"20\", csrf_token:\"\"}"; 
    $forth_param = "0CoJUm6Qyw8W8jud";
    $params = get_params($first_param, $forth_param);
    $encSecKey = get_encSecKey();
    $data = array(
        "params"=> $params,
        "encSecKey"=> $encSecKey
    );
    return array($url, $data);
}
```
#### 2、生成接口的params参数
```php
/*
* 生成接口参数
*/
function get_params($first_param, $forth_param){
    $iv = "0102030405060708";
    $first_key = $forth_param;
    $second_key = 'FFFFFFFFFFFFFFFF';
    $encryptObj = new MagicCrypt();
    $h_encText = $encryptObj->encrypt($first_param,$first_key,$iv);
	$h_encText = $encryptObj->encrypt($h_encText,$second_key,$iv);
    return $h_encText;
}

function get_encSecKey(){
    $encSecKey = "257348aecb5e556c066de214e531faadd1c55d814f9be95fd06d6bff9f4c7a41f831f6394d5a3fd2e3881736d94a02ca919d952872e7d0a50ebfa1769a7a62d512f5f1ca21aec60bc3819a9c3ffca5eca9a0dba6d6f7249b06f5965ecfff3695b54e1c28f3f624750ed39e7de08fc8493242e26dbc4484a01c76f739e135637c";
    return $encSecKey;
}
```

#### 3、AES 加密解密
```php
class MagicCrypt {
    /**
    * @param $encryptStr 需要加密的字符串
    * @param $encryptKey 加密的key
    * @param $iv 向量，增加加密的安全性
    */
    public function encrypt($encryptStr,$encryptKey,$iv) {
        $module = mcrypt_module_open(MCRYPT_RIJNDAEL_128, '', MCRYPT_MODE_CBC, $iv);
        mcrypt_generic_init($module, $encryptKey, $iv);
        //Padding
        $block = mcrypt_get_block_size(MCRYPT_RIJNDAEL_128, MCRYPT_MODE_CBC);
        $pad = $block - (strlen($encryptStr) % $block); //Compute how many characters need to pad
        $encryptStr .= str_repeat(chr($pad), $pad); // After pad, the str length must be equal to block or its integer multiples
        //encrypt
        $encrypted = mcrypt_generic($module, $encryptStr);
        mcrypt_generic_deinit($module);
        mcrypt_module_close($module);
        return base64_encode($encrypted);
    }
     /**
    * @param $encryptStr 需要解密的字符串
    * @param $encryptKey 加密时使用的key
    * @param $iv 加密时使用的向量
    */
    public function decrypt($encryptStr,$encryptKey,$iv) {
        $module = mcrypt_module_open(MCRYPT_RIJNDAEL_128, '', MCRYPT_MODE_CBC, $iv);
        mcrypt_generic_init($module, $encryptKey, $iv);
        $encryptedData = base64_decode($encryptStr);
        $encryptedData = mdecrypt_generic($module, $encryptedData);
        return $encryptedData;
    }
}
```

#### 4、curlPost函数
```php
function _curlPost($url, $curlPost, $cookie_str = '', $header_ary = array()){
    $curl = curl_init();
    curl_setopt($curl, CURLOPT_URL, $url);
    curl_setopt($curl, CURLOPT_HEADER, false);
	curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($curl, CURLOPT_POST, true);
    curl_setopt($curl, CURLOPT_POSTFIELDS, $curlPost);
	if (substr($url,0,5)=='https') {
        curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
        curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false);
    }
    if ($cookie_str) {
        curl_setopt($curl, CURLOPT_COOKIE, $cookie_str);
    }
    if ($header_ary) {
        curl_setopt($curl, CURLOPT_HTTPHEADER, $header_ary);
    }
    $return_str = curl_exec($curl);
    curl_close($curl);
    return $return_str;
}
```
#### 5、请求数据
```php
//http://music.163.com/#/song?id=32507038
get_comment(32507038);

/**
* 获取评论信息
* @param $song_id 歌曲id
*/
function get_comment($song_id){
    $offset = 0;
    $has_more = true;
    while ($has_more) {
        $has_more = false;
        list($url, $data) = crypt_api($song_id, $offset);
        $headers = array(
            'Referer'=>'http://music.163.com/',
            'Cookie'=> 'appver=1.5.0.75771;MUSIC_U=e954e2600e0c1ecfadbd06b365a3950f2fbcf4e9ffcf7e2733a8dda4202263671b4513c5c9ddb66f1b44c7a29488a6fff4ade6dff45127b3e9fc49f25c8de500d8f960110ee0022abf122d59fa1ed6a2;'
            );
        $result = _curlPost($url,http_build_query($data),'',$headers);
        if ($result) {
            $data = json_decode($result);
            if ($data) {
                if ($data->code == 200) {
                    if ($data->comments) {
                        $comments = json_decode(json_encode($data->comments),true);
                        //保存评论信息
                        save_comments($song_id,$comments);
                    }
                    $has_more = $data->more == 'true';
                    //跟get_params里面的limit参数一致
                    $offset += 20;
                }else{
                    handleError("data['code']:$data[code]");	
                }
            }else{
                handleError("json_decode error");
            }
        }else{
            handleError("_curlPost result:null");
        }
        sleep(1);
    }
}
```

参考:

 1. [php实现AES/CBC/PKCS5Padding加密解密（又叫：对称加密）](http://www.cnblogs.com/ygyg/p/5686258.html)
 2. [https://github.com/wenhaoliang/netease-music-spider](https://github.com/wenhaoliang/netease-music-spider)
 3. [http://blog.csdn.net/Ciiiiiing/article/details/62434438](http://blog.csdn.net/Ciiiiiing/article/details/62434438)
