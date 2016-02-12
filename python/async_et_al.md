# Async, Await, Generators, Coroutines, and asyncio.

Here's some clarity on the distinctions between several python constructs:

 * An **iterator** is an interface for producing a sequence of values
 * A **generator** implements the iterator interface by lazily "yield"ing values from a function. This has the effect of repeatedly suspending function execution mid-body, and returning intermediate results.
 * **Coroutines** are like generators, but the flow of data is bidirectional; coroutines send data out when they yield, and callers send data back in when they resume the coroutine.
* **asyncio** is a module in the standard library that uses coroutines as the building blocks for asynchronous code. **[Cooperative multitasking](https://en.wikipedia.org/wiki/Cooperative_multitasking)** is the concurrency paradigm behind asyncio -- the event loop does not preemtively steal execution from any coroutine, but instead waits for the coroutine to yield before switching to another task. Note that this means that asyncio programs are not responsive for free -- they are only responsive if tasks yield frequently enough, and don't do substantial work between yield methods.

Asyncio coroutines yield data structures that the event loop introspects to control context switching. For example in this example

```

@asyncio.coroutine
def countdown(number, n):
    while n > 0:
        print('T-minus', n, '({})'.format(number))
        yield from asyncio.sleep(1)
        n -= 1
```

`asyncio.sleep(1)` doesn't actually perform the sleep (that would prevent other coroutines form running). It returns something the event loop uses to schedule calling send() 1 second in the future


### `yield from` with coroutines

For a generator `yield from sequence` is just syntactic sugar for

```
for item in sequence:
    yield item
```

However in the context of coroutines they are different,
since they also push send() data through each intermediary. For example:

```
def shout():
    value = None
    while True:
        value = yield value
        value = str(value).upper()

def pipe():
    yield from shout()
    
def not_pipe():
    for item in shout():
        yield item
    
    
val = pipe()
val.send(None)  # to start execution
val.send('hi')  # HI

val = not_pipe()
val.send(None)
val.send('hi')  # NONE
```
        
Also note that "x = yield from y" in an intermediary coroutine like `pipe` would exchange a stream of data between
`y` and the coroutine's caller, and then return the result of `y` to `x`:

```
def tally():
    sum = 0
    for _ in range(5):
        x = yield "got it"
        sum += x
    return sum
        
def pipe():
    x = yield from tally()
    print("Sum is %s" % x)
    
val = pipe()
val.send(None) # 'got it'
val.send(2) # 'got it'
val.send(2) # 'got it'
val.send(2) # 'got it'
val.send(2) # 'got it'
val.send(2) # 'Sum is 10'
```

In asyncio coroutines communicate with the event loop by `yield from`ing Futures. This organization lets the event loop interact with the future to resolve it's value and then, when ready, pass result data into the coroutine that yielded the future.


### What does async/await provide

`async def` as added in Python 3.5 to make coroutines
a distinct type in the language, as opposed to an interface similar to generators (at the language level, there's no way to distinguish a coroutine from a generator in <=3.4). This provides some type checking utilities, as well as support for some extra language constructs (async for, async with, etc). In a 3.5 coroutine `await` replaces `yield from`. 

### How asyncio improves performance

It's clear that `yield`ing often enough within coroutines can
provide responsiveness. In order to improve *performance* for IO bound, single-threaded code, such code needs to never block on IO operations. The socket reader code in asyncio use the `socket` library for low-level data exchange, which has a nonblocking API. So the asyncio coroutines poll the socket object for the presence of data and, while it's not yet available, do the equivalent of `yield sleep()`. Roughly it looks like this:

```
async def get_data():
    if sock.has_data():
        return sock.get_data()
    yield from asyncio.sleep()
```

I suspect that `socket` might internally communicate with the network in a separate GIL-released thread to facilitate this.

### References

* http://www.snarky.ca/how-the-heck-does-async-await-work-in-python-3-5
