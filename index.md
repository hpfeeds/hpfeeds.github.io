---
layout: default
title: hpfeeds
---

hpfeeds is a lightweight authenticated publish-subscribe protocol that supports arbitrary binary payloads. It has a simple wire-format so that everyone is able to subscribe to feeds with their favorite language, and quickly.

Different feeds are separated by channels and support arbitrary binary payloads. This means that the channel users have to decide about the structure of data. While it is quite common to send JSON payloads to a channel users are in full control of the structure of the data transmitted and the serialization format used.

To authenticate against a broker you use Authkeys - an identifier and secret pair. The secret is never transmitted on the wired, it his hashed together with a nonce to prove that you have a copy of the secret. This way no eavesdroppers can obtain valid credentials.

Optionally, the protocol can be run on top of TLS.


## Quick Start

You can create a broker in minutes with docker-compose:

```yaml
version: '2.1'

volumes:
  hpfeeds_userdb: {}

services:
  hpfeeds:
    image: hpfeeds/hpfeeds-broker
    container_name: hpfeeds
    ports:
     - "0.0.0.0:10000:10000"
     - "127.0.0.1:9431:9431"
    volumes:
     - hpfeeds_data:/app/var
    restart: always
```

You can start this broker with:

```bash
docker-compose up -d
```

For more details on how to manage the broker and setup access control, see the [broker documentation](https://python.hpfeeds.org/en/latest/broker.html).

If you want to quickly test something as a hpfeeds client you can use the command line tools that come with the python implementation. You can install them with:

```bash
pip install hpfeeds
```

You can send a simple event with the `publish` command:

```bash
hpfeeds publish --host localhost -p 10000 -i myident -s mysecret -c mychannel '{"event": "ping"}'
```

You can follow events on a channel with the `subscribe` command:

```bash
hpfeeds subscribe --host localhost -p 10000 -i myident -s mysecret -c mychannel
```

Ctrl+C to stop following the stream.

You can find more information on the python CLI in the [CLI documentation](https://python.hpfeeds.org/en/latest/cli.html).
