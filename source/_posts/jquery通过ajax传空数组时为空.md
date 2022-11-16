---
title: Jquery通过ajax传空数组时为空
tags:
  - ajax
  - jquery
  - json
categories:
  - - JavaScript
date: 2018-04-23 14:24:04
---
最近工作时发现一个jquery有一个小陷阱，代码如下：

```javascript
var data = {
    'aa':[],
    'bb':'bb'
};
//如果这么写的话，你会发现data中的aa是没有传到后端的
$.ajax({
    url: '__URL__',
    type: 'POST',
    dataType: 'json',
    data: data,
    success: function (data) {
        console.log(data);
    }
});
```

这样写的话，你通过浏览器抓包工具，可以发现，aa是没有传递给后端的。如果遇到后端必须需要aa这个key的情况下，就会出现bug。 

解决的办法是：

```javascript
$.ajax({
    url: '__URL__',
    type: 'POST',
    dataType: 'json',
    data: JSON.stringify(data),
    contentType: 'application/json',
    success: function (data) {
        console.log(data);
    }
});
```

将data通过JSON.stringify来转换成字符串。 

这样即使aa为空，也会传到后端去了。 

当然，为了后端便于解析数据，我建议增加contentType参数，值为"application/json"。 

然后我在后端增加了判断，如果检测到header信息的ContentType 为application/json，就会进行json\_decode转换，这样就方便接收前端提交过来的参数了。代码如下：

```javascript
$get_post = function(){
    if($_SERVER['CONTENT_TYPE']=='application/json'){
        $input = json_decode(file_get_contents('php://input'),true);
    }else{
        $input  =  $_POST;
    }
    return $input;
};
```
