---
layout: default
title: Brokers
permalink: /brokers
---

# hpfeeds-broker

Originally written for python2 and gevent, the original hpfeeds broker has been ported to python3 and asyncio. It's gained prometheus integration for monitoring. If you only have a small-medium number of user accounts you can use a Kubernetes `Secret` resource to manage access control.

[GitHub](https://github.com/hpfeeds/hpfeeds) [Documentation](https://python.hpfeeds.org/en/latest/broker.html)


# HPFBroker

HPFBroker is a hpfeeds broker written in go by [@d1str0](https://github.com/d1str0). It has a webserver GUI for managing channels, users and passwords. It has an API for use as a microservice.

[GitHub](https://github.com/d1str0/HPFBroker)


# Tentacool

Tentacool is a C++ implementation of a hpfeeds broker capable of 65k messages / second (as measured in 2014). As well as simple flat file authentication configuration it can use mongo as its auth store.

[GitHub](https://github.com/tentacool/tentacool)
