# Things to watch out for #

### Callbacks and blocking actions ###

Remember that all NIO callbacks are done on the NIO thread, which means **no** I/O can happen while the callback is running. This means you should never make lengthy or blocking calls as a response.


In particular watch out when implementing the following methods:

`SocketObserver#connectionOpened(NIOSocket)`

`SocketObserver#packetReceived(NIOSocket)`

`ServerSocketObserver#newConnection(NIOSocket)`

`ConnectionAcceptor#acceptConnection(InetSocketAddress)`

### Issues on Android ###

Opening a socket on some versions of Android 2.2 will cause an exception due to a bug in that version (of Android).

A workaround is to add the following:

```
java.lang.System.setProperty("java.net.preferIPv4Stack", "true");
java.lang.System.setProperty("java.net.preferIPv6Addresses", "false");
```