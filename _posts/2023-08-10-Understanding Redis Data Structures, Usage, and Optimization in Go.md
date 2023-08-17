---
layout:     post
title:      Understanding Redis Data Structures, Usage, and Optimization in Go
subtitle:   Delving into Redis
date:       2023-08-10
author:     Minyi
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Redis
    - Distribute System
    - Golang
---

# Read and write in Redis

## Concept

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F26cf923e-dd28-4c82-84db-68bbf347681a_1112x1006.png)

Typically, we utilize MySQL for reading and writing data. But when we need faster access to frequently viewed data due to certain challenges, we consider using Redis. When the server issues a read request, it first accesses Redis. If the corresponding data is not found, it then queries MySQL. When the server issues a write request, it first writes to MySQL, then uses various tools to listen to binlog, subsequently updating Redis

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F534ce024-e9ea-4712-9235-9c138e211c9b_1526x1166.png)

Redis client sends RESP, it will save tables in memory and write those data in disk. Data is saved to the hard drive to prevent data loss upon restart. Incremental data is saved to an AOF (Append Only File). When the server restarts, any unexecuted commands in Redis will be replayed. RDB captures a snapshot of Redis's dataset. Upon restart, the RDB file is read first. Then, it checks if there are any unexecuted commands after the RDB file. If there are, the unexecuted commands from the AOF are loaded and executed.

Redis processes all operation commands using a single thread：

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fff19dd80-aa6b-4501-9170-8ef5a388fff3_1680x500.png)

# String Data Structure in Redis
SDS, or Simple Dynamic String, is a data structure used internally by Redis to represent strings.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff3b05c6a-e2fa-4099-a3dd-325824e21038_1456x902.png)

* An SDS is not just a sequence of characters like a regular C string. Instead, it's a structure that contains metadata about the string.

* This structure contains:
 * The length of the string.
 * The amount of free space (unused bytes) at the end of the string.
 *The actual byte array of characters.

**Advantages:**
**a. Constant Time Length Lookup:**
* Thanks to the length field in the SDS header, Redis can determine the length of an SDS string in constant time (O(1)). In contrast, finding the length of a regular C string would require traversing the entire string until the null-terminator is encountered, which takes linear time (O(n)) for a string of n characters.

**b. Safe against Buffer Overflows:**
* Because the SDS structure keeps track of its own length and allocated space, operations that manipulate the string are less likely to result in buffer overflows compared to traditional C strings, where you might unintentionally overwrite adjacent memory.

**c. Efficient Appends:**
* The free space field allows SDS to perform efficient append operations. If there's enough free space at the end of the string, appending to an SDS can be done without reallocating and copying memory, making it a much faster operation than it would be with traditional strings.

**d. Binary Safe:**
* Unlike C strings, which are null-terminated and can't handle embedded null bytes, SDS strings can store any binary data, including null bytes. This makes them suitable for storing not just text but any binary data.

# Quicklist in Redis

Quicklist is a specialized data structure used internally by Redis to optimize the memory representation of lists. Redis lists can be implemented using either linked lists or ziplists. Both representations have their own pros and cons. To get the best of both worlds, Redis introduced the quicklist, which is essentially a linked list of ziplists.
![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1de10087-c4ee-49cc-a62b-25d8a18420b7_1312x834.png)
Combining doubly linked lists with listpacks, Redis is able to optimize for both memory usage and operational efficiency across different list sizes. The totbytes field tracks the total size of the underlying memory structure, listpack can save more than one element in one entry.

# How to reduce the latency from the client round-trip time

Instead of sending a command and waiting for a response repeatedly, you send multiple commands at once and then read all the responses.
```
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/go-redis/redis/v8"
)

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()

    ctx := context.Background()
    
    // Check the connection
    _, err := rdb.Ping(ctx).Result()
    if err != nil {
        log.Fatalf("Could not connect to Redis: %v", err)
    }

    // Start a pipeline
    pipe := rdb.Pipeline()

    incr := pipe.Incr(ctx, "mycounter")
    pipe.Expire(ctx, "mycounter", time.Hour)

    // Execute all commands in the pipeline
    _, err = pipe.Exec(ctx)
    if err != nil {
        log.Fatalf("Pipeline failed: %v", err)
    }

    fmt.Println(incr.Val())
}
```

# ZSet

