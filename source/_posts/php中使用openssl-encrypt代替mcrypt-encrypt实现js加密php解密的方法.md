---
title: PHP中使用OPENSSL_ENCRYPT代替MCRYPT_ENCRYPT实现JS加密PHP解密的方法
tags:
  - MCRYPT_ENCRYPT
  - OPENSSL_ENCRYPT
  - 加密
  - 解密
id: '177'
categories:
  - - JavaScript
  - - PHP
date: 2017-12-21 15:45:04
---

# 项目背景

*   因为自己开发的接口希望在传递的工程中可以保证参数是密文的形式，主要是前端使用js加密，后端使用php解密
*   在网络上搜索了很多的方法，但是大部分的都是使用mcrypt\_decrypt和mcrypt\_encrypt进行php端的加解密，但是众所周知的问题，这两个方法在php7.1以后将会被废弃，故而采用。

# 实现方式说明

*   php使用mcrypt\_decrypt和mcrypt\_encrypt的组合方式，以及openssl\_decrypt和openssl\_encrypt的组合方式
*   js端使用[Crypto-js](https://github.com/brix/crypto-js/tree/release-3.1.2)
*   为了说明两种方式的区别，在使用mcrypt\_decrypt和mcrypt\_encrypt方式的时候，使用crypto-helper-zeropadding.js来命名自定义的js加密帮助类，使用test\_crypto\_zeropadding.html来命令对应的测试html文件；在使用openssl\_decrypt和openssl\_encrypt的组合方式的时候，使用crypto-helper-pkcs7.js来命名自定义的js加密帮助类，使用test\_crypto\_pcks7.html来命名测试页面 ，详细的区别可以参见 帖子 [https://segmentfault.com/q/1010000009624263](https://segmentfault.com/q/1010000009624263)

### 方法一 ：使用MCRYPT\_DECRYPT和MCRYPT\_ENCRYPT

### PHP加密解密类

```
<?php

class AesJs
{
    /**向量
     * @var string
     */
    const IV = "1234567890123412";//16位
    /**
     * 默认秘钥
     */
    const KEY = '201707eggplant99';//16位

    public static function init($iv = '')
    {
        self::$iv = $iv;
    }

    /**
     * 加密字符串
     * @param string $data 字符串
     * @param string $key 加密key
     * @return string
     */
    public static function encrypt($data = '', $key = self::KEY)
    {
        $encrypted = mcrypt_encrypt(MCRYPT_RIJNDAEL_128, $key, $data, MCRYPT_MODE_CBC, self::$iv);
        return base64_encode($encrypted);
    }

    /**
     * 解密字符串
     * @param string $data 字符串
     * @param string $key  加密key
     * @return string
     */
    public static function decrypt($data = '', $key = self::KEY)
    {
        $decrypted = mcrypt_decrypt(MCRYPT_RIJNDAEL_128, $key, base64_decode($data), MCRYPT_MODE_CBC, self::$iv);
        return rtrim($decrypted, "/0");
    }



    public static function pkcs7_pad($str){
        $len = mb_strlen($str, '8bit');
        $c = 16 - ($len % 16);
        $str .= str_repeat(chr($c), $c);
        return $str;
    }

}
```

### JS端的关键代码

自定义封装的CRYPTO-HELPER-ZEROPADDING.JS

```
var IV = '1234567890123412';

var KEY = '201707eggplant99'
/**
 * 加密
 */
function encrypt(str) {
    key = CryptoJS.enc.Utf8.parse(KEY);// 秘钥
    var iv= CryptoJS.enc.Utf8.parse(IV);//向量iv
    var encrypted = CryptoJS.AES.encrypt(str, key, { iv: iv, mode: CryptoJS.mode.CBC, padding: CryptoJS.pad.ZeroPadding });
    return encrypted.toString();
}
/**
 * 解密
 * @param str
 */
function decrypt(str) {
    var key = CryptoJS.enc.Utf8.parse(KEY);// 秘钥
    var iv=    CryptoJS.enc.Utf8.parse(IV);//向量iv
    var decrypted = CryptoJS.AES.decrypt(str,key,{iv:iv,padding:CryptoJS.pad.ZeroPadding});
    return decrypted.toString(CryptoJS.enc.Utf8);
}
```

JS测试调用页面TEST\_CRYPTO\_ZEROPADDING.HTML

```
<html>
<head>test crypto-js</head>
<script src="crypto-js/aes.js" type="text/javascript"></script>
<script src="crypto-js/md5.js" type="text/javascript"></script>
<script src="crypto-js/components/pad-zeropadding-min.js" type="text/javascript"></script>
<script src="crypto-helper-zeropadding.js" type="text/javascript"></script>

<script type="text/javascript">

var data = '111111';
var encode = encrypt(data);
console.log('encode is =======>'+encode);
var decode = decrypt(encode);
console.log('decode is =======>'+decode);
alert('encode is ====>'+encode+',decode is ====>'+decode);
</script>


</html>
```

## php中使用openssl\_encrypt代替mcrypt\_encrypt实现js加密php解密的方法

### 方法二 ：OPENSSL\_DECRYPT和OPENSSL\_ENCRYPT的组合方式

### PHP加密解密类

```
<?php
/*
+--------------------------------------------------------------------------
   由于在php7.1之后mcrypt_encrypt会被废弃，因此使用openssl_encrypt方法来替换
   ========================================
   by Focus
   ========================================


+---------------------------------------------------------------------------
*/
class OpensslEncryptHelper
{
    /**向量
     * @var string
     */
    const IV = "1234567890123412";//16位
    /**
     * 默认秘钥
     */
    const KEY = '201707eggplant99';//16位

    /**
     * 解密字符串
     * @param string $data 字符串
     * @param string $key 加密key
     * @return string
     */
    public static function decryptWithOpenssl($data,$key = self::KEY,$iv = self::IV){
        return openssl_decrypt(base64_decode($data),"AES-128-CBC",$key,OPENSSL_RAW_DATA,$iv);
    }

    /**
     * 加密字符串
     * 参考网站： https://segmentfault.com/q/1010000009624263
     * @param string $data 字符串
     * @param string $key 加密key
     * @return string
     */
    public static function encryptWithOpenssl($data,$key = self::KEY,$iv = self::IV){

//        echo base64_encode(mcrypt_encrypt(MCRYPT_RIJNDAEL_128, "1234567890123456", pkcs7_pad("123456"), MCRYPT_MODE_CBC, "1234567890123456"));
//        echo base64_encode(openssl_encrypt("123456","AES-128-CBC","1234567890123456",OPENSSL_RAW_DATA,"1234567890123456"));
//        $encrypted = mcrypt_encrypt(MCRYPT_RIJNDAEL_128, $key, $data, MCRYPT_MODE_CBC, self::$iv);
//        return base64_encode(mcrypt_encrypt(MCRYPT_RIJNDAEL_128, "1234567890123456", pkcs7_pad("123456"), MCRYPT_MODE_CBC, "1234567890123456"));
        return base64_encode(openssl_encrypt($data,"AES-128-CBC",$key,OPENSSL_RAW_DATA,$iv));
    }


}
```

### JS端的关键代码

自定义封装的CRYPTO-HELPER-PKCS7.JS

```
var IV = '1234567890123412';

var KEY = '201707eggplant99'
/**
 * 加密
 */
function encrypt(str) {
    key = CryptoJS.enc.Utf8.parse(KEY);// 秘钥
    var iv= CryptoJS.enc.Utf8.parse(IV);//向量iv
    var encrypted = CryptoJS.AES.encrypt(str, key, { iv: iv, mode: CryptoJS.mode.CBC, padding: CryptoJS.pad.Pkcs7});
    return encrypted.toString();
}
/**
 * 解密
 * @param str
 */
function decrypt(str) {
    var key = CryptoJS.enc.Utf8.parse(KEY);// 秘钥
    var iv=    CryptoJS.enc.Utf8.parse(IV);//向量iv
    var decrypted = CryptoJS.AES.decrypt(str,key,{iv:iv,padding:CryptoJS.pad.Pkcs7});
    return decrypted.toString(CryptoJS.enc.Utf8);
}
```

TEST\_CRYPTO\_PCKS7.HTML代码

```
<html>
<head>test crypto-js</head>
<script src="crypto-js/aes.js" type="text/javascript"></script>
<script src="crypto-js/md5.js" type="text/javascript"></script>
<script src="crypto-js/components/pad-zeropadding-min.js" type="text/javascript"></script>
<script src="crypto-helper-pkcs7.js" type="text/javascript"></script>

<script type="text/javascript">

var data = '111111';
var encode = encrypt(data);
console.log('encode is =======>'+encode);
var decode = decrypt(encode);
console.log('decode is =======>'+decode);
alert('encode is ====>'+encode+',decode is ====>'+decode);

</script>


</html>
```