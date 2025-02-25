---
id: admin-api-non-partitioned-topics
title: Managing non-partitioned topics
sidebar_label: "Non-Partitioned topics"
original_id: admin-api-non-partitioned-topics
---


You can use Pulsar's [admin API](admin-api-overview) to create and manage non-partitioned topics.

In all of the instructions and commands below, the topic name structure is:

```shell

persistent://tenant/namespace/topic

```

## Non-Partitioned topics resources

### Create

Non-partitioned topics in Pulsar must be explicitly created. When creating a new non-partitioned topic you
need to provide a name for the topic.

:::note

By default, after 60 seconds of creation, topics are considered inactive and deleted automatically to prevent from generating trash data.
To disable this feature, set `brokerDeleteInactiveTopicsEnabled`  to `false`.
To change the frequency of checking inactive topics, set `brokerDeleteInactiveTopicsFrequencySeconds` to your desired value.
For more information about these two parameters, see [here](reference-configuration.md#broker).

:::

#### pulsar-admin

You can create non-partitioned topics using the [`create`](reference-pulsar-admin.md#create-3)
command and specifying the topic name as an argument.
Here's an example:

```shell

$ bin/pulsar-admin topics create \
  persistent://my-tenant/my-namespace/my-topic

```

:::note

It's only allowed to create non partitioned topic of name contains suffix '-partition-' followed by numeric value like
'xyz-topic-partition-10', if there's already a partitioned topic with same name, in this case 'xyz-topic', and has
number of partition larger then that numeric value in this case 11(partition index is start from 0). Else creation of such topic will fail.

:::

#### REST API

{@inject: endpoint|PUT|/admin/v2/:schema/:tenant/:namespace/:topic|operation/createNonPartitionedTopic?version=2.6.4}

#### Java

```java

String topicName = "persistent://my-tenant/my-namespace/my-topic";
admin.topics().createNonPartitionedTopic(topicName);

```

### Delete

#### pulsar-admin

Non-partitioned topics can be deleted using the [`delete`](reference-pulsar-admin.md#delete-4) command, specifying the topic by name:

```shell

$ bin/pulsar-admin topics delete \
  persistent://my-tenant/my-namespace/my-topic

```

#### REST API

{@inject: endpoint|DELETE|/admin/v2/:schema/:tenant/:namespace/:topic|operation/deleteTopic?version=2.6.4}

#### Java

```java

admin.topics().delete(persistentTopic);

```

### List

It provides a list of topics existing under a given namespace.  

#### pulsar-admin

```shell

$ pulsar-admin topics list tenant/namespace
persistent://tenant/namespace/topic1
persistent://tenant/namespace/topic2

```

#### REST API

{@inject: endpoint|GET|/admin/v2/:schema/:tenant/:namespace|operation/getList?version=2.6.4}

#### Java

```java

admin.topics().getList(namespace);

```

### Stats

It shows current statistics of a given topic. Here's an example payload:

The following stats are available:

|Stat|Description|
|----|-----------|
|msgRateIn|The sum of all local and replication publishers’ publish rates in messages per second|
|msgThroughputIn|Same as msgRateIn but in bytes per second instead of messages per second|
|msgRateOut|The sum of all local and replication consumers’ dispatch rates in messages per second|
|msgThroughputOut|Same as msgRateOut but in bytes per second instead of messages per second|
|averageMsgSize|Average message size, in bytes, from this publisher within the last interval|
|storageSize|The sum of the ledgers’ storage size for this topic|
|publishers|The list of all local publishers into the topic. There can be anywhere from zero to thousands.|
|producerId|Internal identifier for this producer on this topic|
|producerName|Internal identifier for this producer, generated by the client library|
|address|IP address and source port for the connection of this producer|
|connectedSince|Timestamp this producer was created or last reconnected|
|subscriptions|The list of all local subscriptions to the topic|
|my-subscription|The name of this subscription (client defined)|
|msgBacklog|The count of messages in backlog for this subscription|
|msgBacklogNoDelayed|The count of messages in backlog without delayed messages for this subscription|
|type|This subscription type|
|msgRateExpired|The rate at which messages were discarded instead of dispatched from this subscription due to TTL|
|consumers|The list of connected consumers for this subscription|
|consumerName|Internal identifier for this consumer, generated by the client library|
|availablePermits|The number of messages this consumer has space for in the client library’s listen queue. A value of 0 means the client library’s queue is full and receive() isn’t being called. A nonzero value means this consumer is ready to be dispatched messages.|
|replication|This section gives the stats for cross-colo replication of this topic|
|replicationBacklog|The outbound replication backlog in messages|
|connected|Whether the outbound replicator is connected|
|replicationDelayInSeconds|How long the oldest message has been waiting to be sent through the connection, if connected is true|
|inboundConnection|The IP and port of the broker in the remote cluster’s publisher connection to this broker|
|inboundConnectedSince|The TCP connection being used to publish messages to the remote cluster. If there are no local publishers connected, this connection is automatically closed after a minute.|

#### pulsar-admin

The stats for the topic and its connected producers and consumers can be fetched by using the [`stats`](reference-pulsar-admin.md#stats) command, specifying the topic by name:

```shell

$ pulsar-admin topics stats \
  persistent://test-tenant/namespace/topic \
  --get-precise-backlog

```

#### REST API

{@inject: endpoint|GET|/admin/v2/:schema/:tenant/:namespace/:topic/stats|operation/getStats?version=2.6.4}

#### Java

```java

admin.topics().getStats(persistentTopic, false /* is precise backlog */);

```

