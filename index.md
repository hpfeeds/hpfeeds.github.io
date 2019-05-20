---
layout: default
title: hpfeeds
---

hpfeeds is a lightweight authenticated publish-subscribe protocol that supports arbitrary binary payloads.

We tried to design a simple wire-format so that everyone is able to subscribe to the feeds with his favorite language in almost no time.

Different feeds are separated by channels and support arbitrary binary payloads. This means that the channel users have to decide about the structure of data. This could for example be done by choosing a serialization format.

Access to channels is given to so-called Authkeys which essentially are pairs of an identifier and a secret. The secret is sent to the server by hashing it together with a per-connection nonce. This way no eavesdroppers can obtain valid credentials. Optionally the protocol can be run on top of SSL/TLS, of course.

To support multiple data sources and sinks per user we manage the Authkeys in this webinterface after a quick login with a user account. User accounts are only needed for the webinterface - to use the data feed channels, only Authkeys are necessary. Different Authkeys can be granted distinct access rights for channels.

## Language Implementations

If you are building a sensor or data collector or tool that uses hpfeeds you may want to use an existing client. There are implementations in:

 * C: [GitHub][https://github.com/hpfeeds/libhpfeeds]
 * C++: [GitHub][https://github.com/tentacool/tentacool
 * Go: [GitHub][https://github.com/hpfeeds/go-hpfeeds]
 * JavaScript: [GitHub][https://github.com/fw42/honeymap/blob/master/server/node_modules/hpfeeds/index.js]
 * Perl: [GitHub][https://github.com/hpfeeds/perl-POE-Component-hpfeeds]
 * Python [GitHub](https://github.com/hpfeeds/hpfeeds) [Docs](https://python.hpfeeds.org/)
 * Ruby: [GitHub][https://github.com/fw42/hpfeedsrb]
 * More Ruby: [GitHub][https://github.com/vicvega/hpfeeds-ruby]


## Wire Protocol

hpfeeds is a simple TCP messge-based binary protocol.

There are 6 message types and each starts with a standard header containing its length and type:

```c
struct MsgHeader {
  uint32_t messageLength;
  uint8_t opCode;
};
```

The following op codes are available in the protocol:

| Operation        | Code |
|------------------|------|
| `OP_ERROR`       | 0    |
| `OP_INFO`        | 1    |
| `OP_AUTH`        | 2    |
| `OP_PUBLISH`     | 3    |
| `OP_SUBSCRIBE`   | 4    |
| `OP_UNSUBSCRIBE` | 5    |

Fields with a dynamic length (e.g. ``ident`` or ``channel``) will be prefixed by their length as a ``uint8_t`` (i.e. 1 byte).

The final field in a message need not have a length as the message parser can easily calculate 'every that is left'.

When you connect to a broker it will reply with an `OP_INFO` message with its name and a nonce. A client must respond with an `OP_AUTH` message. The broker may reply with an `OP_ERROR` and terminate the connection if you are not authenticated. Otherwise the client may now:

 * send an `OP_SUBSCRIBE` to watch a channel. The broker will send an `OP_PUBLISH` to the client for each message it receives on that channel.
 * send an `OP_PUBLISH` to send a message to a channel.


### Error

``OP_ERROR`` is sent from the server to the client. It usually signifies a permission error. The error may not be fatal and a client can stay connected. If the error is fatal the broker will drop the connection.

A client must not sent an ``OP_ERROR``.

Here is an example error message:

|Field         |Value              |
|--------------|-------------------|
|Length        |``18``             |
|Op Code       |``0``              |
|Error Message |``Invalid ident``  |


The serialized message will look like:

```
00000000:  00 00 00 12 00 49 6e 76  61 6c 69 64 20 69 64 65   |.....Invalid ide|
00000010:  6e 74                                              |nt|
```


### Info

``OP_INFO`` is sent from the server to the client exactly once - when the connection is first established. It identifies the server name and gives the
client some random bytes (a nonce) to sign with its secret.

The client must not send an ``OP_INFO``.

A typical ``OP_INFO`` might look like this:

|Field         |Value               |
|--------------|--------------------|
|Length        |``17``              | 
|Op Code       |``1``               |
|Next          |``7``               |
|Broker name   |``hpfeeds``         |
|Nonce         |``\x00\x00\x00\x00``|

The serialized message will look like:

```
00000000:  00 00 00 11 01 07 68 70  66 65 65 64 73 00 00 00   |......hpfeeds...|
00000010:  00                                                 |.|
```


### Auth

``OP_AUTH`` is sent from the client to the server in response to an ``OP_INFO``.
The client generates a ``sha1(nonce + authkey)`` hash and sends its identity and
this hash back to the server.

The server must not send an ``OP_AUTH``.

A typical ``OP_INFO`` might look like this:

|Field         |Value                                                          |
|--------------|---------------------------------------------------------------|
|Length        |``33``|
|Op Code       |``2``|
|Next          |``7``|
|Client ident  |``client1``|
|Signed Nonce  |``\xaf\xae\xae_\xc7a\x19\x1b\xe3\xf9\xce\xce_\xfbp\xbcPiB\xa4``|

The serialized message will look like:

```
00000000:  00 00 00 21 02 07 63 6c  69 65 6e 74 31 af ae ae   |...!..client1...|
00000010:  5f c7 61 19 1b e3 f9 ce  ce 5f fb 70 bc 50 69 42   |_.a......_.p.PiB|
00000020:  a4                                                 |.|
```


### Publish

``OP_PUBLISH`` is sent from the client to the server to transmit an opaque binary payload. The message comprises of the client ident, destination channel
and payload.

``OP_PUBLISH`` is sent from the server to clients to distribute an incoming ``OP_PUBLISH`` to all interested subscribers.

A typical ``OP_INFO`` might look like this

|Field         |Value|
|--------------|--------------------------------------------------------------------|
|Length        |``85``|
|Op Code       |``3``|
|Next          |``9``|
|Client ident  |``b4aa2@hp1``|
|Next          |``9``|
|Channel       |``mwcapture``|
|Payload       |``137941a3d8589f6728924c08561070bceb5d72b8,http://1.2.3.4/calc.exe``|

The serialized message will look like:

````
00000000:  00 00 00 59 03 09 62 34  61 61 32 40 68 70 31 09   |...Y..b4aa2@hp1.|
00000010:  6d 77 63 61 70 74 75 72  65 31 33 37 39 34 31 61   |mwcapture137941a|
00000020:  33 64 38 35 38 39 66 36  37 32 38 39 32 34 63 30   |3d8589f6728924c0|
00000030:  38 35 36 31 30 37 30 62  63 65 62 35 64 37 32 62   |8561070bceb5d72b|
00000040:  38 2c 68 74 74 70 3a 2f  2f 31 2e 32 2e 33 2e 34   |8,http://1.2.3.4|
00000050:  2f 63 61 6c 63 2e 65 78  65                        |/calc.exe|
````


### Subscribe

``OP_SUBSCRIBE`` is sent from the client to the server to register interest in all future messages on a given channel. If the client doesn't have permission to access this channel an ``OP_ERROR`` will be sent.

The broker must not send an ``OP_SUBSCRIBE``.

A typical subscription request is:

|Field         |Value|
|--------------|--------------|
|Length        |``22``|
|Op Code       |``4``|
|Next          |``7``|
|Client ident  |``client1``|
|Channel       |``mwcapture``|

The serialized message will look like:

```
00000000:  00 00 00 16 04 07 63 6c  69 65 6e 74 31 6d 77 63   |......client1mwc|
00000010:  61 70 74 75 72 65                                  |apture|
```


### Unsubscribe

``OP_UNSUBSCRIBE`` is sent from the client to the server to register interest in all future messages on a given channel. No permission check is needed - if you subscribed to a channel you are allowed to unsubscribe.

The broker must not send an ``OP_UNSUBSCRIBE``.

A typical unsubscribe request is:

|Field         |Value|
|--------------|--------------|
|Length        |``22``|
|Op Code       |``5``|
|Next          |``7``|
|Client ident  |``client1``|
|Channel       |``mwcapture``|

The serialized message will look like:

```
00000000:  00 00 00 16 05 07 63 6c  69 65 6e 74 31 6d 77 63   |......client1mwc|
00000010:  61 70 74 75 72 65                                  |apture|
```
