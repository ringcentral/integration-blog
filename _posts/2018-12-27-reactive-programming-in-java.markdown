# Reactive programming in Java

Nowdays the notion of reactive programming is becoming more and more popular. So what is reactive programming, and what is the problem it is trying to resolve? Let's get started by the defintion in Wikipedia "Reactive Programming is an asynchronous programming paradigm concerned with data streams and the propagation of change."

### Why do we need the reactive programming?
The simple answer is that we want to make our application more responsive to improve the user experience. There is one important word in the reactive programming definition: asynchronous. You are notified when data is emitted in the stream asynchronously – meaning independently to the main program flow. By structuring your program around data streams, you are writing asynchronous code: you write code invoked when the stream emits a new item. Threads, blocking code and side-effects are very important matters in this context.
