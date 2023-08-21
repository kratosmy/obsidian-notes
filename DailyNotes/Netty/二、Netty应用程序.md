
## Echo Server

### EchoServerHandler

![[Pasted image 20230820220214.png]]

Netty是一个异步的事件驱动网络框架，由此可见一斑：
```java
public void start() throws Exception {  
    final EchoServerHandler serverHandler = new EchoServerHandler();  
    EventLoopGroup group = new NioEventLoopGroup();  
    try {  
        ServerBootstrap b = new ServerBootstrap();  
        // event driven
        b.group(group)  
            .channel(NioServerSocketChannel.class)  
            .localAddress(new InetSocketAddress(port))  
            // will be triggered when a new connection is established(async)
            .childHandler(new ChannelInitializer<SocketChannel>() {  
                @Override  
                public void initChannel(SocketChannel ch) throws Exception {  
                    ch.pipeline().addLast(serverHandler);  
                }  
            });  
        // will call wait() method  
        ChannelFuture f = b.bind().sync();  
        System.out.println(EchoServer.class.getName() +  
            " started and listening for connections on " + f.channel().localAddress());  
        f.channel().closeFuture().sync();  
    } finally {  
        group.shutdownGracefully().sync();  
    }  
}
```
