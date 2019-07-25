---
layout: post
title:  "RabbitMQ configure web ui and write a simple producer"
date:   2018-09-07 16:09:54 +0200
categories: queues notes
---

* RabbitMQ 3.6.10
* Erlang 5.10.4

#### Centos Installation

```
# yum update -y
# yum install wget erlang socat -y
# wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.10/rabbitmq-server-3.6.10-1.el7.noarch.rpm
```

#### Enable Web UI:

```
rabbitmq-plugins enable rabbitmq_management

rabbitmq-plugins list
Configured: E = explicitly enabled; e = implicitly enabled
| Status: * = running on rabbit@rabbit1
|/
[e*] amqp_client 3.6.10
[e*] cowboy 1.0.4
[e*] cowlib 1.0.2
[ ] rabbitmq_amqp1_0 3.6.10
[ ] rabbitmq_auth_backend_ldap 3.6.10
[ ] rabbitmq_auth_mechanism_ssl 3.6.10
[ ] rabbitmq_consistent_hash_exchange 3.6.10
[ ] rabbitmq_event_exchange 3.6.10
[ ] rabbitmq_federation 3.6.10
[ ] rabbitmq_federation_management 3.6.10
[ ] rabbitmq_jms_topic_exchange 3.6.10
[E*] rabbitmq_management 3.6.10
[e*] rabbitmq_management_agent 3.6.10
[ ] rabbitmq_management_visualiser 3.6.10
[ ] rabbitmq_mqtt 3.6.10
[ ] rabbitmq_recent_history_exchange 3.6.10
[ ] rabbitmq_sharding 3.6.10
[ ] rabbitmq_shovel 3.6.10
[ ] rabbitmq_shovel_management 3.6.10
[ ] rabbitmq_stomp 3.6.10
[ ] rabbitmq_top 3.6.10
[ ] rabbitmq_tracing 3.6.10
[ ] rabbitmq_trust_store 3.6.10
[e*] rabbitmq_web_dispatch 3.6.10
[ ] rabbitmq_web_mqtt 3.6.10
[ ] rabbitmq_web_mqtt_examples 3.6.10
[ ] rabbitmq_web_stomp 3.6.10
[ ] rabbitmq_web_stomp_examples 3.6.10
[ ] sockjs 0.3.4
```

#### Configure firewalld

```
firewall-cmd --add-port 15672/tcp --permanent
```
#### Create a user allowed to access the web UI

```
rabbitmqctl add_user max max
rabbitmqctl set_user_tags max administrator
rabbitmqctl set_permissions -p / max ".*" ".*" ".*"
```

#### A simple pthon producer with pika

```python
#!/usr/bin/env python

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='IoTData-queue')

channel.basic_publish(exchange='',
routing_key='IoTData-queue',
body='Un altro messaggio')
```

#### Show queues

```
rabbitmqctl list_queues
```
