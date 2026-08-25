---
title: "Why Kafka Partitions Are Limited per Broker"
date: 2026-08-25T12:00:00+02:00
draft: false
tags: ["kafka", "confluent"]
categories: ["development"]
ShowToc: true
TocOpen: true
weight: 2
---

A common question when designing a Kafka cluster is: how many partitions can a broker support? There is no single answer that applies to every cluster. Kafka topics are divided into partitions so that producers and consumers can work in parallel, but every partition also creates work and state for the brokers that host it. The practical limit depends on the broker resources, replication factor, Kafka version, and operational requirements, so partition counts should be treated as an operational capacity limit, not only as a logical design choice.

## A partition is broker state

A partition is not just a number in topic metadata. On a broker, each partition normally has:

* A log directory containing segment files and indexes.
* An entry in the broker's metadata and replica state.
* A leader or follower replication task.
* Network and disk activity for producing, fetching, and replicating records.
* Recovery work after a restart or an unclean shutdown.

A topic with many partitions therefore consumes more broker resources even when traffic is low. Empty partitions still have metadata, files, and lifecycle costs.

## Replication makes the number larger

The useful number to calculate is usually the number of partition replicas on each broker, not just the number of topic partitions.

A rough cluster-wide estimate is:

```text
partition replicas = topic partitions * replication factor
replicas per broker = partition replicas / broker count
```

For example, a topic with 1,000 partitions and a replication factor of 3 creates 3,000 replicas. In a six broker cluster, that is approximately 500 replicas per broker, assuming an even distribution.

The actual distribution can be temporarily uneven during reassignment, broker maintenance, or failure.

## KRaft improves cluster scaling, not partition cost

KRaft does not increase the number of partitions that a broker can support by itself. The per-node partition limit still depends on the broker's available memory, storage, file descriptors, CPU, and network capacity.

KRaft does improve how Kafka manages metadata. KRaft controllers store cluster metadata in a replicated metadata log, including topics, partitions, and replica state. This avoids the separate ZooKeeper metadata path and makes metadata propagation and controller failover more efficient.

There are therefore two limits to consider:

* The per-node partition count limit, which is constrained by the resources of each broker.
* The cluster-wide partition limit, which defines the total scale of the Kafka cluster.

KRaft is designed to handle a larger number of partitions at cluster level, but the cluster-wide limit still depends on adding enough nodes to provide the required capacity. In other words, KRaft can improve the cluster's ability to manage metadata, but it does not turn additional partitions into free capacity on an existing broker.

## What becomes expensive?

### Memory and metadata

Kafka keeps metadata about partitions and replicas in broker and controller processes. The exact memory usage depends on the Kafka version, configuration, and workload, but a large partition count increases the amount of metadata that must be loaded, propagated, and managed.

### File descriptors and files

Each partition has directories, log segments, and index files. A broker with many partitions may need substantially more file descriptors and filesystem operations. This is one reason operating-system limits and log-directory performance matter when increasing partition counts.

### Recovery time

After a broker restarts, Kafka must discover logs and bring replicas back into service. More partitions mean more replica state to load and more recovery operations. A broker can be available while some replicas are still recovering, which can leave the cluster under-replicated for longer.

### Leader elections and controller work

Partitions have leaders. When a broker fails, leadership must be reassigned for the affected partitions. A failure involving thousands of leaders produces more metadata changes and more client activity than a failure involving a small number of partitions.

### Replication and rebalancing

Replication traffic is driven by data volume, but partition count adds scheduling and bookkeeping overhead. Moving a large number of replicas between brokers also takes time and can compete with normal client traffic.

## More partitions are not always more throughput

Partitions provide parallelism only when the workload can use it. For example, a consumer group can process multiple partitions concurrently, but one consumer instance normally processes a partition at a time.

Adding partitions can fail to improve performance when:

* Consumers are already limited by CPU, downstream services, or network bandwidth.
* Producers use keys that concentrate records on a small number of partitions.
* The workload is limited by disk throughput rather than partition parallelism.
* The extra partitions increase coordination and recovery costs more than they increase useful concurrency.

Partition count is also difficult to reduce later. Kafka supports increasing a topic's partition count, but reducing it generally requires creating a new topic and migrating the data.

## Practical sizing questions

Before creating many partitions, estimate:

1. Required producer and consumer parallelism.
2. Expected traffic and retention, including peak traffic.
3. Replication factor and the resulting replicas per broker.
4. The number of brokers and expected future growth.
5. Recovery and reassignment time that the team can tolerate.
6. File-descriptor, storage, memory, and network limits.

A useful starting point is to choose enough partitions for the required consumer concurrency and expected growth, then validate the result with a load test. Avoid multiplying the count by an arbitrary large factor just because partitions cannot easily be reduced.

## My takeaways

Any Kafka-as-a-Service offering should enforce a limit on the number of partitions per broker. It should also limit other resource-sensitive parameters, such as client connections and requests per second. These limits help prevent sudden spikes from destabilizing the cluster.

The service should also enforce a maximum number of partitions per topic and provide client quotas. Without these safeguards, a single service or tenant could create too many partitions or consume a disproportionate share of cluster resources, monopolizing the cluster and affecting other users.

## Conclusion

Kafka does not impose a single universal partition limit that is appropriate for every cluster. The practical limit depends on the broker resources, Kafka version, replication factor, traffic pattern, operational tooling, and recovery requirements.

The important distinction is this: partitions provide parallelism, while replicas consume broker capacity. Size both deliberately, monitor replicas per broker, and treat partition growth as an operational change rather than a harmless topic setting.

### Documentation

* [Kafka documentation](https://kafka.apache.org/documentation/)
* [Confluent documentation: Kafka partitions](https://docs.confluent.io/kafka/design/partitions.html)
* [Confluent documentation: Scaling Kafka with KRaft](https://docs.confluent.io/platform/current/kafka-metadata/kraft.html#scaling-ak-with-kraft)
