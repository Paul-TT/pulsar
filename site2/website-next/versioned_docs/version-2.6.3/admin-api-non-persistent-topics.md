---
id: admin-api-non-persistent-topics
title: Managing non-persistent topics
sidebar_label: "Non-Persistent topics"
original_id: admin-api-non-persistent-topics
---

Non-persistent can be used in applications that only want to consume real time published messages and
do not need persistent guarantee that can also reduce message-publish latency by removing overhead of
persisting messages.

In all of the instructions and commands below, the topic name structure is:

```shell

non-persistent://tenant/namespace/topic

```

## Non-persistent topics resources

### Get stats

It shows current statistics of a given non-partitioned topic.

  -   **msgRateIn**: The sum of all local and replication publishers' publish rates in messages per second

  -   **msgThroughputIn**: Same as above, but in bytes per second instead of messages per second

  -   **msgRateOut**: The sum of all local and replication consumers' dispatch rates in messages per second

  -   **msgThroughputOut**: Same as above, but in bytes per second instead of messages per second

  -   **averageMsgSize**: The average size in bytes of messages published within the last interval

  -   **publishers**: The list of all local publishers into the topic. There can be zero or thousands

  -   **averageMsgSize**: Average message size in bytes from this publisher within the last interval

  -   **producerId**: Internal identifier for this producer on this topic

  -   **producerName**: Internal identifier for this producer, generated by the client library

  -   **address**: IP address and source port for the connection of this producer

  -   **connectedSince**: Timestamp this producer was created or last reconnected

  -   **subscriptions**: The list of all local subscriptions to the topic

  -   **my-subscription**: The name of this subscription (client defined)

  -   **type**: This subscription type

  -   **consumers**: The list of connected consumers for this subscription

  -   **consumerName**: Internal identifier for this consumer, generated by the client library

  -   **availablePermits**: The number of messages this consumer has space for in the client library's listen queue. A value less than 1 means the client library's queue is full and receive() isn't being called. A non-negative value means this consumer is ready to be dispatched messages.

  -   **replication**: This section gives the stats for cross-colo replication of this topic

  -   **connected**: Whether the outbound replicator is connected

  -   **inboundConnection**: The IP and port of the broker in the remote cluster's publisher connection to this broker

  -   **inboundConnectedSince**: The TCP connection being used to publish messages to the remote cluster. If there are no local publishers connected, this connection is automatically closed after a minute.

  -   **msgDropRate**: for publisher: publish: broker only allows configured number of in flight per connection, and drops all other published messages above the threshold. Broker also drops messages for subscriptions in case of unavailable limit and connection is not writable.

```json

{
  "msgRateIn": 4641.528542257553,
  "msgThroughputIn": 44663039.74947473,
  "msgRateOut": 0,
  "msgThroughputOut": 0,
  "averageMsgSize": 1232439.816728665,
  "storageSize": 135532389160,
  "msgDropRate" : 0.0,
  "publishers": [
    {
      "msgRateIn": 57.855383881403576,
      "msgThroughputIn": 558994.7078932219,
      "averageMsgSize": 613135,
      "producerId": 0,
      "producerName": null,
      "address": null,
      "connectedSince": null,
      "msgDropRate" : 0.0
    }
  ],
  "subscriptions": {
    "my-topic_subscription": {
      "msgRateOut": 0,
      "msgThroughputOut": 0,
      "msgBacklog": 116632,
      "type": null,
      "msgRateExpired": 36.98245516804671,
       "consumers" : [ {
        "msgRateOut" : 20343.506296021893,
        "msgThroughputOut" : 2.0979855364233278E7,
        "msgRateRedeliver" : 0.0,
        "consumerName" : "fe3c0",
        "availablePermits" : 950,
        "unackedMessages" : 0,
        "blockedConsumerOnUnackedMsgs" : false,
        "address" : "/10.73.210.249:60578",
        "connectedSince" : "2017-07-26 15:13:48.026-0700",
        "clientVersion" : "1.19-incubating-SNAPSHOT"
      } ],
      "msgDropRate" : 432.2390921571593

    }
  },
  "replication": {}
}

```

