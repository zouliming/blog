---
title: >-
  用EasySwoole向WebSocket端Push消息时报“the connected client of connection[1] is not a
  websocket client”错误
tags:
  - easyswoole
  - swoole
categories:
  - - php
    - Swoole
date: 2018-01-19 11:17:47
---
最近在研究学习Swoole和EasySwoole。 按照EasySwoole中的例子，写了WebsocketController，会响应到一个Html页面，这个Html页面中的Js会连接WebSocket。

```
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>
<body>
<div>
    <div>
        <p>info below</p>
        <ul  id="line">

        </ul>
    </div>
    <div>
        <select id="action">
            <option value="who">who</option>
            <option value="hello">hello</option>
        </select>
        <input type="text" id="says">
        <button onclick="say()">发送</button>
    </div>
</div>
</body>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.js"></script>
<script>
    var wsServer = 'ws://192.168.33.10:9501';
    var websocket = new WebSocket(wsServer);
    window.onload = function () {
        websocket.onopen = function (evt) {
            addLine("Connected to WebSocket server.");
        };

        websocket.onclose = function (evt) {
            addLine("Disconnected");
        };

        websocket.onmessage = function (evt) {
            addLine('Retrieved data from server: ' + evt.data);
        };

        websocket.onerror = function (evt, e) {
            addLine('Error occured: ' + evt.data);
        };
    };
    function addLine(data) {
        $("#line").append("<li>"+data+"</li>");
    }
    function say() {
        var content = $("#says").val();
        var action = $("#action").val();
        $("#says").val('');
        websocket.send(JSON.stringify({
            action:action,
            content:content
        }));
    }
</script>
</html>
```

然后在Event .php中onWorkStart事件中，订阅了Redis的广播，如果收到广播，则向Server推送消息。 

[这个例子其实是参考了韩天峰在SegmentFault中的例子。](https://segmentfault.com/a/1190000010986855?_ea=3137625)

```
$server = new swoole_websocket_server("0.0.0.0", 9501);

$server->on('workerStart', function ($server, $workerId) {
    $client = new swoole_redis;
    $client->on('message', function (swoole_redis $client, $result) use ($server) {
        if ($result[0] == 'message') {
            foreach($server->connections as $fd) {
                $server->push($fd, $result[1]);
            }
        }
    });
    $client->connect('127.0.0.1', 6379, function (swoole_redis $client, $result) {
        $client->subscribe('msg_0');
    });
});

$server->on('open', function ($server, $request) {

});

$server->on('message', function (swoole_websocket_server $server, $frame) {
    $server->push($frame->fd, "hello");
});

$server->on('close', function ($serv, $fd) {

});

$server->start();
```

所以我是这么写的：

```
function onWorkerStart(\swoole_server $server, $workerId)
    {
        // TODO: Implement onWorkerStart() method.  
        //订阅redis广播，给websocket发送消息
        if($workerId==0){
            $client = new \swoole_redis();
            $client->on('message', function (\swoole_redis $client, $result) use ($server) {
                if ($result[0] == 'message') {
                    foreach($server->connections as $fd) {
                        $info = $server->connection_info($fd);
                            //给客户端发送了反馈
                            $server->push($fd, $result[2]);
                    }
                }
            });
            $client->connect('127.0.0.1', 6379, function (\swoole_redis $client, $result) {
                $client->subscribe('test');
            });
        }
      
    }
```

但是，当我另开一个进程，向Redis发布广播时，虽然WebSocket端能收到广播消息，但是Server端后台，却会报一条错误信息： “the connected client of connection\[1\] is not a websocket client” 

后来经过和群主的反复讨论，终于找到了原因。 

韩天峰的Swoole例子里，只开启了一个WebSocket Server，所以向所有Server的connection push 消息没有什么问题。 

但是我的例子里，WebSocket页面是在一个html里，这个html是经过EasySwoole Server响应出来的页面，也就是它经过了EasySwoole里的Server。 

那么这个html中的请求，其实都属于Server中的connection。 

从浏览器的F12里，我们看不到有什么额外的请求，这是因为Favicon.ico请求，在F12里是看不到的。 

但是我们可以在EasySwoole里的beforeWorkerStart事件里绑定server的connect事件，和close事件，打印fd来看一下。

```
function beforeWorkerStart(\swoole_server $server)
{
    $server->on('connect',function($serv,$fd){
        Logger::getInstance()->console("client {$fd} close");
    });
    $server->on("close",function ($ser,$fd){
        Logger::getInstance()->console("client {$fd} close");
    }
    );
}
```

并且在OnRequest事件里将请求打印出来。

```
function onRequest(Request $request, Response $response)
{
    // TODO: Implement onRequest() method.
    var_dump("req:".$request->getUri()->__toString());
}
```

经过我的测试，的确存在favicon.ico的请求，占用了一个connection。 ![](https://i.loli.net/2018/01/19/5a6161d268fa3.png) 原理说一下：html本来是一个http连接，因为websocket连接，转换成了websocket协议。但是favicon.ico还是一个http连接。 

在向$server->connection->push消息的时候，websocket是push成功的，但是http push失败。所以会有报错信息。 

解决办法： 

push消息时，需要判断连接类型，只有成功建立连接的websocket连接，才可以push消息。

```
if($info['websocket_status']){
    //给客户端发送了反馈
    $server->push($fd, $result[2]);
}
```

[至于websocket\_status，可以参照swoole的文档。](https://wiki.swoole.com/wiki/page/413.html)
