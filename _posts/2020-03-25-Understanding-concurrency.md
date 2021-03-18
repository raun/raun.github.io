---
layout: post
title: Understanding Concurrency?
categories: [Web-Service, Threads, Asyncio, Python]
---

## What is concurrency?

Dictionary definition of concurrency is simultaneous occurrence. In Python, the things that are occurring simultaneously are called by different names (thread, task, process). Each one can be stopped at certain points, and the CPU that is processing them can switch to a different one. 

In essence only multiprocessing actually does work simultaneously as it uses multiple processors available on the machine. Threading and AsyncIO both run on a single processor. They just cleverly find ways to take turns to speed up the overall process.

Before we start all the codes used in the post are available here: [Cuncurrency Demo](https://github.com/raun/concurrency-demo)

### Threads
- In threading, the operating system actually knows about each thread and can interrupt it at any time to start running a different thread. This is called Pre-emptive multitasking.
- Pre-emptive multitasking is handy in the sense that code in the thread doesn’t need to do anything to make the switch. It can also be difficult because of that “at any time” phrase.

### Asyncio
- Tasks must cooperate by announcing when they are ready to be switched out. This is called Cooperative multitasking.
- That means that the code in the AsyncIO task has to change slightly to announce that it is ready to be switched out.
- The benefit of doing this extra work up front is that you always know where your task will be swapped out.

## When is concurrency useful?
- CPU-bound tasks
	- There are classes of programs that do significant computation without talking to the network or accessing a file
	- These are the CPU-bound programs, because the resource limiting the speed of your program is the CPU, not the network or the file system
- I/O-bound tasks
	- I/O-bound problems cause your program to slow down because it frequently must wait for input/output (I/O) from some external resource
	- They arise frequently when your program is working with things that are much slower than your CPU

## Lets write some code

## IO Bound Task

#### Synchronous Version


```python
import requests
import time
def download_site(url, session):
    with session.get(url) as response:
        print("Read {} from {}".format(len(response.content), url))

def download_all_sites(sites):
    with requests.Session() as session:
        for url in sites:
            download_site(url, session)

if __name__ == "__main__":
    sites = [
        "https://www.amazon.com",
        "https://www.stackoverflow.com/",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print("Downloaded {} in {} seconds".format(len(sites), duration))
```

Output: `Downloaded 160 in 15.8924131 seconds`

**Why Synchronous version rocks?**
- It is easy to write, easy to debug, single train of thought

**Problems with synchronous version**
- It is dead slow! Although being slower is not necessarily a problem if:
	- It runs rarely
	- The completion time is within SLA

### Threading Version


```python
import concurrent.futures
import requests
import threading
import time
thread_local = threading.local()
def get_session():
    if not hasattr(thread_local, "session"):
        thread_local.session = requests.Session()
    return thread_local.session

def download_site(url):
    session = get_session()
    with session.get(url) as response:
        print("Read {} from {}".format(len(response.content), url))

def download_all_sites(sites):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        executor.map(download_site, sites)

if __name__ == "__main__":
    sites = [
        "https://www.amazon.com",
        "https://www.stackoverflow.com/",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print("Downloaded {} in {} seconds".format(len(sites), duration))
```

Output: `Downloaded 160 in 3.858917 seconds`

**There are few changes as compared to Synchronous version**:
Now `download_all_sites()` changed from calling the function once per site to a more complex structure. In this version, you’re creating a `ThreadPoolExecutor`, which seems like a complicated thing. Let’s break that down: `ThreadPoolExecutor` = `Thread` + `Pool` + `Executor`. This object is going to create a pool of threads, each of which can run concurrently. The Executor is the part that’s going to control how and when each of the threads in the pool will run. `map()` method runs the passed-in function on each of the sites in the list. The great part is that it automatically runs them concurrently using the pool of threads it is managing. The other interesting change in our example is that each thread needs to create its own `requests.Session()` object. Because the operating system is in control of when your task gets interrupted and another task starts, any data that is shared between the threads needs to be protected, or thread-safe. Threading.local() creates an object that look like a global but is specific to each individual thread.

One of the key challenge here is How to decide the number of thread in the pool? This require a little hit and trial(Some estimation & Calculation as well). 

You might expect that having one thread per download would be the fastest but, at least on my system it was not. Why is that? There are two main competing force working here:
- Speed up due to concurrency
- Overhead of creating & destroying thread

On my machine, the optimal value was 10 threads. Another interesting thing is, with 10 threads, the time has reduced slightly more than 1/10th of the original time. Why is that? This is because we added the overhead of creating and destroying the threads and there is part of the problem that can not run in parallel.

**Why the threading version rocks?**

Its fast! Or atleast faster than the synchronous version. But why? Because it allow your program to overlap the waiting times and get the final result faster.

**Problems with the threading version?**

We had to write more code to make it happen and think about how to keep objects and code thread safe. Threads can interact in ways that are subtle and hard to detect. These interaction can cause race conditions which can result random, intermittent bugs that are difficult to debug.

### Asyncio version


```python
import asyncio
import time
import aiohttp
async def download_site(session, url):
    async with session.get(url) as response:
        print("Read {0} from {1}".format(response.content_length, url))

async def download_all_sites(sites):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in sites:
            task = asyncio.ensure_future(download_site(session, url))
            tasks.append(task)
        await asyncio.gather(*tasks, return_exceptions=True)

if __name__ == "__main__":
    sites = [
        "https://www.amazon.com",
        "https://www.stackoverflow.com/",
    ] * 80
    start_time = time.time()
    asyncio.get_event_loop().run_until_complete(download_all_sites(sites))
    duration = time.time() - start_time
    print("Downloaded {} sites in {} seconds".format(len(sites), duration))
```

Output: `Downloaded 160 sites in 3.422111 seconds`

**There are few changes as compared to Synchronous version**:
A single Python object, called the event loop, controls how and when each task gets run. The event loop is aware of each task and knows what state it’s in. Let's assume a simplified event loop which has 2 states(i.e. Ready to work & Waiting for external thing). Your simplified event loop maintains two lists of tasks, one for each of these states. 

Your simplified event loop selects one of the ready tasks and starts it back to running. That task is in complete control until it cooperatively hands the control back to the event loop.
When the running task gives control back to the event loop, the event loop places that task into either the ready or waiting list and then goes through each of the tasks in the waiting list to see if it has become ready by completing an I/O operation.

An important point of asyncio is that the tasks never give up control without intentionally doing so. They never get interrupted in the middle of an operation. This allows us to share resources a bit more easily in asyncio than in threading. 


**Why the asyncio version rocks?**

It is faster than the threading version. It forces you to think about when a given task will get swapped out, which can help you create a better, faster, design. It is much more scalable. With threading version you have to fine tune the number of thread based on problem and hardware you are working with.

**Problems with asyncio version**

You need special async versions of libraries to gain the full advantage of asyncio. Had you just used requests for downloading the sites, it would have been much slower because requests is not designed to notify the event loop that it’s blocked.
All of the advantages of cooperative multitasking get thrown away if one of the tasks doesn’t cooperate. A minor mistake in code can cause a task to run off and hold the processor for a long time, starving other tasks that need running. There is no way for the event loop to break in if a task does not hand control back to it.

### multiprocessing version


```python
import requests
import multiprocessing
import time
session = None
def set_global_session():
    global session
    if not session:
        session = requests.Session()

def download_site(url):
    with session.get(url) as response:
        name = multiprocessing.current_process().name
        print("{}:Read {} from {}".format(name, len(response.content), url))

def download_all_sites(sites):
    with multiprocessing.Pool(initializer=set_global_session) as pool:
        pool.map(download_site, sites)

if __name__ == "__main__":
    sites = [
        "https://www.amazon.com",
        "https://www.stackoverflow.com/",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print("Downloaded {} in {} seconds".format(len(sites), duration))
```

Output: `Downloaded 160 in 4.1463499 seconds`

**There are few changes as compared to Synchronous version**:
Code looks is much shorter than asyncio version. multiprocessing on high level create a new instance of Python interpreter to run on each CPU and then farming out part of our program to run on it.

**Why the multiprogramming version rocks?**

Relatively easy to setup & requires little extra code. Takes full advantage of the CPU power on your computer.

**Problems with multiprogramming version**

Slower than asyncio and threading. Require some setup from synchronous version. We have spend some time thinking about which variables will be accessed in each process.


## CPU Bound task

### Synchronous Version


```python
import time
def cpu_bound(number):
    return sum(i * i for i in range(number))

def find_sums(numbers):
    for number in numbers:
        cpu_bound(number)

if __name__ == "__main__":
    numbers = [5000000 + x for x in range(20)]
    start_time = time.time()
    find_sums(numbers)
    duration = time.time() - start_time
    print("Duration {} seconds".format(duration))
```

Output: `Duration 9.244601 seconds`

This code calls cpu_bound() 20 times with a different large number each time. It does all of this on a single thread in a single process on a single CPU. Unlike the I/O-bound examples, the CPU-bound examples are usually fairly consistent in their run times. Why is that? It is because networks are unreliable and can respond slow or fast with time.

Clearly we can do better than this. This is all running on a single CPU with no concurrency.

How much do you think rewriting this code using threading or asyncio will speed this up? Lets see!

### threading & asyncio versions


```python
import concurrent.futures
import time
def cpu_bound(number):
    return sum(i * i for i in range(number))

def find_sums(numbers):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        executor.map(cpu_bound, numbers)

if __name__ == "__main__":
    numbers = [5000000 + x for x in range(20)]
    start_time = time.time()
    find_sums(numbers)
    duration = time.time() - start_time
    print("Duration {} seconds".format(duration))
```

Output: `Duration 9.190521 seconds`

In your I/O-bound example, much of the overall time was spent waiting for slow operations to finish. threading and asyncio sped this up by allowing you to overlap the times you were waiting instead of doing them sequentially. On a CPU-bound problem, however, there is no waiting. The CPU is cranking away as fast as it can to finish the problem. One CPU is doing all of the work of the non-concurrent code plus the extra work of setting up threads or tasks.

Lets see how multiprocessing perform on this problem!

### multiprocessing Version


```python
import multiprocessing
import time
def cpu_bound(number):
    return sum(i * i for i in range(number))

def find_sums(numbers):
    with multiprocessing.Pool() as pool:
        pool.map(cpu_bound, numbers)

if __name__ == "__main__":
    numbers = [5000000 + x for x in range(20)]
    start_time = time.time()
    find_sums(numbers)
    duration = time.time() - start_time
    print("Duration {} seconds".format(duration))
```

Output: `Duration 1.586885 seconds`

**Why the multiprocessing version rocks?**
Easy to setup with very little code. Take full advantage of CPU power on your computer

**Problems with multiprocessing version**
Splitting your problem up so each processor can work independently can sometimes be difficult. Many solutions require more communication between the processes. This would add more complexity to your synchronous version and sometime these communication overhead can outweigh the benefit.


## Conclusion
We have covered a lot of ground here. It’s time to review those ideas and discuss about decision points.

The first step of this process is deciding if you should use a concurrency module. While the examples here make each of the libraries look pretty simple, concurrency always comes with extra complexity and can often result in bugs that are difficult to find. Hold out on adding concurrency until you have a known performance issue and then determine which type of concurrency you need. Once you’ve decided that you should optimize your program, figuring out if your program is CPU-bound or I/O-bound.

CPU-bound problems only really gain from using `multiprocessing`. `threading` and `asyncio` did not help this type of problem at all.

For I/O-bound problems, there’s a general rule of thumb in the Python community: “Use `asyncio` when you can, `threading` when you must.” `asyncio` can provide the best speed up for this type of program, but sometimes you will require critical libraries that have not been ported to take advantage of `asyncio`.

