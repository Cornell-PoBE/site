---
layout: post
---

## Client-Server Model

# Request-Response

The client-server model partitions tasks or workloads between the providers of a resource or service, called servers, and service requesters, called clients.
A server host runs one or more server programs which share their resources with clients. Clients themselves do not share any of their resources, but request a server's content or service function and starts communication sessions with servers which await incoming requests. The World Wide Web leverages this style of distributed application architecture. 

![Client-Server Architecture]({{ https://en.wikipedia.org/wiki/Client%E2%80%93server_model#/media/File:Client-server-model.svg }}/assets/client-server-model.png)

The relationship itself can be described by saying that the server provides a function or service to one or many clients, who initiate requests for these services. Servers are unique and can be described by the function that they serve. A web server for instance serves web pages while a file server serves files. The sharing of resources with the client is what makes these abstractions: services.

What is important to note is that the service functions as a black-box abstraction where the client does not need to know how the server handles the request and response. As such, the client only needs to understand the input format of the request and the output format of the returned response. This format is defined by an application communication protocol which we will elaborated upon later in the lecture. 

Clients and servers communicate via the exchange of messages in a request-response messaging pattern where the client sends a request and awaits a response from the server. This communication exchange is a subset of inner-process communication which is the method which unique and seperate processes share data. The request-response messaging pattern leveraged by the client-server model is quite ubiquitous and is commonly seen in distributed computing. However, as expected, it is not the only communication model that exists under IPC. We will be studying others as well, for example Pub-Sub. For eager readers, [here](http://www.mdpi.com/1424-8220/12/6/7648/pdf) is an interesting article that reviews both models and their appropriate advantages. 

# Communication-Protocols

As we must be have a common lingustiic symantic and syntax to speak with each other, for computers to communicate they must have a common language and an established set of rules so that both the client and the server know what to expect. 

[Notes on Networking: TCP from CS4410](http://www.cs.cornell.edu/courses/cs4410/2015fa/slides/08-networking.pdf)

