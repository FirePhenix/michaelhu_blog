+++
title = "Let's talk about Go channel"
date = 2024-08-16T15:08:18-04:00
tags = ["Go", "Dev"]
ShowToc = true
TocOpen = true
description= "learning Go Channel without pain"
summary= ""
+++

### What is Channel
If you're a new Go developer, you've likely heard of or used a channel before. But what exactly is a channel, and what is its purpose? Before we delve deeper, let's first discuss this.
So, what is a channel? According to the official Go documentation, a channel is

```
Channels are a typed conduit through which you can send and receive values with the channel operator, <-.

```

basically, you can say that Channel is a communication mechanism. Normaly, we use Channels to send and receive values between goroutines.

For channel, only 4 ways to opreate them.
* Create a channel through ```make()```function.
* Add data to a channel, ``` channel <- data ```.
* Retrieve data from a channel ``` data <- channel ```, Sometimes you might just want to retrieve the data and discard it, so you can write like this ``` <-channel ```.
* close a channel with the ``` close() ``` function.


### Why we need channel

as I montioned before, Channel is the way can send and reciver values betwwen goroutine. so, let's take a very simple exaplam for it, that will help you understand why we need channel.

Imagine your team leader asks you to gather data from two different sources: one from a database and the other from an API endpoint. Once you have all the data, you need to merge it for further processing.

Without using Go channels and goroutines, your code might look something like this:
``` go
dbData := GetDataFromDatabase()
apiData := GetDataFromAPI()
Merge(dbData,apiData)
```
looks ok right? but has a significant drawback: the two tasks (fetching data from the database and the API) are performed sequentially. If the database or API call is slow, your program will waste time waiting for one to complete before starting the other. let's see how a channel can complete this job quicker and more effectively.

``` go
dbChan := make(chan []string)
apiChan := make(chan []string)
// we defined 2 channel here, will use it revice the data
go func() {
		dbChan <- GetDataFromDatabase()
	}()

go func() {
		apiChan <- GetDataFromAPI()
	}()
// Receive results from the channels
dbData := <-dbChan
apiData := <-apiChan
Merge(dbData, apiData)
```
The main goroutine waits to receive data from both channels using the value := <-channel syntax. This operation blocks until data is available, ensuring that the merge operation only happens after both data sources are ready.

### When using it
just like the example, we use the channel communicate bwtween the goroutine, remember, never use Channel to exchange data between different functions in the same goroutine, why? sending or receiving data on a channel is a blocking operation unless the channel is buffered and has space available. If you use a channel within the same goroutine, the goroutine could block indefinitely, waiting for itself to read from or write to the channel, leading to a deadlock.

### Buffered and unbuffered, which one

Channels can be buffered or unbuffered, corresponding to synchronous and asynchronous operations. Let me explain this and help you differentiate between them.

#### unbuffered
define a unbufferd will be like this ``` var ch = make(chan []string) ```. This creates a synchronous channel. An unbuffered channel will block during both write and read operations. Because of blocking behavior, you should use unbuffered channel carefulluy to avoid deadlock and goroutine leakage. You might be wondering what is goroutine leak is, I will explain it in the end of this article as a bonus.

The send operation on an unbuffered channel is not completed until a receiver has received the value; otherwise, it remains blocked. Therefore, the sent data must be read before the send operation can complete.

Generally, it is necessary to use select statements with timeout handling; therefore, you should include a timeout in this context.

Also, don't assume that simply closing a channel will resolve blocking issues. Closing a channel does nothing to prevent blocking if you send data to a channel that isn't read. Even after closing it, the operation can still be blocked.

#### buffered
define a buffered will be like this ``` var ch = make(chan []string,10) ```, this is a buffered channel and buffer size is 10.

A buffered channel functions like a blocking queue. The writing goroutine will block when the queue is full, and the reading goroutine will block when the queue is empty.

With buffering, the write operation returns immediately after the data is written. Buffered channels, compared to their unbuffered counterparts, are less prone to deadlocks.

### Cons and Pros

#### Cons
* Channel may cause loop blocking or coroutine leakage, which is the most important thing to pay attention to.

* Passing pointers in Channel will cause data race/race conditions

* All data passed in the Channel is a copy of the data, which may affect performance. However, based on our current machine performance, the CPU consumption caused by data copying can be ignored in most cases.

#### Pros

* Simplified concurrency management
* Built-in synchronization
* Clear and predictable communication
* Avoidance of shared memory issues
* Supports select statements

### Goroutine leak

What is goroutine leakage? It occurs when a goroutine is initiated and remains active after its task is complete, without being properly terminated. If it becomes useless and still doesn't terminate, it results in a goroutine leak.

A leaked goroutine can consume CPU resources and potentially other system resources, thereby reducing the efficiency of the main goroutine and sometimes even causing exceptions.

### Summary

Channels are powerful tools for managing concurrency in Go. They act as communication mechanisms between goroutines, allowing for safe and efficient data transmission. However, there are some pitfalls you need to be cautious of. If used properly, channels can make your application more efficient and your code cleaner and simpler.

I hope this article helps you understand Go channels thoroughly. If you have any questions, feel free to leave a comment.

