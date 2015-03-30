## A short introduction to EventMachine ##

(For this example I was originally going to include an walk-through on how to write a chat server with login and idle timeouts using EventMachine,
but the code had a bit too little EventMachine functionality to be a good example, but if anyone is interested in the source code I can always
add it to the naga.examples package)

EventMachine is a very thin wrapper around NIOServices that carries an internal thread. Both the NIOService and events will be executed on this single thread:

```
EventMachine eventMachine = new EventMachine();
eventMachine.start();
System.out.println(Thread.currentThread() + ": Scheduling an event to be executed on the event machine thread.");
eventMachine.asyncExecute(new Runnable()
{
    public void run()
    {
        System.out.println(Thread.currentThread() + ": This is called on the event machine thread!");
    }
});
```

Here is output from a test run:
```
Thread[main,5,main]: Scheduling an event to be executed on the event machine thread.
Thread[Thread-0,5,main]: This is called on the event machine thread!
```

### Scheduled Events ###

We can schedule events to occur later:
```
final long time = System.currentTimeMillis();
eventMachine.executeAt(new Runnable()
{
    public void run()
    {
        System.out.println("'ExecuteAt' Called after " + (System.currentTimeMillis() - time) + "ms.");
    }
}, new Date(time + 5000));
eventMachine.executeLater(new Runnable()
{
    public void run()
    {
        System.out.println("'ExecuteLater' Called after " + (System.currentTimeMillis() - time) + "ms.");
    }
}, 10000);
```

This examples shows how an event can be scheduled to either run at a specific date or after a delay.

A test run on my machine gives the output:

```
'ExecuteAt' Called after 5000ms.
'ExecuteLater' Called after 10002ms.
```

### Handling Slow/Blocking Code ###

Since EventMachine runs all IO and event handling in a single thread it is not recommended to do any blocking or lengthy operations, just as with the NIOSocket callbacks.

If a blocking or slow operation needs to be done, it should be deferrred to a separate thread or threadpool (look at the Executor classes for an easily available thread pool solution) and then posted back to the EventMachine using asyncExecute.

For instance, imagine the following snippet of code:
```
boolean ok = validateUserPasswordFromDb(user, password);
if (ok)
{
    loginUser(user);
}
else
{
    refuseLogin();
}
```

In this case we might want to run the db validation in another thread, like this:

```
// On the EventMachine thread we spawn a separate thread to perform our query.
Thread t = new Thread()
{
    public void run()
    {
        // We execute the lengthy query on the thread.
        final boolean ok = validateUserPasswordFromDb(user, password);
        // Now that we have answer, post an event to finish the validation.
        eventMachine.asyncExecute(new Runnable()
        {
            public void run()
            {
                // This code is executed on the EventMachine thread
                if (ok)
                {
                    loginUser(user);
                }
                else
                {
                    refuseLogin();
                }
            }
        });
    }
}.start();
```