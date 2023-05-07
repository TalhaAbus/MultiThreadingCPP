**When is multithreading is useful?**

- There are basically two seperate situations where you might want to use multithreading.

1. When you are waiting for something to happen external to your computer and you want to execute code whil you are waiting. So you want a synchronous execution.

**Synchronous Execution**, let's say we run a function and we have to wait for it to finish and then we run another function. So the functions are synchronized with each other.

- Multithreading gives us asynchronous execution. This is where functions are not synchronized with each other. They can run at the same time. 

> * Example: pinging remote servers

> * Example: drawing graphics while also processing user input

![image](https://user-images.githubusercontent.com/75746171/236685822-4c21ce89-22da-4e32-a008-c8378fa492f0.png)

- In this system, there are 6 cores. In my central processing unit, there are 6 cores. So that's effectively six processors that can run things at the same time, more or less. Each of those cores actually uses something called "hyper threading", meaning each call can actually run 2 threads at any given time. Which means that in theory I can run 12 seperate threads at the same time on my computer.

**In summary:**
- Multithreading is useful if you want to distribute some heavy processing across multiple cores in your computer, but the you will only find that you get to speed up if you actually do have multiple cores so that your computer can run multiple threads at the same time.










