
### 阻塞I/O调用

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


### NIO非阻塞调用

