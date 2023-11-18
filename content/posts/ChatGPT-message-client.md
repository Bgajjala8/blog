---
title: "Building a message client with ChatGPT"
date: 2022-12-12T22:46:48-06:00
draft: false
---
As an early career engineer, I have always been fascinated by the vast and diverse world of engineering. One area that has particularly captivated my attention is distributed systems, and I have been eager to learn more about them. Recently, I have been hearing a lot about chatGPT, and I was intrigued by the potential of this approach as a tool for learning about distributed systems.

In order to explore the potential of this approach, I decided to conduct an experiment using chatGPT to prototype a simple distributed chat system. I was curious to see if chatGPT could provide the guidance and assistance needed to prototype such a system, and whether it could be an effective tool for early career engineers like myself.

My goal for this experiment was to create a straightforward single-server, dual-client chat system. I wanted to minimize modifications to the provided code, and see if I could use it as is while still gaining an understanding of how it works, like a plug and play solution.

Initially, I requested that chatGPT provide me with a diagram of the architecture for a single-server distributed chat system.

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

From the drawing, I can see what chatGPT was trying to convey. The server would have had the ability to read and send messages, relaying them to the appropriate destinations. Although this diagram wasn’t entirely correct, I decided to use the approach that I thought chatGPT was suggesting in order to continue.

My next step was to create the server.

*   Can you write me a server in java that listens and relays in a local network?

This is the server chatGPT provided me

{{<github-code-snippets 65a51e8550f4412abe5cb628593f38e5>}}

Now that the server is up and running time to create the clients…

*   Now, can you write a client that sends and listens to this server?

The client implementation provided by chatGPT

{{<github-code-snippets 675239a3964bf6e56ed298c309acaec9>}}

Initially, there were several issues with my question. It was not specific enough, and chatGPT needed more information in order to provide an accurate response. For example, I didn’t mention that I wanted logging or that the client should be connected to port 1234. Additionally, the server code had some errors, such as comparing `Socket` objects instead of their `RemoteSocketAddress` strings, and using the wrong `PrintWriter` to send messages. These issues were addressed in the revised code provided by chatGPT.

The final code implementation provided by chatGPT was successful in addressing the issues with the initial code and providing a functional messaging system.

{{<github-code-snippets cfd4febb12f0891f7ae5d9f7bd33cf50>}}

{{<github-code-snippets 6b324297e72ab561782e116403eacaa8>}}

**Local test**

It took me around 3 hours to get this “built”. It was a fun experiment, and I can see the potential for this technology to help many people understand complex concepts quickly. If I were to write this code from scratch, it would have easily taken me half a day or more. However, even though it would have taken longer, I believe I would have learned more in the process. Despite this, being able to prototype this code and see it work has given me a confidence boost and shown me what to expect in the world of distributed systems.

Although there is much more complexity in distributed systems, with the addition of components such as load balancers, data stores, and message brokers, I can say that using chatGPT has really helped me gain a glimpse into what the world of a distributed systems engineer may look like. I’m excited to see the future possibilities that this technology brings forward.

Code: [https://github.com/Bgajjala8/chatGPTExperiment](https://github.com/Bgajjala8/chatGPTExperiment)