When you aim to achieve tens of millions of accesses on a single server, using Redis's zset is an excellent choice. In such scenarios, MySQL might very likely become overwhelmed and crash.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4045e09a-ea52-4cb7-a3c4-077a89979b42_976x1226.png)

1. Hash Table: The hash table stores the elements of the zset as keys and their scores as values. This allows for O(1) time complexity when fetching an element's score.

2. zskiplist (Skip List): The skip list allows for the elements to be ordered by their scores, making range queries efficient. In a skip list, elements are stored in multiple levels, allowing for faster traversal in a sorted manner. When inserting a new element, a "level" for it is chosen randomly, and the element is inserted into all levels up to the chosen one. This random assignment ensures that the skip list remains balanced, providing log(n) time complexity for search, insert, and delete operations.

# Distributed Lock

The SETNX command in Redis is used to set a key's value only if the key does not already exist. The name SETNX stands for "SET if Not exists."

Redis executes commands in a single thread, and the setnx command succeeds only if it has not been set previously.
![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fceb38112-dece-4a0d-b3d1-2a3c377ca180_1124x1176.png)

While setnx can be used as a primitive method to implement distributed locks in Redis, it's not a highly available solution for several reasons:

**No Deadlock Handling:** If the process that acquired the lock crashes or gets delayed, the lock might never be released. This can be mitigated using a timeout with the lock, but handling timeouts correctly can be complex. (Then you will add time out)

**Failover Issues:** In a Redis cluster, if the primary node fails after granting a lock but before replicating it to a secondary node, another client can acquire the same lock from the secondary after failover, leading to two clients holding the lock or no lock simultaneously. Two masters: If vote process gets mistakes, there are two masters in the system.


# Some tips in Redis Design

**Large Key solutions:** To optimize Redis performance, it's advisable to avoid using large keys. 

**Split Value:** When you have a big value associated with a key, instead of storing it as a single large chunk, consider breaking it down:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe381fc08-61ac-4990-b126-fff243412973_1414x460.png)

Before (as depicted by blue): Store the entire value as a single big key.

After: Split the value into smaller chunks and store them in Redis. When retrieving the value, combine these chunks back to get the original value.

This approach not only ensures better memory management and performance in Redis but also offers more granular control over individual chunks. It's a good practice to compress the value before writing it into Redis. Various compression algorithms can be employed for this purpose, including:

*gzip: A widely used compression method.
*snappy: Known for its fast compression and decompression speeds.
*lz4: Offers a balance between speed and compression ratio.

Additionally, if you are storing JSON strings, you might consider using MessagePack, which not only compresses the data but also provides a more efficient serialization format compared to traditional JSON.

**Hash Modulus:** By hashing the key and then taking the modulus of the result, you can divide a large key's associated data across multiple smaller keys or slots. After hashing, using a bitmask can help you extract certain bits from the hash. This can assist in further segmenting your data, allowing you to distribute it across even more keys, further reducing the individual key size.

**Hot-Cold Partitioning:** Large keys often have varying access patterns. Segmenting data based on its access frequency can help manage large keys efficiently. 'Hot' data, or data that is accessed frequently, can be kept in more readily accessible partitions or slots. Meanwhile, 'cold' data, which is less frequently accessed, can be moved to other partitions. This ensures that large data sets don't hinder the performance of frequently accessed data."

##Hot Key solutions:

If some keys are accessed multiple times within a short period in a system like Redis, it indicates that these keys are "hot" in terms of data access patterns. There are several considerations and strategies you can employ to handle such scenarios:

**Caching:** If you're not already using Redis or another caching mechanism (Localcache), then it's time to consider it. For keys that are frequently accessed, caching can significantly improve system performance and reduce the load on primary data stores, e.g., Bitcache in Golang.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F09bb1857-4c73-4ec0-9a6c-3cbe3717b267_1350x624.png)

**Sharding:** If a particular set of keys is frequently accessed and causing a bottleneck, consider sharding your data. By splitting your dataset and distributing it across multiple instances or nodes, you can distribute the load and reduce contention for these hot keys.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F80513bf7-769e-451c-a2a5-edb5b5efd0d6_596x490.png)

**Proxy:**
![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd959872c-ce69-49b7-9238-39af1a26ccd3_1028x854.png)

> Code example: <https://gitee.com/wedone/redis_course> 

