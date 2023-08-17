---
layout:     post
title:      End-to-end Protocols (Remote Procedure Call)
subtitle:   RPC concept
date:       2023-08-02
author:     Minyi
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - RPC
    - Distribute System
    - Golang
---

# Introduction 

## Concept

You have the option to create your own RPC framework, which is in line with the approach taken by the CMU lab’ s requirement:

<https://www.andrew.cmu.edu/course/14-736/index/labs_index.html>

For this lab, I designed a client that allows developers to interact with objects stored on remote servers and receive replies from methods implemented in the server. The library enables developers to invoke remote methods on server objects and get return values in a method. I've created a server that calls the NewService and server start functions in the remote library using a for-loop and checks the server's status at the end of each iteration. The server will implement all interface methods and provide the results to the client.

The server creates a service object, which accepts a server object and the server interface as input parameters. The server interface explains the methods that the client can call from a remote location, whereas the server object contains the actual server name. The `NewService` function initializes the service object and starts listening for incoming connections on a given port. The `Start` method accepts connections and the `Stop` method ends the service. 

The client uses the `StubFactory()` function to construct a stub object that corresponds to the server interface. This function accepts the stub object, the server address, and two boolean values. After the stub object is constructed, the client may use its methods to remotely call the server object's equivalent methods. However, in order to present the responses better I provide considerable log print and hardcode the logics. The client application initiates a request to the server to determine whether John is a student, using the method `IsStudent`, with a string argument and a boolean response. The client also requests TA records by calling `ReturnTARecords`, which returns a map. Next, the client registers a person named John using the method `Register`, which takes a Person object as an argument and returns a boolean value. Finally, the client creates a student record for John using the `NewCSStudent` method, which requires a Person object and returns a Student object.

To implement a basic RPC framework in go is easy but when it comes to optimize code would be complicated. 

Furthermore, if RPC catches your interest, feel free to review this paper as well.

<https://web.eecs.umich.edu/~mosharaf/Readings/RPC.pdf>

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F95390b73-c031-4e17-91a5-d902d61bdf90_1132x500.png)

During the RPC runtime, when a user invokes a function locally, the user stub will encapsulate the message and send it to the callee. Upon receiving the message, the server-stub will extract and execute the corresponding function, then send the response back to the caller.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc5b72bf5-a5ff-420b-a597-df0fcb97a476_640x422.png)

Compared to local function calls, RPC calls need to address the following issues:

1.  Function mapping
1.  Data conversion to byte streams
1.  Network transmission

The architecture of Thrift looks like this:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe99d548b-71db-497f-acd0-e707041f06e3_1272x678.png)

reference: <https://sunchao.github.io/posts/2015-09-22-understanding-how-thrift-works.html>


## Processor & Protocol

Both the client and server utilize the same Interface Definition Language (IDL) file, allowing it to support code generation in various programming languages.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F18f1e84d-3fcc-4b28-b547-548c8ba5c4d2_1494x750.png)

**Some important points:**

Language-specific formats, such as `java.io.Serializable`, are designed for a particular programming language or platform.

Text formats, such as JSON, XML, CSV, etc., are human-readable and widely used for data interchange between systems.

Binary encodings, like Thrift's BinaryProtocol and Protobuf, offer more compact and efficient representations. Implementations may vary, such as TLV (Tag-Length-Value) encoding and Varint encoding.

## Protocol

1.  LENGTH (32 bits): Represents the size of the remaining part of the data packet in bytes, excluding the length of the LENGTH field itself.
1.  HEADER MAGIC (16 bits): Set to the value 0x1000, serving as an identifier for the protocol version, enabling quick verification during protocol parsing.
1.  FLAGS (16 bits): A reserved field currently unused, with the default value of 0x0000.
1.  SEQUENCE NUMBER (32 bits): Indicates the sequence ID of the data packet, allowing for multiplexing, and should ideally be monotonically increasing within a single connection.
1.  HEADER SIZE (16 bits): Equals the number of header bytes divided by 4. The header length is calculated from the 14th byte until just before the PAYLOAD section. (Note: The maximum length of the header is 64K).
1.  PROTOCOL ID (uint8 encoding): Can have two values: - ProtocolIDBinary = 0 - ProtocolIDCompact = 2
1.  NUM TRANSFORMS (uint8 encoding): Represents the number of TRANSFORMs.
1.  TRANSFORM ID (uint8 encoding): Indicates the compression method, such as zlib or snappy.
1.  INFO ID (uint8 encoding): Specific values are defined in the subsequent sections and used to transmit custom meta-information.
1.  PAYLOAD: Contains the actual message content.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe59126db-283e-41c4-805d-d279815d2f31_2212x248.png)

Retrieve the **MagicNumber**, which corresponds to the **HEADER MAGIC** and indicates the protocol type. Next, determine the payload type using the **PayloadCodec**, and finally, decode the payload accordingly.

## Transport

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3ee1ae55-bf46-4112-9791-312e3978ce6a_1386x1088.png)

Use socket API to implement the transport layer, here is the layer model:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F22020001-1abe-42ee-bd6e-ee78e7480a59_904x468.png)

## How to optimize RPC

### Stability

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F29c62f08-a72e-4668-9e89-2324abef019a_2468x1100.png)

send backup request when timeout

Left one is normal request, right one is using backup request

### Scalability

Middleware: Middleware will be constructed into an ordered chain of calls, executed one by one, for tasks such as service discovery, routing, load balancing, timeout control, etc. Option: Used as initialization parameters. The core layer is extensible: encoding, protocols, and network transport layer. The code generation tool also supports plugin extensions.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc1b9cc42-cc24-414d-a08c-741d22cc081e_1180x1210.png)

Other idea:

Reuse goroutine: create goroutine pool.

> Useful link: https://juejin.cn/post/7192268132470227000?searchId=20230803034702729AC52C84222F00F505

> Here is my substack link: <https://open.substack.com/pub/minyizou/p/end-to-end-protocols-remote-procedure?r=2k1xwy&utm_campaign=post&utm_medium=web>
