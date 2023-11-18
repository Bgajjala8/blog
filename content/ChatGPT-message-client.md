---
title: "Building a message client with ChatGPT"
date: 2022-12-12T22:46:48-06:00
draft: false
---
As an early career engineer, I have always been fascinated by the vast and diverse world of engineering. One area that has particularly captivated my attention is distributed systems, and I have been eager to learn more about them. Recently, I have been hearing a lot about chat GPT, and I was intrigued by the potential of this approach as a tool for learning about distributed systems.

In order to explore the potential of this approach, I decided to conduct an experiment using chat GPT to prototype a simple distributed chat system. I was curious to see if chat GPT could provide the guidance and assistance needed to prototype such a system, and whether it could be an effective tool for early career engineers like myself.

My goal for this experiment was to create a straightforward single-server, dual-client chat system. I wanted to minimize modifications to the provided code, and see if I could use it as is while still gaining an understanding of how it works, like a plug and play solution.

Initially, I requested that Chat GPT provide me with a diagram of the architecture for a single-server distributed chat system.

*   Can you draw me a simple one server distributed chat system architecture diagram?

```
 +------------+  
                            |            |  
                            |  Client 1  |  
                            |            |  
                            +------+-----+  
                                   |  
                                   |       +------------+  
                                   +------>+            |  
                                           |  Server 1  |  
                                           |            |  
                                           +------------+  
                                                      
                                   +------+--------+  
                                   |            |  
                                   |  Client 2  |  
                                   |            |  
                                   +------------+
```

From the drawing, I can see what ChatGPT was trying to convey. The server would have had the ability to read and send messages, relaying them to the appropriate destinations. Although this diagram wasn’t entirely correct, I decided to use the approach that I thought Chatgpt was suggesting in order to continue.

My next step was to create the server.

*   Can you write me a server in java that listens and relays in a local network?

This is the server chatGPT provided me

**messageRelayServer.java**

```java
 import java.io.*;
 import java.net.*;
 import java.util.*;

 public class Message RelayServer {

     public static void main(String[] args) throws IOException {
         // Create a new server socket on localhost port 1234
         ServerSocket serverSocket = new ServerSocket(1234);
         System.out.println("MessageRelayServer started on localhost:1234");
         // Create a list to store connected clients List<Socket> clients = new ArrayList>();
         // Continuously accept new client connections while (true) {
         // Accept a new client connection
         Socket client = serverSocket.accept();
         // Add the new client to the list of connected clients clients.add(client);
         // Print a message to the console

         System.out.println("New client connected: + client.getRemoteSocketAddress());
             // Create a new thread to handle communication with the client new Thread(() → {
             try {


             }

         }
     }
     // Create input and output streams for the client
     BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
     PrintWriter out = new PrintWriter(client.getOutputStream(), true);
     // Continuously read messages from the client
     while (true) {}
     // Read a message from the client String message = in.readLine();
     // Print the message to the console
     System.out.println("Received message from " + client.getRemoteSocketAddress() +
         +message);
     // Send the message to all other clients, excluding the client that sent the message for (Socket otherClient : clients) {
 }
 if (otherClient client) {}
 out.println(message);
 }
 catch (IOException e) {
     e.printStackTrace();
 }).start();
 }
```

Now that the server is up and running time to create the clients….

*   Now, can you write a client that sends and listens to this server?

The client implementation provided by Chat GPT

**messageRelayClient.java**

```java
import java.io.*;
import java.net.*;
import java.util.logging.*;

public class Message RelayClient {

    private static final Logger logger = Logger.getLogger(MessageRelayClient.class.getName());
    public static void main(String[] args) throws IOException {
        // Create a socket to connect to the server
        Socket socket = new Socket("localhost", 1234);
        // Create input and output streams to read from and write to the server BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream())); PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
        // Listen for messages from the server in a separate thread
        Thread listener = new Thread(()→ {
                try {
                    String response;
                    while ((response = in .readLine()) = null) {}
                    Logger.info("Received message from server: System.out.println(response);
                    }
                    catch (IOException e) {
                        +response);
                    Logger.severe("Error reading from server: + e.getMessage());

                    }
                });
                 Listener.start();
        }

    }
    // Read input from the CLI and write it to the server
    BufferedReader cli = new BufferedReader(new InputStreamReader(System.in));
    String input;
    while ((input = cli.readLine()) = null) {
        out.println(input);
        logger.info("Sent message to server: " + input);
    }
        // Close the socket when done 
        socket.close();
```

