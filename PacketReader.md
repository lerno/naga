A packet reader is a class that converts the incoming byte stream into separate packets.

Since bytes sent from a client might get chopped up during transfer, a reader typically needs to buffer bytes until a complete packet is received.

# Introduction #

Naga provides four different types of readers:
  * `RawPacketReader`
  * `RegularPacketReader`
  * `DelimitedPacketReader`
  * `AsciiLinePacketReader`
  * `ZeroDelimiterPacketReader`

It is possible to roll your own packet reader, but to do so takes a little bit of knowledge of ByteBuffers and how Naga internally uses the packet reader.



# `RegularPacketReader` #

This reader assumes packets come with a header followed by the actual data.

You can define a regular reader to use 1, 2, 3 or 4 bytes for the header, which in turn limits the number of bytes a single packet can contain.

| **Header Size** | **Max Content Size (bytes)** |
|:----------------|:-----------------------------|
| 1 | 255 |
| 2 | 65,535 |
| 3 | 16,777,215 |
| 4 | 2,147,483,646 |

The bytes of the header contains the size of the content that follows, which allows the reader to dynamically resize its buffer depending on the header.

The header itself can either be big or small endian, i.e highest byte first or lowest byte first.

### Example ###

A big endian reader with header size 3 encounters the bytes [0, 0, 3, 1, 2, 3, 0, 0, 1] will parse [0, 0, 3] to be the header with a content of 3 bytes. It will therefore read three more bytes, and dispatch them as a packet [1, 2, 3].

It will then continue reading [0, 0, 1] as a packet with a content of 1 byte, and at this time it will pause as the content is not yet available in the stream.

### Typical Use ###

Anywhere where you want to use binary data. Just make sure that the header and endian-ness is identical on both sides.



# `RawPacketReader` #

This reader is attached to a `NIOSocket` by default. It reads as much as it can in a single pass, limited ony
by its internal buffer size and the amount of bytes available.

It will then dispatch a read from a single pass as a packet.

### Example ###

Let us suppose we have a socket with a `RawPacketReader` containing with buffer size 3.

Encountering the bytes 0, 1, 2, 3, 4, 5 it will read [0, 1, 2] into a single packet and dispatch it, then read [2, 3, 4] and dispatch it, then finally read [5](5.md) and dispatch it as a packet.

### Typical Use ###

If you want to make your own custom parsing and concatenation of data further down, this type of reader could be the way to go.



# `DelimitedPacketReader` #

This reader will separate data given some byte delimiter. It can either use a maximum packet size (recommended in server applications) or an unbounded packet size.

### Example ###

If we create a packet reader with the delimiter 1, we get the following behaviour.

Encountering the bytes [0, 0, 3, 1, 1, 2, 3, 0, 0, 1, 8] it will start reading until it reaches the first 1, and pack the bytes found up until then into a packet [0, 0, 3] (not that the delimiter does not occur in the packet). It will then proceed to the next delimiter and send a zero length packet. The next packet is [2, 3, 0, 0] and the final 8 will remain in the internal buffer.


### Typical Use ###

Text based protocols see `AsciiLineDelimitedPacketReader` and other simple protocols.



# `AsciiLinePacketReader` and `ZeroDelimiterPacketReader` #

These readers are subclasses of `DelimitedPacketReader` with delimiters \n (10) and 0 respectively.

# Creating your own reader #

### `getNextPacket()` ###

This method will **always** be called whenever reading to the byte buffer yields 1 or more bytes. For this reason this method can safely be used to prepare the byte buffer for the next read.

You are guaranteed that this method will be called repeatedly until a null is returned, so in general this method can consume data from the internal buffer one packet a time.

Since only returning null will allow a new call to getBuffer(), it is ok to leave the byte buffer in an invalid state if the last return was a non-null.


### `getBuffer()` ###

This method should return the next buffer you want to read data into. In general you should _never_ return a ByteBuffer that does not have any space left to add data since this is interpreted as there being no data left in the stream.