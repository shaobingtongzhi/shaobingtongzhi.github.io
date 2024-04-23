---
title: Springboot 集成 WebSocket
date: 2024-04-23
categories:
  - 学习笔记
  - 01-Spring学习笔记
tags:
  - WebSocket
---

# 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
    <version>2.7.18</version>
</dependency>
```

# 创建 WebSocketConfig 配置类

WebSocketConfig.java

```java
@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240423/Snipaste_2024-04-23_11-21-35.v9nnyb6nlow.webp)

# 创建 WebSocketServer 工具类

WebSocketServer.java

```java
@ServerEndpoint("/websocket") //WebSocket服务连接地址
@Component
public class WebSocketServer {
    //concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象
    private static CopyOnWriteArraySet<WebSocketServer> webSocketSet = new CopyOnWriteArraySet<>();
    //与某个客户端的连接会话，需要通过它来给客户端发送数据
    private Session session;

    @OnOpen
    public void onOpen(Session session) throws IOException {
        this.session = session;
        HashMap<String, String> mess = new HashMap<>();
        mess.put("code","200");
        mess.put("msg","连接成功");
        sendMessage(mess);
        //加入set中
        webSocketSet.add(this);
        System.out.println("websocket open ...");
    }
    @OnClose
    public void onClose() throws IOException{
        System.out.println("websocket close !!!");
        //从set中删除
        webSocketSet.remove(this);
    }
    @OnMessage
    public void onMessage(String mess,Session session){
        System.out.println("received mess: " + mess);
    }
    @OnError
    public void onError(Throwable err){
        System.out.println("出错了！！！");
        err.printStackTrace();
    }
    public static void sendInfo(Map<String,String> message) throws IOException{
        for (WebSocketServer item : webSocketSet) {
            try {
                //这里可以设定只推送给这个sid的，为null则全部推送
                item.sendMessage(message);
            } catch (IOException e) {
                continue;
            }
        }
    }
    /**
     * 实现服务器主动推送
     */
    public void sendMessage(Map<String,String> message) throws IOException {
        String mess = JSONUtil.toJsonStr(message);
        this.session.getBasicRemote().sendText(mess);
    }
}
```

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240423/Snipaste_2024-04-23_11-25-32.6k6z2umn93c0.webp)

# 创建测试接口

```java
 @GetMapping("/testWebSocket")
  public Map testWebSocket() throws IOException{
      HashMap<String, String> mess = new HashMap<>();
      mess.put("code","204");
      mess.put("msg","报警了");
      WebSocketServer.sendInfo(mess);
      return mess;
  }
```

# 测试WebSocketServer

测试地址：http://www.jsons.cn/websocket/

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240423/Snipaste_2024-04-23_11-28-41.4eenoyalrew0.webp)



# 前端测试

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <style type="text/css">
        #connect-container {
            float: left;
            width: 400px
        }

        #connect-container div {
            padding: 5px;
        }

        #console-container {
            float: left;
            margin-left: 15px;
            width: 400px;
        }

        #console {
            border: 1px solid #CCCCCC;
            border-right-color: #999999;
            border-bottom-color: #999999;
            height: 170px;
            overflow-y: scroll;
            padding: 5px;
            width: 100%;
        }

        #console p {
            padding: 0;
            margin: 0;
        }
    </style>
    <script type="text/javascript">
        var ws = null;

        function setConnected(connected) {
            document.getElementById('connect').disabled = connected;
            document.getElementById('disconnect').disabled = !connected;
            document.getElementById('echo').disabled = !connected;
        }

        function connect() {
            var target = document.getElementById('target').value;
            ws = new WebSocket(target);
            ws.onopen = function () {
                setConnected(true);
                log('Info: WebSocket connection opened.');
            };
            ws.onmessage = function (event) {
                log('Received: ' + event.data);
            };
            ws.onclose = function () {
                setConnected(false);
                log('Info: WebSocket connection closed.');
            };
        }

        function disconnect() {
            if (ws != null) {
                ws.close();
                ws = null;
            }
            setConnected(false);
        }

        function echo() {
            if (ws != null) {
                var message = document.getElementById('message').value;
                log('Sent: ' + message);
                ws.send(message);
            } else {
                alert('WebSocket connection not established, please connect.');
            }
        }

        function log(message) {
            var console = document.getElementById('console');
            var p = document.createElement('p');
            p.style.wordWrap = 'break-word';
            p.appendChild(document.createTextNode(message));
            console.appendChild(p);
            while (console.childNodes.length > 25) {
                console.removeChild(console.firstChild);
            }
            console.scrollTop = console.scrollHeight;
        }
    </script>
</head>
<body>
<div>
    <div id="connect-container">
        <div>
            <input id="target" type="text" size="40" style="width: 350px" value="ws://127.0.0.1:8081/websocket"/>
        </div>
        <div>
            <button id="connect" onclick="connect();">Connect</button>
            <button id="disconnect" disabled="disabled" onclick="disconnect();">Disconnect</button>
        </div>
        <div>
            <textarea id="message" style="width: 350px">Here is a message!</textarea>
        </div>
        <div>
            <button id="echo" onclick="echo();" disabled="disabled">send message</button>
        </div>
    </div>
    <div id="console-container">
        <div id="console"></div>
    </div>
</div>
</body>
</html>
```

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240423/Snipaste_2024-04-23_11-48-52.1dpeiksozy0w.webp)

参考链接：https://blog.csdn.net/m0_52208135/article/details/127613516