#### pulsar-admin

Topic stats can be fetched using [`stats`](reference-pulsar-admin.md#stats) command.

```shell

$ pulsar-admin non-persistent stats \
  non-persistent://test-tenant/ns1/tp1 \

```

#### REST API

{@inject: endpoint|GET|/admin/v2/non-persistent/:tenant/:namespace/:topic/stats|operation/getStats?version=2.6.3}


#### Java

```java

String topic = "non-persistent://my-tenant/my-namespace/my-topic";
admin.nonPersistentTopics().getStats(topic);

```

### Get internal stats

It shows detailed statistics of a topic.

#### pulsar-admin

Topic internal-stats can be fetched using [`stats-internal`](reference-pulsar-admin.md#stats-internal) command.

```shell

$ pulsar-admin non-persistent stats-internal \
  non-persistent://test-tenant/ns1/tp1 \

{
  "entriesAddedCounter" : 48834,
  "numberOfEntries" : 0,
  "totalSize" : 0,
  "cursors" : {
    "s1" : {
      "waitingReadOp" : false,
      "pendingReadOps" : 0,
      "messagesConsumedCounter" : 0,
      "cursorLedger" : 0,
      "cursorLedgerLastEntry" : 0
    }
  }
}

```

#### REST API

{@inject: endpoint|GET|/admin/v2/non-persistent/:tenant/:namespace/:topic/internalStats|operation/getInternalStats?version=2.6.3}

#### Java

```java

String topic = "non-persistent://my-tenant/my-namespace/my-topic";
admin.nonPersistentTopics().getInternalStats(topic);

```

### Create partitioned topic

Partitioned topics in Pulsar must be explicitly created. When creating a new partitioned topic you need to provide a name for the topic as well as the desired number of partitions.

:::note

By default, after 60 seconds of creation, topics are considered inactive and deleted automatically to prevent from generating trash data.
To disable this feature, set `brokerDeleteInactiveTopicsEnabled` to `false`.
To change the frequency of checking inactive topics, set `brokerDeleteInactiveTopicsFrequencySeconds` to your desired value.
For more information about these two parameters, see [here](reference-configuration.md#broker).

:::

#### pulsar-admin

```shell

$ bin/pulsar-admin non-persistent create-partitioned-topic \
  non-persistent://my-tenant/my-namespace/my-topic \
  --partitions 4

```

#### REST API

{@inject: endpoint|PUT|/admin/v2/non-persistent/:tenant/:namespace/:topic/partitions|operation/createPartitionedTopic?version=2.6.3}

#### Java

```java

String topicName = "non-persistent://my-tenant/my-namespace/my-topic";
int numPartitions = 4;
admin.nonPersistentTopics().createPartitionedTopic(topicName, numPartitions);

```

### Get metadata

Partitioned topics have metadata associated with them that you can fetch as a JSON object. The following metadata fields are currently available:

Field | Meaning
:-----|:-------
`partitions` | The number of partitions into which the topic is divided

#### pulsar-admin

```shell

$ pulsar-admin non-persistent get-partitioned-topic-metadata \
  non-persistent://my-tenant/my-namespace/my-topic
{
  "partitions": 4
}

```

#### REST API

{@inject: endpoint|GET|/admin/v2/non-persistent/:tenant/:namespace/:topic/partitions|operation/getPartitionedMetadata?version=2.6.3}


#### Java

```java

String topicName = "non-persistent://my-tenant/my-namespace/my-topic";
admin.nonPersistentTopics().getPartitionedTopicMetadata(topicName);

```

### Unload topic

It unloads a topic.

#### pulsar-admin

Topic can be unloaded using [`unload`](reference-pulsar-admin.md#unload) command.

```shell

$ pulsar-admin non-persistent unload \
  non-persistent://test-tenant/ns1/tp1 \

```

#### REST API

{@inject: endpoint|PUT|/admin/v2/non-persistent/:tenant/:namespace/:topic/unload|operation/unloadTopic?version=2.6.3}

#### Java

```java

String topic = "non-persistent://my-tenantmy-namespace/my-topic";
admin.nonPersistentTopics().unload(topic);

```

