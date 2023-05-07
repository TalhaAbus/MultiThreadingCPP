**When is multithreading is useful?**

- There are basically two seperate situations where you might want to use multithreading.

1. When you are waiting for something to happen external to your computer and you want to execute code whil you are waiting. So you want a synchronous execution.

**Synchronous Execution**, let's say we run a function and we have to wait for it to finish and then we run another function. So the functions are synchronized with each other.

- Multithreading gives us asynchronous execution. This is where functions are not synchronized with each other. They can run at the same time. 

> * Example: pinging remote servers

> * Example: drawing graphics while also processing user input