Initially, there were several issues with my question. It was not specific enough, and ChatGPt needed more information in order to provide an accurate response. For example, I didn’t mention that I wanted logging or that the client should be connected to port 1234. Additionally, the server code had some errors, such as comparing `Socket` objects instead of their `RemoteSocketAddress` strings, and using the wrong `PrintWriter` to send messages. These issues were addressed in the revised code provided by ChatGPt.

The final code implementation provided by ChatGPt was successful in addressing the issues with the initial code and providing a functional messaging system.

**messageRelayServer.java**

```java
import java.io.\*;  
import java.net.\*;  
import java.util.\*;  
  
public class MessageRelayServer {  
  public static void main(String\[\] args) throws IOException {  
    // Create a new server socket on localhost port 1234  
    ServerSocket serverSocket = new ServerSocket(1234);  
    System.out.println("MessageRelayServer started on localhost:1234");  
  
    // Create a list to store connected clients  
    List<Socket> clients = new ArrayList<>();  
  
    // Continuously accept new client connections  
    while (true) {  
      // Accept a new client connection  
      Socket client = serverSocket.accept();  
  
      // Add the new client to the list of connected clients  
      clients.add(client);  
  
      // Print a message to the console  
      System.out.println("New client connected: " + client.getRemoteSocketAddress());  
  
      // Create a new thread to handle communication with the client  
      new Thread(() -> {  
        try {  
          // Create input and output streams for the client  
          BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));  
          // PrintWriter out = new PrintWriter(client.getOutputStream(), true); -> This was sending it to itself   
  
          // Continuously read messages from the client  
          while (true) {  
            // Read a message from the client  
            String message = in.readLine();  
  
            // Print the message to the console  
            System.out.println("Received message from " + client.getRemoteSocketAddress() + ": " + message);  
  
            // Send the message to all other clients, excluding the client that sent the message  
            for (Socket otherClient : clients) {  
              if (!otherClient.getRemoteSocketAddress().equals(client.getRemoteSocketAddress())) {  
                //Added: -> This now allow to write to other clients   
                PrintWriter otherClientOut = new PrintWriter(otherClient.getOutputStream(), true);  
                otherClientOut.println(message);  
    
              }  
            }  
          }  
        } catch (IOException e) {  
          e.printStackTrace();  
        }  
      }).start();  
    }  
  }  
}
```

**messageRelayClient.java**

```java
import java.io.\*;  
import java.net.\*;  
import java.util.logging.\*;  
  
public class MessageRelayClient {  
    private static final Logger logger = Logger.getLogger(MessageRelayClient.class.getName());  
  
    public static void main(String\[\] args) throws IOException {  
        // Create a socket to connect to the server  
        Socket socket = new Socket("localhost", 1234);  
          
        // Create input and output streams to read from and write to the server  
        BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));  
        PrintWriter out = new PrintWriter(socket.getOutputStream(), true);  
  
        // Listen for messages from the server in a separate thread  
        Thread listener = new Thread(() -> {  
            try {  
                String response;  
                while ((response = in.readLine()) != null) {  
                    logger.info("Received message from server: " + response);  
                    System.out.println(response);  
                }  
            } catch (IOException e) {  
                logger.severe("Error reading from server: " + e.getMessage());  
            }  
        });  
        listener.start();  
  
        // Read input from the CLI and write it to the server  
        BufferedReader cli = new BufferedReader(new InputStreamReader(System.in));  
        String input;  
        while ((input = cli.readLine()) != null) {  
            out.println(input);  
            logger.info("Sent message to server: " + input);  
        }  
  
        // Close the socket when done  
        socket.close();  
    }  
}
```

**Local test**

It took me around 3 hours to get this “built”. It was a fun experiment, and I can see the potential for this technology to help many people understand complex concepts quickly. If I were to write this code from scratch, it would have easily taken me half a day or more. However, even though it would have taken longer, I believe I would have learned more in the process. Despite this, being able to prototype this code and see it work has given me a confidence boost and shown me what to expect in the world of distributed systems.

Although there is much more complexity in distributed systems, with the addition of components such as load balancers, data stores, and message brokers, I can say that using ChatGPt has really helped me gain a glimpse into what the world of a distributed systems engineer may look like. I’m excited to see the future possibilities that this technology brings forward.

Code: [https://github.com/Bgajjala8/chatGPTExperiment](https://github.com/Bgajjala8/chatGPTExperiment)
