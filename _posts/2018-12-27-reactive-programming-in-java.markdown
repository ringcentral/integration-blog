# Reactive programming in Java

Nowdays the notion of reactive programming is becoming more and more popular. So what is reactive programming, and what is the problem it is trying to resolve? Let's get started by the defintion in Wikipedia "Reactive Programming is an asynchronous programming paradigm concerned with data streams and the propagation of change."

### Why do we need the reactive programming?
The simple answer is that we want to make our application more responsive to improve the user experience. There is one important word in the reactive programming definition: asynchronous. You are notified when data is emitted in the stream asynchronously – meaning independently to the main program flow. By structuring your program around data streams, you are writing asynchronous code: you write code invoked when the stream emits a new item. Threads, blocking code and side-effects are very important matters in this context.

In the traditional programming model, there is one dilemma we need to face which is ***Blocking***. The modern applications usually need to accommodate a huge numbers of concurrent users, so the performance is becoming a key conern. In the traditional ways, we often program using blocking code. and when there is a performance bottleneck the normal way we do is introducing additional threads runing with similar blocking code which is very wastefull. Compared to the threads model, the reactive programming is non-blocking, asynchronous and resilient.

### When do we use reactive programming and what is the benefit?
The key expected benefit of reactive and non-blocking is the ability to scale with a small, fixed number of threads and less memory. That makes applications more resilient under load because they scale in a more predictable way. It allows you to treat streams of asynchronous events with the same sort of simple, composable operations that you use for collections of data items like arrays. It frees you from tangled webs of callbacks, and thereby makes your code more readable and less prone to bugs.

### Reactive extension
In software programming, Reactive Extensions is a set of tools allowing imperative programming languages to operate on sequences of data regardless of whether the data is synchronous or asynchronous. It provides a set of sequence operators that operate on each item in the sequence. Like ReactiveX, it's not only a API but provide the library for composing asynchronous and event-based programs by using observable sequences. it includes a set of lanuage implementation like RxJava, RxJS, RxCPP, etc. 

### Introduction to RxJava
There are different implementation on Java for reactive programming, like:
- Java 9 Flow API
- RxJava
- Reactor 

Here I would like to take RxJava as an example. 
To use RxJava in our Maven project, we’ll need to add the following dependency to our pom.xml:
```
<dependency>
    <groupId>io.reactivex</groupId>
    <artifactId>rxjava</artifactId>
    <version>${rx.java.version}</version>
</dependency>
```

