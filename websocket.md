> WebSocket:
> 它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于[服务器推送技术]的一种。（前提是保持连接的状态！）
> （1）建立在 TCP 协议之上，服务器端的实现比较容易。
> （2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
> （3）数据格式比较轻量，性能开销小，通信高效。
> （4）可以发送文本，也可以发送二进制数据。
> （5）没有同源限制，客户端可以与任意服务器通信。
> （6）协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。

> 实现单一客户端连接：页面传递过来用户的标识，在onMessage()中判定用户实现单一通讯。

```
ws://example.com:80/some/path

```

##### js部分

```
 <!DOCTYPE html>
 <html>
 <head>
     <title>Java后端WebSocket的Tomcat实现</title>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">     
 </head>
 <body>
     Welcome<br/><input id="text" type="text"/>
     <button onclick="send()">发送消息</button>
     <hr/>
     <button onclick="closeWebSocket()">关闭WebSocket连接</button>
     <hr/>
     <div id="message"></div>
 </body>
 
 <script type="text/javascript">
     var uid = "admin";
     var websocket = null;
     //判断当前浏览器是否支持WebSocket
     if ('WebSocket' in window) {
         websocket = new WebSocket("ws://localhost:8080/WebSocketTest/websocket/"+uid);
     }
     else {
         alert('当前浏览器 Not support websocket')
     }
 
     //连接发生错误的回调方法
     websocket.onerror = function () {
         setMessageInnerHTML("WebSocket连接发生错误");
     };
 
     //连接成功建立的回调方法
     websocket.onopen = function () {
         setMessageInnerHTML("WebSocket连接成功");
     }
 
     //接收到消息的回调方法
     websocket.onmessage = function (event) {
         setMessageInnerHTML(event.data);
     }
 
     //连接关闭的回调方法
     websocket.onclose = function () {
         setMessageInnerHTML("WebSocket连接关闭");
     }
 
     //监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
     window.onbeforeunload = function () {
         closeWebSocket();
     }
 
     //将消息显示在网页上
     function setMessageInnerHTML(innerHTML) {
         document.getElementById('message').innerHTML += innerHTML + '<br/>';
     }
 
     //关闭WebSocket连接
     function closeWebSocket() {
         websocket.close();
     }
 
     //发送消息
     function send() {
         var message = document.getElementById('text').value;
         websocket.send(message);
         
     }
 </script>
 </html>

```

##### java部分

```
package test;
import java.io.IOException;
import java.util.concurrent.CopyOnWriteArraySet;
import javax.websocket.*;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;

/**
 * @ServerEndpoint 注解是一个类层次的注解，它的功能主要是将目前的类定义成一个websocket服务器端,
 *                 注解的值将被用于监听用户连接的终端访问URL地址,客户端可以通过这个URL来连接到WebSocket服务器端
 */
@ServerEndpoint(value = "/websocket/{param}")//{}中的数据代表一个参数，多个参数用/分隔
public class WebSocketTest {
    
    private String uname;
    //
    // 静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
    private static int onlineCount = 0;
    // concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。若要实现服务端与单一客户端通信的话，可以使用Map来存放，其中Key可以为用户标识
    private static CopyOnWriteArraySet<WebSocketTest> webSocketSet = new CopyOnWriteArraySet<WebSocketTest>();
    // 与某个客户端的连接会话，需要通过它来给客户端发送数据
    private Session session;
    /**
     * 连接建立成功调用的方法
     * 
     * @param session
     * 可选的参数。session为与某个客户端的连接会话，需要通过它来给客户端发送数据
     */
    @OnOpen
    public void onOpen(@PathParam(value = "param") String uid, Session session) {
        this.session = session;
        this.uname = uid;
        System.out.println(uid);
        webSocketSet.add(this); // 加入set中
        addOnlineCount(); // 在线数加1
        System.out.println("有新连接加入！当前在线人数为" + getOnlineCount());
    }
    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose() {
        webSocketSet.remove(this); // 从set中删除
        subOnlineCount(); // 在线数减1
        System.out.println("有一连接关闭！当前在线人数为" + getOnlineCount());
    }
    /**
     * 收到客户端消息后调用的方法
     * 
     * @param message
     *            客户端发送过来的消息
     * @param session
     *            可选的参数
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        System.out.println("来自客户端的消息:" + message);
        // 群发消息
        for (WebSocketTest item : webSocketSet) {
            try {
                if (item.uname.equals(this.uname)) {
                    item.sendMessage(item.uname + ":" + message);
                }
            } catch (IOException e) {
                e.printStackTrace();
                continue;
            }
        }
    }
    /**
     * 发生错误时调用
     * 
     * @param session
     * @param error
     */
    @OnError
    public void onError(Session session, Throwable error) {
        System.out.println("发生错误");
        error.printStackTrace();
    }

    /**
     * 这个方法与上面几个方法不一样。没有用注解，是根据自己需要添加的方法。
     * 
     * @param message
     * @throws IOException
     */
    public void sendMessage(String message) throws IOException {
        this.session.getBasicRemote().sendText(message);
        // this.session.getAsyncRemote().sendText(message);
    }
    public static synchronized int getOnlineCount() {
        return onlineCount;
    }
    public static synchronized void addOnlineCount() {
        WebSocketTest.onlineCount++;
    }
    public static synchronized void subOnlineCount() {
        WebSocketTest.onlineCount--;
    }

}


```

> 以上代码来自网络。可以实现前段和后端的相互通讯。

--------------------------------------------------------------分割线------------------------------------------------------------------------

