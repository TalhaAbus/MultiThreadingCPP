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

## Mutex

```CPP
#include <iostream>
#include <thread>
#include <atomic>
#include <chrono>
#include <mutex>

using namespace std;

int main()
{
    int count = 0;
    const int ITERATIONS = 1E6;

    mutex mtx;

    auto func = [&]() {
        for (int i = 0; i < ITERATIONS; i++)
        {
            mtx.lock();
            ++count;
            mtx.unlock();
        }
    };

    thread t1(func);
    thread t2(func);

    t1.join();
    t2.join();

    cout << count << endl;

    return 0;
}
```
- What is going on here? The mutex here stops more than one thread at a time from accessing this variable between lock and unclok here. (++count). Any code that's in here we call a critical sention. The point about critical section is that only one thread can access it at a time. So only one thread can lock the mutex at a given time, if another thread tries to lock the mutex when it is already locked, second thread will just wait until the mutex is unlcoked and then the second thread will be able to lock the mutex.

=================================
**What happens if the code in critical section throws and exception?**
- Then the mutex never get unclocked. So for that reason, it's more common to use **unique lock** or **lock guards**. 

## Lock Guards
- Using mutex this diretcly is not ideal because you could get and exception thrown in the critical section and then it will never unlock. So for that rason we prefer use our RAII (Resource Acquisition is Initialization) idiom. 
- Idea is that if you want to acquire some resource, so in this case, we want to acquire a lock, you do it by initializing some variable. And then if that variable should go out of scope for any reason, even if an exception is thrown, then it will release the resource or it can be made to do that. So we are going to use lockguard instead of mutex. And to do that, we just declare here a **lock_guard** (this is actually a template type and you have to see what kind of mutex you are expected to wrap or to work with) 

```CPP
#include <iostream>
#include <thread>
#include <atomic>
#include <chrono>
#include <mutex>

using namespace std;

void work(int& count, mutex& mtx)
{
    for (int i = 0; i < 1E6; i++)
    {
        lock_guard<mutex> guard(mtx);
        ++count;
    }
}

int main()
{
    int count = 0;

    mutex mtx;

    thread t1(work, ref(count), ref(mtx));
    thread t2(work, ref(count), ref(mtx));

    t1.join();
    t2.join();

    cout << count << endl;

    return 0;
}
```
- Here when you declare a lock_guard and you pass a mutex to it, it will actually acquire the lock if it can, otherwise it is going to wait until it can acquire the lock. So basically it is the same as doing calling lock on a mutex. When it goes out of scope, this is a different thing. It will release the lock. 

```CPP
++count;
```
> So this code can't run until the lock is acquired on the mutex. 

**A situation where one thread has to wait for another thread before it can proceed**
```CPP
#include <iostream>
#include <thread>
#include <chrono>

using namespace std;

int main()
{
	bool ready = false;

	thread t1([&]() {
		this_thread::sleep_for(chrono::milliseconds(2000));
		ready = true;
	});
	t1.join();
}
```
- In this code, i wanted to know when "ready" becomes true, which happens after 2 seconds. But you can imagine that this thread might be doing something more complicated. We don't know when this variable is going to become true. 























