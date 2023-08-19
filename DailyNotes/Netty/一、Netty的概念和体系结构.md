
## 阻塞I/O调用

服务端：
```java
/* Listing 1.1 Blocking I/O example */
public class BlockingIoExample {  

	public void serve(int portNumber) throws IOException {  
        ServerSocket serverSocket = new ServerSocket(portNumber);  
        Socket clientSocket = serverSocket.accept();  
        BufferedReader in = new BufferedReader(  
                new InputStreamReader(clientSocket.getInputStream()));  
        PrintWriter out =  
                new PrintWriter(clientSocket.getOutputStream(), true);  
        String request, response;  
        while ((request = in.readLine()) != null) {  
            if ("Done".equals(request)) {  
                System.out.println("Closing connection");  
                break;  
            }  
            response = processRequest(request);  
            out.println(response);  
        }  
    }  
  
    private String processRequest(String request){  
        return "Processed";  
    }  
  
    public static void main(String[] args) {  
        BlockingIoExample blockingIoExample = new BlockingIoExample();  
        try {  
            blockingIoExample.serve(4200);  
        } catch(IOException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}
```

客户端：

```java
public class ClientExample {  
    private Socket clientSocket;  
    private PrintWriter out;  
    private BufferedReader in;  
  
    @SneakyThrows  
    public void startConnection(String ip, int port) {  
        clientSocket = new Socket(ip, port);  
        out = new PrintWriter(clientSocket.getOutputStream(), true);  
        in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));  
    }  
  
    @SneakyThrows  
    public String sendMessage(String message) {  
        out.println(message);  
        return in.readLine();  
    }  
  
    @SneakyThrows  
    public void close() {  
        in.close();  
        out.close();  
        clientSocket.close();  
    }  
  
    public static void main(String[] args) {  
        ClientExample clientExample = new ClientExample();  
        clientExample.startConnection("127.0.0.1", 4200);  
        for (int i = 0; i < 10; i++) {  
            String response = clientExample.sendMessage("Hello");  
            System.out.println(response);  
        }  
        String response = clientExample.sendMessage("Done");  
        System.out.println(response);  
    }  
}
```

要注意，Client和Server各自的`InputStream`和`OutputStream`都是相对的。


## NIO非阻塞调用

通过`java.nio.channels.Selector`实现轮询非阻塞，示例代码见：[Java NIO - 基础详解 | Java 全栈知识体系](https://pdai.tech/md/java/io/java-io-nio.html)


## Netty核心组件

### Channel

可以理解成“网线”。

### 回调

例如当一个新的连接建立的时候，`ChannelInboundHandlerAdapter`的`channelActive()`会被触发。

### Future

注意下面这段代码：
```java
public class ConnectExample {  
    private static final Channel CHANNEL_FROM_SOMEWHERE = new NioSocketChannel();  
  
    public static void connect() {  
        Channel channel = CHANNEL_FROM_SOMEWHERE; 
        ChannelFuture future = channel.connect(  
                new InetSocketAddress("192.168.0.1", 25));  
        future.addListener(new ChannelFutureListener() {  
            @Override  
            public void operationComplete(ChannelFuture future) {  
                if (future.isSuccess()) {  
                    ByteBuf buffer = Unpooled.copiedBuffer(  
                            "Hello", Charset.defaultCharset());  
                    ChannelFuture wf = future.channel()  
                            .writeAndFlush(buffer);  
                    // ...  
                } else {  
                    Throwable cause = future.cause();  
                    cause.printStackTrace();  
                }  
            }  
        });  
  
    }  
}
```

这里的`channel.connect(ip, port)`是不会阻塞的，连接结果如何都交给后台处理了。
所以Future其实是回调的一个更加精细的版本，一个更具体的实现。

### ChannelHandler

Netty内部有很多开箱即用的事件处理器，用于处理特定的事件（HSF的设计灵感也是基于此），这是责任链模式的优秀实践。