> 而我在项目中，需要后台与另一个服务端（非本服务端）建立连接，并相互通讯。本人初学，总结和参考了网上各位前辈的方法，实现如下：

#### 后台代码：

```
    public String getAccess(String access) {
        System.out.println("dao:"+access);
         Session session = null; 
         WebSocketContainer container = ContainerProvider.getWebSocketContainer();  
         String uri = "ws://localhost:8095//testWebsocket/websocket/userSever/8080"; 
         try {
            session = container.connectToServer(MyClient.class, URI.create(uri));
        } catch (DeploymentException e) {           
            e.printStackTrace();
        } catch (IOException e) {           
            e.printStackTrace();
        }  
         /**发送页面传过来的数据*/
         try {
            session.getBasicRemote().sendText(access);
        } catch (IOException e) {       
            e.printStackTrace();
        }
        String acc = "";
        while(true) {
            acc = MyCache.catche2.get("a1");
            if(acc!=null) {
                return acc;             
            }
             try {
                Thread.sleep(500);
            } catch (InterruptedException e) {              
                e.printStackTrace();
            }
        }
        
        
    }
}

```

```
@ClientEndpoint  
public class MyClient {  
    @OnOpen  
    public void onOpen(Session session) {  
        System.out.println("Connected to endpoint: " + session.getBasicRemote());  
        try {  
            session.getBasicRemote().sendText("Hello");  
        } catch (IOException ex) {  
        }  
    }  

    @OnMessage  
    public void onMessage(String message) {  
        System.out.println(message);  
        MyCache.catche2.put("a1", message);
    }  

    @OnError  
    public void onError(Throwable t) {  
        t.printStackTrace();  
    }  
}

```

#### 远端服务端:

```
/**在远端服务器上建立websocket服务器*/
@ServerEndpoint(value = "/websocket/{relationId}/{userCode}")
public class UserWebSocketMain {
    /**
     * 打开连接时触发
     * @param relationId
     * @param userCode
     * @param session
     */
    @OnOpen
    public void onOpen(@PathParam("relationId") String relationId,
                       @PathParam("userCode") int userCode,
                       Session session){
        System.out.println("连接成功"+relationId+":"+userCode);  
        SessionUtils.put(relationId, userCode, session);
    }

    /**
     * 收到客户端消息时触发
     * @param relationId
     * @param userCode
     * @param message
     * @return
     */
    @OnMessage
    public String onMessage(@PathParam("relationId") String relationId,
                            @PathParam("userCode") int userCode,
                            String message) {
        try {           
                sendMessage("userSever", 8080, message);
        } catch (IOException e) {           
            e.printStackTrace();
        }
        return "Got your message (" + message + ").Thanks !";
        
    }

    /**
     * 异常时触发
     * @param relationId
     * @param userCode
     * @param session
     */
    @OnError
    public void onError(@PathParam("relationId") String relationId,
                        @PathParam("userCode") int userCode,
                        Throwable throwable,
                        Session session) {
        //SessionUtils.remove(relationId, userCode);
        throwable.printStackTrace();
    }

    /**
     * 关闭连接时触发
     * @param relationId
     * @param userCode
     * @param session
     */
    @OnClose
    public void onClose(@PathParam("relationId") String relationId,
                        @PathParam("userCode") int userCode,
                        Session session) {       
        SessionUtils.remove(relationId, userCode);
    }

    public void sendMessage( String relationId,int userCode,String message) throws IOException {
        Session session = SessionUtils.get(relationId, userCode);
        System.out.println(session);
        if(session!=null) {
            session.getBasicRemote().sendText(message);
        }else {
            
        }
        
        // this.session.getAsyncRemote().sendText(message);
    }
    
}

```

```
/**管理连接*/
public class SessionUtils {

    public static Map<String, Session> clients = new ConcurrentHashMap<>();
    private static final AtomicInteger connectionIds = new AtomicInteger();
    

    public static void put(String relationId, int userCode, Session session){
        System.out.println("保存会话"+relationId+":"+userCode);  
        
        clients.put(getKey(relationId, userCode), session);
    }

    public static Session get(String relationId, int userCode){
        System.out.println("取出会话"+relationId+":"+userCode);  
        return clients.get(getKey(relationId, userCode));
    }

    public static void remove(String relationId, int userCode){
        System.out.println("关闭连接"+relationId+":"+userCode);  
        clients.remove(getKey(relationId, userCode));
    }

    /**
     * 判断是否有连接
     * @param relationId
     * @param userCode
     * @return
     */
    public static boolean hasConnection(String relationId, int userCode) {
        System.out.println("判断是否有会话："+relationId+":"+userCode);  
        return clients.containsKey(getKey(relationId, userCode));
    }

    /**
     * 组装唯一识别的key
     * @param relationId
     * @param userCode
     * @return
     */
    public static String getKey(String relationId, int userCode) {
        System.out.println("组装会话唯一标识"+relationId+":"+userCode);  
        return relationId + "_" + userCode;
    }
}

```

> 将两边服务端启动，调用后台的getAccess（）方法就可以建立连接，因为想在getAccess（）中直接拿到响应的数据，所以在MyClient 类中做了一个全局Map（MyCache.catche2），当消息传回调用MyClient 的onMessage（）方法时将数据存入map，再在getAccess（）中用一个轮询得到map的数据（方法很笨，只是不知道还有什么其他方法，见谅！！！）

> 以上可实现远端服务端连接，本人java小白刚参加工作，写此心得希望各位大神指正，对于获取即时数据这一块，我用了map，不知还有其他方法可以实现，跪求！！！！！！