# Release Schedule for Naga #


## Releases ##

## 1.0 - Done ##

First release to get something shipping.

## 1.1 - Done ##

Full javadoc, packet readers extended to the planned the default set. Bug fixes.

## 1.2 - Done ##

Single bug fix. Issue when using selectBlocking() and write() on different threads.

## 1.3 - Done ##

Default acceptor to allow all instead of deny all when creating a server socket. Expanded javadocs. Exposed wakeup() to interrupt selectBlocking().

## 2.0 - Done ##

Simple EventMachine to work with Naga. Some improvement to javadoc.

## 2.1 - Done ##

Bug fixes.

## 3.0 - In Progress ##

Features:
  * SSL support (Beta)
  * Callback on packet written.
  * Simplified PacketReader / PacketWriter
  * Bug fixes

_3.0 Will not be 100% compatible with 1.x-2.x_

## 3.x - TBA ##

Completed SSL support.