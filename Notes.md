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

**An Example of Basic Multithreading:**
```CPP
#include <iostream>
#include <thread>
#include <chrono>

using namespace std;

void work()
{
    for (int i = 0; i < 10; i++)
    {
        this_thread::sleep_for(chrono::milliseconds(500));
        cout << "Loop " << i << endl;
    }
}
int main()
{
    thread t1(work);
    thread t2(work);

    t1.join();
    t2.join();

    return 0;
}
```

**Problem when threads share data:**

```CPP
#include <iostream>
#include <thread>
#include <atomic> 
#include <chrono>

using namespace std;

int main()
{
    int count = 0;
    const int ITERATIONS = 1E6;

    thread t1([&count]() {
        for (int i = 0; i < ITERATIONS; i++)
        {
            ++count;
        }
        });

    thread t2([&count]() {
        for (int i = 0; i < ITERATIONS; i++)
        {
            ++count;
        }
        });

    t1.join();
    t2.join();

    cout << count << endl;

    return 0;
}
```
- Your are going to find that count is not equal to twice of the value of ITERATIONS. Even though we have incremented in that many times in two loops. We will see different output for each time. 
- So what's happening here is that the threads are interfering with each other's incrementing count. And you'd perhaps think that incrementing an integer is an atomic operation, meaning it happens in one step. But that's not true. 
- What happening here is that we get the integer (count), we are reserving some memory for it in the stack. And to actually increment it, i believe it has to be copied to a register in the CPU. And the it is incremented and it's copied back into the memory where it normally resides, because it's a local variable, will be in the stack. So **essentially** the value of count has to be copied somewhere, and incremented, and then copied back again to where it is supposed to be.
- This is a general problem with multithreading that if you have got multiple threads operating on the same data, they will usually mess up each other's efforts. To fix this, we will use a simple method which is not going to work generally in a general case. But it will work for this simple case.

```CPP
#include <iostream>
#include <thread>
#include <atomic> 
#include <chrono>

using namespace std;

int main()
{
    atomic<int> count = 0;
    const int ITERATIONS = 1E6;

    thread t1([&count]() {
        for (int i = 0; i < ITERATIONS; i++)
        {
            ++count;
        }
        });

    thread t2([&count]() {
        for (int i = 0; i < ITERATIONS; i++)
        {
            ++count;
        }
        });

    t1.join();
    t2.join();

    cout << count << endl;

    return 0;
}
```
- This atomic thing is something that makes operations behave as though they are atomic. Now we actually get 2 million.

- We fixed it temporarily but the general problem is what happens when multiple threads work on shared data and we're going to have to get into things like mutexes to fix that problem.



