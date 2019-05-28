# websocket 相关

##### 1、后端安装

```
pip install websocket-server
```

##### 2、后端代码

```
# coding: utf-8

from websocket_server import WebsocketServer

# Called for every client connecting (after handshake)                          

def new_client(client, server):
        print("New client connected and was given id %d" % client['id'])
        server.send_message_to_all("Hey all, a new client has joined us")

# Called for every client disconnecting                                         

def client_left(client, server):
        print("Client(%d) disconnected" % client['id'])

# Called when a client sends a message                                          

def message_received(client, server, message):
        if len(message) > 200:
                message = message[:200]+'..'
        print("Client(%d) said: %s" % (client['id'], message))

PORT=9001
server = WebsocketServer(PORT, "0.0.0.0")
server.set_fn_new_client(new_client)
server.set_fn_client_left(client_left)
server.set_fn_message_received(message_received)
server.run_forever()
```

##### 3、前端使用

```
<!DOCTYPE HTML>
<html>
   <head>
   <meta charset="utf-8">
   <title>websocket</title>

  <script type="text/javascript">
     function WebSocketTest()
     {
        if ("WebSocket" in window)
        {
           console.log("您的浏览器支持 WebSocket!");
           
           // 打开一个 web socket
           var ws = new WebSocket("ws://192.168.110.82:9001/echo");
           // console.log(ws);
            
           ws.onopen = function()
           {
              // Web Socket 已连接上，使用 send() 方法发送数据
              ws.send("hello world");
              console.log("数据发送中...");
           };
            
           ws.onmessage = function (evt) 
           { 
              var received_msg = evt.data;
              console.log("数据已接收...");
              // console.log(evt);
              console.log(received_msg);
           };
            
           ws.onclose = function()
           { 
              // 关闭 websocket
              console.log("连接已关闭..."); 
           };

           ws.onerror = function(e) {                                              
              output("onerror");                                                  
              console.log(e)                                                      
           };  
        }
        
        else
        {
           // 浏览器不支持 WebSocket
           console.log("您的浏览器不支持 WebSocket!");
        }
     }
  </script>

   </head>
   <body>

  <div id="sse">
     <a href="javascript:WebSocketTest()">运行 WebSocket</a>
  </div>

   </body>
</html>
```

