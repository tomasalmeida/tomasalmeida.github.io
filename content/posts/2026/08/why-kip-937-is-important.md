---
title: "Why KIP-937 Is Important for Kafka Message Timestamps"
date: 2026-08-28T12:00:00+02:00
draft: false
tags: ["kafka", "confluent"]
categories: ["development"]
ShowToc: true
TocOpen: true
weight: 2
---

Kafka records have timestamps, and most of the time we barely think about them. That changes quickly when a producer sends nanoseconds instead of milliseconds. Suddenly, a record can appear to come from hundreds of years in the future. Kafka may accept it, while the original mistake stays hidden until it starts affecting retention, log segments, or downstream consumers.

[KIP-937](https://cwiki.apache.org/confluence/spaces/KAFKA/pages/255071414/KIP-937+Improve+Message+Timestamp+Validation) addresses this problem directly. It separates timestamps in the past from timestamps in the future, so operators can allow legitimate historical data without also accepting dates that are clearly implausible.

The frustrating part is how small the original bug can be. A single conversion mistake in a shared producer helper may be enough to fill a topic with records dated far beyond the current time. The producer team may see successful sends, while the operations team later sees segments that refuse to age out and stream-processing jobs that produce surprising results.

## The problem with one symmetric limit

Kafka historically used `log.message.timestamp.difference.max.ms` to validate the absolute difference between a record timestamp and the broker's current time:

```text
time_difference = absolute(record_timestamp - broker_timestamp)

if time_difference > difference_max:
    reject the record
```

The rule is simple, but it treats both directions of time as equivalent. A record from last week and a record from the year 2326 are both just records with a large difference from the broker clock.

That is not a very useful operational assumption. Historical timestamps are often intentional: replaying archived events, importing data, and rebuilding topics all need them. A timestamp far in the future is more likely to mean that a producer has a clock problem, used the wrong unit, or applied a bad transformation.

The old default was `Long.MAX_VALUE`, which effectively allowed timestamps almost anywhere in the representable range. That was convenient for compatibility, but it left Kafka with very little protection against future-dated records.

## Why future timestamps cause real problems

Kafka uses timestamps in several parts of log management. Depending on the topic configuration, timestamps can influence retention and segment rolling. A record dated far in the future can therefore make a log segment look newer than it really is.

That can lead to:

* Log segments that do not roll or expire when operators expect.
* Retained data consuming more disk space than planned.
* Confusing monitoring and debugging, because event time no longer resembles ingestion time.
* Downstream applications making incorrect decisions based on apparently future events.
* Difficult incident investigation when a small producer configuration error changes broker behavior.

The problem is not that Kafka supports event timestamps. Event time is valuable. The problem is accepting an obviously wrong future timestamp without a clear boundary around it.

## What KIP-937 changes

KIP-937 introduces two cluster-wide settings:

* `log.message.timestamp.before.max.ms` controls how far a record may be behind the broker clock.
* `log.message.timestamp.after.max.ms` controls how far a record may be ahead of the broker clock.

The validation becomes directional:

```text
if record_timestamp <= broker_timestamp:
    difference = broker_timestamp - record_timestamp
    validate against before.max.ms
else:
    difference = record_timestamp - broker_timestamp
    validate against after.max.ms
```

That separation is the useful part of the proposal. A team can allow a long historical replay window while keeping the future window small.

For example, a workload that imports events from the previous year might need a generous past limit, but only a small amount of clock skew into the future. That is a much better fit than forcing both cases through one absolute value.

## The compatibility story is deliberately gradual

The new settings initially use the old maximum value as their default. This preserves existing behavior for clusters that upgrade without changing their configuration. The old `log.message.timestamp.difference.max.ms` setting is deprecated, but the initial defaults avoid suddenly rejecting producers that were previously accepted.

The KIP also describes a future direction in which the default future allowance becomes one hour, or `3600000` milliseconds. Kafka will warn when it sees future timestamps beyond that proposed threshold so that operators can find problematic producers before a stricter default arrives.

In practice, the migration does not have to be dramatic. It can happen gradually:

1. Upgrade Kafka without changing the effective timestamp policy.
2. Observe warnings and identify producers with future-dated records.
3. Decide how much historical data each workload legitimately needs.
4. Configure separate past and future limits.
5. Prepare producers and operations for the eventual stricter future default.

Changing the configuration is not risk-free, though. After KIP-937, a future timestamp can be rejected with error code `32`, `INVALID_TIMESTAMP`. If one record in a batch fails timestamp validation, Kafka rejects the entire batch.

## Why the error matters

A rejected record is much easier to fix when the producer gets a useful explanation. The validation error includes the record timestamp, its offset, and the acceptable range calculated from the broker time and the two configured limits.

Common mistakes become much easier to diagnose too. A producer using Unix nanoseconds instead of milliseconds, for example, sends a value many orders of magnitude too large. The error points toward the timestamp conversion instead of leaving operators to guess from strange segment or retention behavior.

The rejection also happens before the message is appended to the active log segment. Kafka is therefore protecting the log before invalid temporal data becomes part of its on-disk lifecycle.

## `CreateTime` and `LogAppendTime`

The change is most relevant when the topic uses `CreateTime`, where the producer supplies the record timestamp. With `LogAppendTime`, the broker overwrites the timestamp with its own current time and does not perform this same validation against the producer-provided value.

That distinction is worth remembering when choosing the topic timestamp type:

* `CreateTime` preserves producer event time and is useful for event-time processing, but producer clocks and timestamp units must be managed carefully.
* `LogAppendTime` gives the broker control over the timestamp and is useful when ingestion time is the desired operational reference.

KIP-937 improves validation for producer-supplied timestamps. It does not remove the design decision about which notion of time a topic should represent.

## Who should care about timestamp validation?

### Operators and administrators

Retention is based on the timestamps Kafka uses to determine the age of log segments. If a producer writes a record with a timestamp far in the future, Kafka can treat the segment containing that record as newer than it really is. As a result, data that is genuinely old from an operational point of view may remain on disk because the broker does not consider the segment eligible for deletion yet.

This can undermine the retention policy administrators configured for the topic. Disk usage can grow beyond forecasts, alerts can arrive late, and removing old data may require correcting the producer and waiting for segments to roll before normal cleanup resumes. A well-configured `log.message.timestamp.after.max.ms` limits the chance that one malformed or misconfigured timestamp will extend the apparent lifetime of old data.

![Diagram showing how a future-dated Kafka record can make a log segment appear newer and delay retention cleanup](retention-future-timestamp.svg "A future timestamp can delay Kafka retention cleanup")

*A future-dated record can make a segment appear newer than it really is, keeping old data on disk beyond the intended retention window.*

This is why operators and administrators should care. Timestamp validation protects the relationship between the retention policy and the storage actually used by the cluster. A sensible future limit exposes bad data early, before it turns into disk pressure, delayed cleanup, or confusing capacity forecasts. The past limit needs equal attention: set it too aggressively and a legitimate replay or backfill can fail when the team needs it most.

### Developers

For developers, the responsibility starts with producing timestamps Kafka can interpret correctly. Problems commonly come from:

* Sending nanoseconds instead of milliseconds since the Unix epoch.
* Using a clock that is significantly ahead of the broker.
* Reusing an invalid timestamp during a replay or backfill.
* Applying a transformation that changes the timestamp's unit or meaning.
* Assuming that a future timestamp is harmless because the record itself is accepted by the client.

These mistakes can cause `INVALID_TIMESTAMP` errors, reject an entire record batch, or make downstream event-time processing calculate the wrong windows and delays. In Kafka Streams, one bad event timestamp can put a record in the wrong window or change how grace periods are handled. In Apache Flink, it can push a watermark forward too early and close a window before the remaining events arrive. The reverse problem is possible too: a timestamp far in the past can hold the watermark back while the job waits for an event time that never appears. Once the bad timestamp reaches Kafka, it can also contribute to the retention problems described above.

#### How one wrong event can affect a stream

Picture a five-minute aggregation receiving events around `10:00`. One event really happened at `10:01`, but the producer sends its timestamp in the wrong unit or gives it a date far in the future. The stream processor has no way to know that the value is wrong; it treats the timestamp as truth. Kafka Streams may put the event in another window, change the aggregate, or consider the window complete at the wrong time. Flink may build a watermark from that future event and close the `10:00` window before the other valid events arrive. Those events are then late, dropped, sent to a late-events side output, or handled through recovery logic.

The opposite error is harmful as well. A timestamp far in the past can hold event-time progress behind the workload, delaying windows, timers, joins, and alerts. One malformed event can make the results look like an application bug even though the processing code is doing exactly what the timestamps tell it to do.

Developers should document whether a topic carries event time or ingestion time, use a shared timestamp conversion library, test boundary and replay cases, and monitor producer errors. Getting this small detail right keeps application behavior predictable and stops a local data-format mistake from becoming a broker-storage problem.

## What operators should check

Before enabling a stricter future limit, inspect both producers and existing topic policies:

* Confirm that clients send milliseconds since the Unix epoch.
* Check whether producer hosts synchronize their clocks.
* Identify replay and backfill jobs that intentionally use old event timestamps.
* Review topics using `CreateTime` and their retention and segment-roll settings.
* Monitor `INVALID_TIMESTAMP` responses during rollout.
* Set `log.message.timestamp.before.max.ms` and `log.message.timestamp.after.max.ms` based on workload needs, not one universal guess.

A one-hour future window may be sensible for many applications, but it is not automatically right for every system. Delayed processing, disconnected devices, and scheduled event production may need more room. The important improvement is that the choice is now visible and independent from the historical-data policy.

## The larger design lesson

KIP-937 is more than a new pair of broker configurations. It recognizes that “old” and “future” are different data-quality problems. Kafka should be permissive enough to support replay and historical imports, while still making accidental future data difficult to introduce silently.

The proposal also points toward a broader limitation in Kafka's current message format: a record has one timestamp whose meaning depends on whether the topic uses create time or append time. The KIP identifies capturing both client and broker timestamps as future work, which could eventually provide better observability and more precise retention decisions.

## Conclusion

Timestamp validation sits between application correctness and broker operations. That boundary is easy to ignore until a small producer bug reaches several systems at once: storage behavior changes, retention expectations stop matching reality, monitoring becomes confusing, and event-time logic starts producing unexpected results.

KIP-937 gives Kafka a clearer policy model: configure how far records may be in the past and how far they may be in the future independently. Its compatibility-focused rollout gives teams time to find timestamp problems, and its directional validation makes the eventual policy easier to reason about. For Kafka operators and application owners, that is a small configuration change with a meaningful improvement in data integrity.

### Documentation

* [KIP-937: Improve Message Timestamp Validation](https://cwiki.apache.org/confluence/spaces/KAFKA/pages/255071414/KIP-937+Improve+Message+Timestamp+Validation)
* [Kafka documentation: `log.message.timestamp.type`](https://kafka.apache.org/documentation/#brokerconfigs_log.message.timestamp.type)
* [Kafka documentation: `log.message.timestamp.difference.max.ms`](https://kafka.apache.org/documentation/#brokerconfigs_log.message.timestamp.difference.max.ms)
* [KIP-32: Add timestamps to Kafka message](https://cwiki.apache.org/confluence/display/KAFKA/KIP-32+-+Add+timestamps+to+Kafka+message)
