---
layout: post
---

## Client-Server Model

# Request-Response

The client-server model partitions tasks or workloads between the providers of a resource or service, called servers, and service requesters, called clients.
A server host runs one or more server programs which share their resources with clients. Clients themselves do not share any of their resources, but request a server's content or service function and starts communication sessions with servers which await incoming requests. The World Wide Web leverages this style of distributed application architecture.

![Client-Server Architecture](
https://upload.wikimedia.org/wikipedia/commons/thumb/c/c9/Client-server-model.svg/500px-Client-server-model.svg.png)

The relationship itself can be described by saying that the server provides a function or service to one or many clients, who initiate requests for these services. Servers are unique and can be described by the function that they serve. A web server for instance serves web pages while a file server serves files. The sharing of resources with the client is what makes these abstractions: services.

What is important to note is that the service functions as a black-box abstraction where the client does not need to know how the server handles the request and response. As such, the client only needs to understand the input format of the request and the output format of the returned response. This format is defined by an application communication protocol which we will elaborated upon later in the lecture.

Clients and servers communicate via the exchange of messages in a request-response messaging pattern where the client sends a request and awaits a response from the server. This communication exchange is a subset of inner-process communication which is the method which unique and seperate processes share data. The request-response messaging pattern leveraged by the client-server model is quite ubiquitous and is commonly seen in distributed computing. However, as expected, it is not the only communication model that exists under IPC. We will be studying others as well, for example Pub-Sub. For eager readers, [here](http://www.mdpi.com/1424-8220/12/6/7648/pdf) is an interesting article that reviews both models and their appropriate advantages.

# Communication Protocols

As we must be have a common lingustiic symantic and syntax to speak with each other, for computers to communicate they must have a common language and an established set of rules so that both the client and the server know what to expect. These sets of rules that describe what a client and server are expecting in there communciation with each other. This "language" and rules are defined by the communication protocol.

The communication protocal is a system of strict rules that would allow a set of communication systems to trasmit information in previously agreed upon well-defined format. There are multiple protocols that are combined to describe a single communication. These group of protocols are combined to make up the protocol suite, where we will be focusing on the protocol stack for that is the software implementation. This protocol stack is a computer networking protocol suite where each layer can be described by a layer. The following figure shows an example protocol stack in increasing rank of abstraction:


| Protocol      | Layer           |
| --------------|-----------------|
| HTTP	        | Application     |
| TCP/UDP       | Transport       |
| IP	          | Internet/Network|
| Ethernet	    | Data Link/Link  |
| IEEE 802.3u	  | Physical        |


In this class, you will only need to understand three protocols within the transport and application layer, at a very high level. However if you would like to dig deeper into networking, definitly check out the networking slides from CS4410, 2015fa, [here](http://www.cs.cornell.edu/courses/cs4410/2015fa/slides/08-networking.pdf).

In this class we will focus on the Internet Protocol (IP) which is a principal protocol communication stack for sending messages, called datagrams, across the network. This routing of messages is fundamental for the internet and is the basis for internetworking. You can review the IP protocol suite [here](https://en.wikipedia.org/wiki/Internet_protocol_suite). The three key ones we will review are User Datagram Protocl (UDP) and Transmission Control Protocol (TCP) from the transport layer, and Hypertext Transfer Protocol (HTTP) from the application layer.

User Datagram Protocol is a protocol defining the method of sending messages or datagrams across an IP network. Datagrams, in essence, are just transfer units structured in header and payload sections that are sent across a packet-switched network. These datagrams provide  connectionless communication by loading important header information like destintation/source and checksums in the header and the data in the payload. You can read more about this in the slides linked [above](http://www.cs.cornell.edu/courses/cs4410/2015fa/slides/08-networking.pdf). One important thing to note is that UDP has no handshaking, or back-and-forth communication, that is necessary for reliability in the network. As such, there is no gauarentee of delivery, ordering, or duplicate protection. If we are to seek such reliability via hand-shaking this can be provided via TCP.

Transmission Control Protocol is a protocol defining the method of sending different types of messages (we will focus on UDP-like datagrams) with the inclusion of a handshaking routine to provide reliable, ordered, and error-checked delivery of datagrams being sent across an IP network. TCP is a fairly complicated and intricate protocol that is well-addressed in these [videos](https://www.youtube.com/playlist?list=PLB7C6Y9smovkPRzGLsa_GaKc5DOvr6UDd), if you are interested in looking into it further, but that is just optional research for this class.

Hyptertext Transfor Protocol is the main application protocol for exchanging and transfering hypertexts (logical links between nodes) that acts as the overarching framework that acts as the request-response protocol in the client-server computing model, that we have described above. Because HTTP is an application layer protocol it leverages the use of some form of underlying and reliable transport layer protoocl like TCP. As such, HTTP treats the lower level layers, like the TCP transport layer, as black boxes that provide a stable network connection across which to communication. HTTP defines defines the methods to indicate the desired action to be performed when the client is making a request and awaiting a response from the server. We will walk through each of the possible requests and the request-response lifecycle.

# HTTP Request-Response Lifecycle
