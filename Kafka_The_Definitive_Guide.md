# Chapter 1

## Publish / Subscribe
- Publisher classifies and publishes message; receiver subscribes to that class of message; broker facilitates
- Starting State: Many services writing directly to multiple receivers. Hard to trace.
- Intermediate: Centralized service receives all messages from services, sends them out to receivers, but many teams make their own intermediaries. Lots of duplication potential.
- Kafka: Durable, specialized centralized service to pub/sub.

## Messages and Batches
- Messages are untyped / unformatted from Kafka's perspective. An optional key can be provided in order to partition data.
- Messages are written in batches to lessen network overhead. Trade-off between network overhead and when a message arrives, though, since a batch may be held slightly longer than a single record. Batches are generally compressed for transfer/storage.

## Schemas
- Schema is recommended so that messages are easily understood.
- Can use JSON / XML / etc, though these do lack built-in type and version handling. Apache Avro also possible, which handles both.
- MUST keep schemas well-defined so that you don't need to publish all receivers in order to publish a subscriber update.

## Topics and Partitions
- _Topics_ categorize messages, similar to a folder or database table.
- Topics can be broken into _Partitions_, an append-only bounded group of messages, similar to a single day's log file for an ongoing log source.
- Since a Topic will generally have multiple partitions, there's no time-ordering guarantee across the entire Topic. Inside a Partition, though, append-only guarantees time ordering.
- Partitions allow redundancy and scale - each partition can be hosted on a separate server, scaling horizontally.
- A _Stream_ generally refers to a single Topic.

## Producers and Consumers

- _Producers_ create and publish messages. Generally won't care about partitioning, but if they do then they'll take on the key's hashing and the mapping of those hashes using a partitioner.
- _Consumers_ subscribe to and read messages. Subscribes to one or more topics and reads the messages in the order produced.
  - Consumers track which messages have been read using _offset_ metadata, which Kafka adds to each message as it is produced. Offsets are unique to a given partition. If a Consumer restarts, it can find its place again simply by using the offset.
- _Consumer Groups_ are groups of Consumers that work together to read a Topic. 
  - Assumes that each Partition is only consumed(_owned_) by a single Consumer in the group.
  - Groups allow horizontal scaling on a larger number of messages. If a single Consumer fails, the others can rebalance to take over for the missing Consumer.

## Brokers and Clusters

- _Broker_: A single Kafka server. Receives messages from Producers, gives them offsets, stores them. Responds to Consumers, serving fetch requests for Partitions.
- _Cluster_: A group of Brokers.
- _Cluster Controller_: An automatically-elected Broker in a Cluster that performs administrative overhead, like assigning partitions to Brokers and monitoring for Broker failures.
- A partition is owned by a single broker. That broker is the _Leader_ for the partition. If a partition is assigned to multiple brokers, it will be replicated so that if the leader fails, another can take its place. All broker operations for that partition must connect to the leader.
- _Retention_: The durable storage of messages for some period of time.
  - Brokers have a default retention setting for topics, either based on a time period or until the topic reaches a certain size. Once limit is reached, messages are expired and deleted. 
  - Individual topics can have their own retention settings - A tracking topic might have a multi-day policy, while app metrics might only be for a few hours.
  - Individual topics can be _log compacted_ so that Kafka retains only the latest message for a given key(e.g., when you only care about the last update to a message).

## Multiple Clusters
- Allow for segregation of data for security, multiple datacenters, etc.
- For multiple datacenters, generally required that messages be copied between clusters. This can be handled by _MirrorMaker_ - A Kafka Consumer / Producer that consumes from one cluster and produces for another.

## Why Kafka?
- Kafka handles multiple producers, allowing easy aggregation and consistent data. 
- Kafka handles multiple consumers - Designed for multiple consumers to read a stream without interfering with each other. 
- Disk-based Retention - Consumers don't need to work in real-time since messages will be retained. If a consumer falls behind, no danger of losing data. Consumers can be upgraded without concern for data loss.
- Scalable by number of brokers, number of clusters, etc. Expansions can be performed while the cluster is online without hurting availability, and a cluster can handle the failure of a single broker.

## The Data Ecosystem - Use Cases
- Activity Tracking - Frontend apps generate messages about user actions, pub to one or more topics, consumed by applications that can generate reports, etc.
- Messaging - Notifications(e.g., email) can be produced without the producer worrying about how they'll be sent. Consumer can handle formatting / sending / etc.
- Metrics / Logging - Apps publish metrics; monitoring / alerting systems consume. 
- Commit Log - 

# Chapter 2 - Installing Kafka

## Up Front
- Choosing an OS: Java-based, so runs anywhere, needs JRE.
- _Zookeeper_: Stores metadata about the Kafka cluster as well as consumer client details.
  - Don't _need_ a Zookeeper install alongside, but trivial to do so and recommended.
  - Instructions for getting Zookeeper up and running are in book

## Installing a Broker

## Broker Configuration 
- General
  - `broker.id`: Integer id, defaults to 0. Must be unique within a Kafka cluster. One idea: if hostnames contain a number(web1, web2), make that the id.
  - `port`: Default is 9092.
  - `zookeeper.connect`: zookeeper url, e.g. localhost:2181/zookeeper
  - `log.dirs`: comma-separated list of local filepaths in which log segments will be stored. if more than one path is specified, will store in each dir in a least-used fashion _based on number of partitions_, not on disk space usage.
  - `num.recovery.threads.per.data.dir` - default is 1 thread per directory. Kafka maintains a pool of threads used 1) when starting normally, to open a partition's segments; 2) when starting after failure, to check and truncate each partition's segments; 3) when shutting down, to close log segments.
    - since these are only used at startup/shutdown, can bump threads to quickly restart a broker with lots of partitions.
    - the number is _per log directory_, so if `log.dirs` is 4 and this setting is 3, you will have 12 threads.
  - `auto.create.topics.enable`: default `true` sets the broker to automatically make a topic 1) when a producer starts writing messages to a topic; 2) when a consumer starts reading messages from a topic; 3) when any client requests metadata for a topic
    - This may not be desirable, since checking for existence of a topic will cause it to be created.
- Topic Defaults
  - `num.partitions`: default `1`. how many partitions a topic will be created with.
    - Number of partitions can only be increased, never decreased - if a topic requires fewer, must manually create to avoid inheriting this setting.
    - Important to balance this so that load is distributed across cluster as brokers are added. Can default to number of brokers in the cluster, allowing for even distribution, but can also balance load by having multiple topics.
    - Considerations for choosing the number of partitions
      - What consumer throughput do you need in bytes/s?
      - What is the max throughput you expect when consuming from a single partition? A partition will always be consumed by a single consumer, so if you know that your consumer has a throughput limit then that's your value.
      - Can do same exercise for producers, but not always needed since producers are generally faster.
      - If you are partitioning based on keys, adding partitions later can be hard. Calculate based on expected future usage, not current.
      - Consider # of partitions you will put on each broker, available diskspace, and network bandwidth per broker.
      - Each partition costs memory and broker resources and will increase time for leader elections, so don't wildly overestimate.
      - If you know target throughput and expected consumer throughput, target / consumer = number of partitions(e.g., want to write/read at 1GB/s from a topic, each consumer can handle 50MB/s, need 20 partitions)
      - If you _don't_ have these numbers, limit such that partition size is < 6GB/day
  - `log.retention.hours`: Defaults to expiring messages in one week.
    - `log.retention.minutes`; `log.retention.ms`: same as above. lowest unit size has precedence(ms>minute>hour)
    - Retention is done by examining last modified time(`mtime`) on each log segment. Generally this is the time a segment was closed and timestamp of last record, but when moving segments via admin tools the time will not be accurate and will result in excess retention.
  - `log.retention.bytes`: Expires messages by the total number of bytes retained by a given partition(8 partitions with a setting of 1GB => 8GB of retained data, max).
    - Since all retention is performed for an individual partition, not a topic, if you expand the number of partitions then you'll also increase max disk space usage(e.g., 9 partitions -> 9GB)
  - If you spec both size and time, messages may be expired when either criteria is met _not both_.
  - `log.segment.bytes`: How large a segment can be in bytes, default 1GB. A segment will never be considered for expiration until it is closed, so a smaller segment size slows down write efficiency but allows expiration sooner.
    - Useful if a topic has a low produce rate. If topic only gets 100MB/day and this setting is default 1GB, it will take 10 days to fill a segment. If `log.retention.ms` is default(1 week), you will wind up retaining 17 days of data - 10 days to close the segment, then another 7 before expiration takes hold.
    - Also affects fetching offsets by timestamp, as Kafka uses the creation and last modified time to serve these requests.
  - `log.segment.ms`: Amount of time before a segment should be closed, default is not set.
    - Expiring by time may lead to disk contention when multiple segments are closed simultaneously e.g., for many log files that will never reach the size of `bytes`. Those files will always start the clock when the broker starts and then hit expiration at the same time.
  - If both ms and bytes are set, will close when either is hit.
  - `message.max.bytes`: Max size of a message that can be produced _after compression_, defaults to 1MB(1000000)
    - A producer that exceeds this limit will receive an error back.
    - Larger message size means that each broker thread will work longer on each request. Also increases the size of disk writes, impacting I/O.
    - Must be coordinated with `fetch.message.max.bytes` on consumers. If smaller than `fetch.message.max.bytes`, consumers that encounter large messages will fail to consume and get stuck. Same with `replica.fetch.max.bytes` on brokers in a cluster.

## Hardware

- Disk Throughput
  - Kafka messages must be persisted to disk when produced. Most clients will wait until a broker confirms that commit before responding with a success. _Faster disk writes -> less latency_
- Disk Capacity: Driven by message retention + at least 10% overhead. Also depends on replication strategy.
- Memory: More memory -> More segments stored in page cache -> faster reads -> inmproved throughput. Kafka itself requires very little(150k messages at 200MB/s -> 5GB heap). Don't colo with another hungry application, as they will need to share page cache.
- Networking: Driven by number of producers writing / number of consumers reading / replication. If an interface becomes saturated, can cause cluster to fall behind.
- CPU: Used for compression and broker overhead. Not as important as disk/mem.
- Cloud-based
  - Instance type driven by latency / retention needs. Low latency? I/O optimized, etc. m4(greater retention, lower throughput due to EBS) or r3(better throughput via SSD, but may limit data amount) are common on AWS. next step is i2 or d2, much more expensive.

## Kafka Clusters

- Multiple servers allows horizontal scale, replication(guards against data loss, allows in-place maintenance). 
- How many brokers?
  - (Max Bytes Retained) / (Max Bytes Stored by Broker) => 10GB of total retention / 2 GB retention per broker = 5 brokers.
  - Replication will increase storage by replication factor. Replicated to one other server -> 10 brokers to consume all.
  - How much can the NICs handle? Can they handle bursts? If NIC on a broker is used at 80% capacity at peak and there are two consumers, the consumers will not be able to keep up with traffic without two brokers. Replication counts as an additional consumer.
- Broker Configuration - Allowing multiple brokers to join a cluster
  - All brokers must spec same `zookeeper.connect`
  - All brokers must have unique `broker.id` - if two try to join with same id, the second will log an error and fail.
- OS Tuning: Tweaking virtual memory / networking can improve perf
- Virtual Memory
  - Best to avoid swap file usage due to perf hit. Swap usage indicates there's not enough memory being allocated to the page cache. Setting `vm.swappiness` to a low value, like `1`, will prefer using swap space rather than dropping pages from the page cache. Best not to use `0`, as after kernel 3.5 that means "never swap".
  - Reduce `vm.dirty_background_ratio` from default 10 to 5
  - Increase `vm.dirty_ratio` above default of 20 to 60-80. Can go higher if you have replication to guard against sys fails.
- Disk
  - ext4 or xfs, with xfs outperforming ext4 without tuning
  - set `noatime` mount option to avoid updating `atime` on every read. `atime` is never used by Kafka.
- Networking
  - Recommended to adjust default / max memory allocation for send / receive buffers per socket(`net.core.wmem_default`=131072, `net.core.rmem_default`=131072, `net.core.wmem_max`=2097152, `net.core.rmem_max`=2097152, 131072=128KB, 2097152=2MB) -> Increases large transfer performance
  - `net.ipv4.tcp_wmem`, `net.ipv4.tcp_rmem`: TCP socket settings, can't be larger than `net.core.wmem_max` / `net.core.rmem_max`. e.g., `4096 65536 2048000` => 4KB min, 64KB default, 2MB max buffer. Increase maxes if require more buffering.
  - `net.ipv4.tcp_window_scaling`=1: Allows clients to transfer data more efficiently, allows buffering on broker's side.
  - `net.ipv4.tcp_max_syn_backlog`>1024: Allows more simulataneous connections
  - `net.core.netdev_max_backlog`>1000: Helps during bursts of network traffic by allowing more packets to be queued

## Production Concerns

- Garbage Collector Settings
  - MaxGCPauseMilis: Preferred pause time per G1 GC cycle. Default `200ms`. Not fixed - GC can exceed if needed. Scheduler will attempt to set freq of GC such that each GC takes 200ms.
  - InitiatingHeapOccupancyPercent: % of heap that can be in use before GC starts a cycle. Default of 45 -> No collection til 45% used.
  - For Kafka, can tune these lower. On a server with 64GB mem and a 5GB heap, MaxGCPauseMillis can be 20ms; InitiatingHeapOccupancyPercent = 35
  - Kafka uses mark-and-sweep by default, not G1 collector. Can pass through `+UseG1GC`, `+DisableExplicitGC` to swap over
- Datacenter Layout - Kafka can assign partitions to broker by rack such that replicas for a given partition never share a rack(so if a rack goes down, still ok). Install each broker in a cluster in a different rack so that they don't share power / network / etc.
- Colocating Apps on Zookeeper
  - Zookeeper stores metadata about brokers / topics / partitions. Writes to Zookeeper only on updates to membership of consumer groups or chanegs to the cluster itself. Minimal traffic, so a single cluster doesn't really need a dedicated Zookeeper ensemble. Many deployments will use a single ensemble for multiple clusters.
  - Consumers can be configured to use either Zookeeper or Kafka for offset commits; can also configure interval between commits. If consumer uses Zookeeper for offsets, each consumer will write to Zookeeper at every interval for every partition it consumes. Default to 1m, as this is the period of time over which a consumer group will read duplicate messages in the case of a consumer failure. These commits can cause a lot of Zookeeper traffic - May need to increase commit interval if ensemble isn't able to handle.
  - *Recommendation*: Recommended to use Kafka for offset commits rather than Zookeeper.
  - *Recommendation*: Don't share a single ensemble between multiple clusters and other applications. Kafka is dependent on Zookeeper latency. +latency = unpredictable brokers - multiple offline simultaneously, etc.

# Chapter 3: Kafka Producers - Writing Messages to Kafka

- Every Kafka usage will have a producer app, a consumer app, or an app that does both
  - e.g., Credit card tx => Frontend produces transaction event when payment is attempted; back-end app consumes, produces event spec'ing whether approved or denied; frontend consumes response and acts; separate app consumes both events and stores them to a database
- List of 3rd party clients available [here](https://cwiki.apache.org/confluence/display/KAFKA/Clients)

## Producer Overview

- Use Cases: recording user actions for auditing/analysis, recording metrics, storing logs, etc.
- Considerations: Can we lose some messages? Are we ok with duplication? Are there throughput/latency requirements?
- Flow
  - Create a record - Must include the topic and a value; Optionally, a partition and key.
  - Record is serialized and sent to partitioner
  - If partition is set, partitioner uses it; else, partitioner chooses(usually based on key)
  - After partition is selected, producer batches it with other records heading to the same topic and partition - a separate thread is responsible for sending these to brokers
  - Broker receives messages, responds with `RecordMetadata`(topic; partition; offset) or an error.

## Constructing a Producer(Java)

- (required)`bootstrap.servers`: List of host:port pairs of brokers that the producer will use to establish initial connection to the Kafka cluster. Doesn't need to include all brokers since it will get more info after initial connect, but should be 2+ in case one broker goes down.
- (required)`key.serializer`: Name of class used to serialize record keys.
- (required)`value.serializer`: Name of class used to serialize record values.
- Other props [here](https://kafka.apache.org/documentation.html#producerconfigs)
- Producer supports Fire-and-forget; Synchronous send; Async send

## Configuring Producers

- [Full list](http://kafka.apache.org/documentation.html#producerconfigs)
- `acks`: # of partition replicas that must receive record before the write is considered successful. Fewer => Higher probability of loss.
  - `0` => No ack, fire and forget
  - `1` => Success response from broker as soon as leader replica receives the message. If the message can't be written to leader, producer receives an error and can retry. Can still lose a message if the leader crashes and a replica without the message is elected as new leader(unclean leader election).
  - `all` => Success response from broker once all replicas have received.
- `buffer.memory`: Amount of memory producer will use to buffer waiting messages. Once exhausted, `send` may block or throw an exception.
- `compression.type`: Default: uncompressed. `snappy`, `gzip`, `lz4`.
- `retries`: # of times to retry on _transient_ error(non-transient errors, like "Message too large", will notbe retried). default wait of 100ms, controlled by `retry.backoff.ms`. 
- `batch.size`: amount of memory _in bytes_ used per batch. Producer _will not_ always wait for a full batch before sending. 
- `linger.ms`: ms to wait for additional messages before sending batch. default is `0` => send when a thread is available.
- `client.id`: string to identify messages sent from client, used for logging, metrics, quotas.
- `max.in.flight.requests.per.connection`: # messages producer will send without receving a response. Too high can reduce batching. `1` => guaranteed in-order delivery to broker even when retries occur.
- `timeout.ms`, `request.timeout.ms,` and `metadata.fetch.timeout.ms`: ms to wait for in-sync replicas to ack the message for `acks` setting; ms to wait when sending data; ms to wait when fetching metadata.
- `max.blocks.ms`: ms to block when sending / requesting metadata, which will block if buffer is full or metadata unavailable.
- `max.request.size`: max send size, often match `message.max.bytes` on broker side.
- `receive.buffer.bytes` and `send.buffer.bytes`: TCP send/recv buffers. `-1` => OS default. Increase as latency goes up / bandwidth down.

## Ordering Guarantees

- Message order preserved _within partition_.
- `retries` > 0, `max.in.flight.requests.per.connection` > 1 => Broker may fail first batch, succeed with second, retry first, so order not guaranteed.
- `retries` > 0, `max.in.flight.requests.per.connection` = 1 => while batch is retrying, additional messages are not sent. *Low throughput*

## Serializers

- Use what you want: Custom, Avro, Thrift, Protobuf
- Avro: Schema in json, binary serialization(each record includes schema). When json schema changes, old records can still be read successfully.
- Avro + Schema Registry: Schema in registry, binary serialization(each record includes SchemaId).

## Partitions

- All messages with same Key will go to same partition. Key can be null, but usually won't be.
- Null key => Round-robin partition balance.
- Key => Hashed key => Map to partition. Mapping is only consistent if number of partitions is consistent; once add a partition, bucketing will change, so mapping will change.
- Can create custom partitioners.

## Old Producer APIs(Scala)

# Chapter 4 - Kafka Consumers

## Consumer Concepts

### Consumers and Consumer Groups

- Consumer groups: Enable multiple consumers to subscribe to the same topic. Each consumer will receive messages from one or more of the topic's partitions, no overlap.
- More consumers than partitions? Some will be idle.
- More partitions enables better horizontal scale-out, as supports more consumers.
- Individual apps should have their own Consumer Groups to ensure they get the full message stream. No perf impact of multiple Groups on one Topic.
 
### Consumer Groups and Partition Rebalance

- **Rebalance**: Triggered by adding or removing a consumer(even if one crashes); or when the Topics consumed by a Group are modified(e.g., new Partitions are added). Partitions once read by the original Consumer(s) are split off to the new one.
- Two types of Rebalance
  - Eager: All consumers stop consuming, rejoin the Group, get new Partition assingments. Consumers are completely unavailable.
  - Cooperative / Incremental: A subset of Partitions are reassigned. Consumers continue to read from Partitions that aren't affected.
    - Group Leader tells Consumers they will lose ownership of some partition(s) -> Consumers stop consuming these partition(s) -> Leader assigns orphaned partition(s).
    - May take a few iterations to settle, but avoids full unavailability.
  - Unavailability time driven by size of Group
- Consumers presumed alive as long as they send **heartbeats**(background thread) to a designated Group Coordinator Broker to maintain group membership / Partition ownership. When heartbeats stop, session times out and Coordinator will trigger a rebalance.
  - If a Consumer crashes, Coordinator will take a few seconds to realize it's gone - Partitions previously owned by that Consumer will not be processed during this.
  - Consumer shut down cleanly informs Coordinator first, Coordinator rebalances immediately, so lower gap in processing.
  - Cfg options for heartbeat freq; sessions timeouts; etc.
- How does a Consumer join a Group(Initially or during Rebalance)? `JoinGroup` request to Coordinator. First Consumer becomes **Leader**: Receives a list of all live Consumers in the group from the Coordinator -> Assigns partitions using a `PartitionAssignor` -> Sends assignment list to the Coordinator -> Coordinator sends assignments to Consumers. Each Consumer only sees its own assignment - Only Leader has full list of Consumers and Assigments.

### Static Group Membership

- Default: Consumer Identity is transient - New _or_ rejoining Consumers are issued new Ids and receive new partitions through rebalance protocol.
- Static: Configure Consumer with `group.instance.id`. On initial join, will receive partitions as usual. If it crashes / shuts down, it will be re-assigned the same partitions it once had when it rejoins.
  - Two consumers with same Id will throw an error.
  - Static Consumers don't proactively leave the group when they shut down - Membership determined by `session.timeout.ms`. Must be high enough to avoid rebalancing on a fast restart, low enough to allow re-assigment during significant downtime.
  - Pro: When app maintains local state / cache populated by assigned partitions, because re-creating that cache could be costly if done on every consumer restart.
  - Con: Partitions will not be re-assigned during brief Consumer restart, so Partitions will not be processed. Consumer must be able to handle backlog of messages when it comes back.
  
## Creating a Consumer

- Requires `bootstrap.servers`(conn string), `key.deserializer`, `value.deserializer`(classes that can turn byte arrays into an object)
  - Also generally `group.id` to specify Consumer Group. Rare to run without a group.
  
## Subscribing

- Accepts a *list* of topics or a RegEx that matches topic names.
  - If RegEx, a new topic that matches will trigger a rebalance to force Consumers to pick up the topic's partitions. Useful when e.g., replicating data between Kafka and another system; or for stream processing.
  - RegEx may filter on the client-side(as of Kafka 2.6), so Consumers will request a full topic / partition list at regular intervals. Large topic list + Many consumers => Significant overhead at Broker, Client, and network. Topic metadata may be larger than the bandwidth used to actually send data.

## The Polling Loop

- Generally an infinite loop that polls for data on a specified interval - If a Consumer stops polling, it will be considered dead and trigger a rebalance.
- `Poll` returns a list of records with the record's Topic, Partition, Offset, key, and value.
  - On first `Poll`, responsible for joining the group and receiving an assigment. `Poll` handles rebalances, too, including any callbacks.
- *Do not block within Poll*: `Poll` interval must be < `max.poll.interval.ms` or Consumers will be considered dead.
- Thread safety: Multiple consumers in the same group can't run in one thread; Multiple threads can't use the same consumer.
  - Approach 1: Consumer logic is wrapped into an object, start many threads for Consumers.
  - Approach 2: Consumer populates a queue of events, multiple worker threads feed from queue.
- `Poll(long)` vs `Poll(Duration)`: `long` blocks until it receives metadata needed, even if that exceeds timeout duration; `Duration` returns immediately if no data and obeys timeout restrictions.
  - Need to get metadata without consuming? Can't just use `Duration(0)` - May be able to listen to `OnPartitionAssignment`, triggers after metadata is received but before records.
  
## Configuration

- `fetch.min.bytes`: Default=1. If a Broker receives a request for records but has too few to fill min bytes, it will wait to send until it has enough. Reduces load on both Consumer and Broker(less back-and-forth for low-traffic topics). Useful to reduce Consumer CPU usage, Broker load with many Consumers.
- `fetch.max.wait.ms`: Default=500ms. Waits to respond to Consumer until Kafka has enough data, so results in 500ms latency for requests where not enough data to fill.
- Combining these => Kafka responds when it has enough data OR when the wait is hit.
- `fetch.max.bytes`: Default=50MB. Limits memory usage used of Consumer. *If first record batch exceeds this size, limit will be ignored so that Consumer can continue making progress.* Can also be configured by a Kafka Admin on the Broker side => Reduce disk reads and large network sends.
- `max.poll.records`: As stated
- `max.partition.fetch.bytes`: Default=1MB. Max number of bytes returned by `poll`. *No control over how many partitions are in response, so hard to make this work. Consider `fetch.max.bytes` instead*
- `session.timeout.ms` and `heartbeat.interval.ms`: Default=10s. If session is exceeded, Consumer will be considered dead => Rebalance. `heartbeat` must be lower in order to ensure processing - Recommended 1/3 of timeout. Lowering `session.timeout.ms` allows detecting and recovering from failure more quickly but more rebalances; Raising it reduces rebalance risk but lags on failure.
- `max.poll.interval.ms`: Default=5m. How long a consumer can go without polling before considered dead. Since `heartbeat` is sent on a background thread, can have a dead `poll` with a steady heartbeat. Must be long enough to be rarely reached by a healthy consumer, short enough to avoid impact on hangs.
- `default.api.timeout.ms`: Default=1m. Default timeout for API calls from the consumer when not specified in the call. Does not apply to `poll`, which always takes a timeout param.
- `request.timeout.ms`: Default=30s(rec: do not lower). If Broker doesn't respond within time, client closes connection and attempts to reconnect. Must be high enough to give Broker enough time to respond - Little gain in resending requests to an overloaded Broker + thundering herd.
- `auto.offset.reset`: Default=`latest`. Consumer start point if it has no committed offset or if offset is invalid(e.g., consumer was down long enough that the record with that offset has been aged out). `latest` => Reads from newest records(written after Consumer started); `earliest` => Reads entire partition from the beginning.
- `enable.auto.commit`: Default=`true`. `false` allows Consumers to minimizes dupes and avoid missing data. If `true`, may want to use `auto.commit.interval.ms` to control how frequently you commit. 
- `partition.assignment.strategy`: Consider C1 and C2 subscribe to T1 and T2, each with P1, P2, P3
  - `Range`: Default. Each consumer is assigned a consecutive subset of partitions. C1 receives T1P1, T1P2, T2P1, T2P2; C2 receives T1P3, T2P3.
  - `RoundRobin`: All partitions from all topics are assigned sequentially. C1 receives T1P1, T1P3, T2P2; C2 receives T1P2, T2P1, T2P3.
  - `Sticky`: On rebalance, aims to move as few partitions as possible to reduce overhead. If all Consumers in a Group are sub'd to same topic, Sticky will be as balanced as RoundRobin and subsequent assignments will have less partition movement; If Consumers in a Group are sub'd to different topics, Sticky will be more balanced than RoundRobin.
  - `Cooperative Sticky`: Sticky with Cooperative Rebalancing: consumers can continue consuming from partitions that aren't re-assigned during rebalance.
  - Custom: Can point to a custom class.
- `client.id`: Used by brokers to id client's messages for logging, metrics, quotas.
- `client.rack`:  Enables fetching from partition's closest replica(by datacenter or AZ) instead of leader replica. Works in concert with `replica.selector.class`=`RackAwareReplicaSelector`.
- `group.instance.id`: Used for static group membership.
- `receive.buffer.bytes`, `send.buffer.bytes`: TCP send / receive buffers. -1 for OS defaults. Increase if going cross-datacenter to accommodate higher latency / lower bandwidth.
- `offsets.retention.minutes`: *Broker* configuration, Default=7d. How long a Broker maintains its offset commit records for a Group after the group appears empty. Once dropped, when the group returns it will be treated as brand new with no offset records.

## Commits and Offsets

- Consumers `commit` their offset to Kafka to preserve their position by producing a message to `__consumer_offsets` topic with the committed offset for each partition. During rebalance, consumers can track previous consumer's progress on partition using this offset. 
  - If offset is smaller than the last message processed, those in between will be processed twice. If larger, messages in between will be missed.

### Automatic: Commits at default interval.
  - 5s default => Rebalance at 3s mark? Consumers will re-process the messages from 0s-3s.
  - Commit will use offset returned by last `poll` - It doesn't know what was actually processed, so _must_ process all `poll`ed data before calling again, even during exceptions / stops.

### Commit Current: Dev controls when commit happens.
  - `CommitSync`: Blocks on Broker response. Commits the latest offset returned by `poll`, throws if unrecoverable, otherwise does a blocking retry until it succeeds. Again, only call after processing all records or may miss some / re-process. 
  - `CommitAsync`: Doesn't block on Broker but doesn't retry on error - By the time `CommitAsync` receives a response, another `commit` may have occurred.
    - e.g., CommitAsync(2000) => Broker doesn't respond due to network issue => Another batch is processed and CommitAsync(3000) succeeds => Shouldn't retry `2000`, which is now in the past and would cause dupes.
    - Supports callbacks if you want to log errors or do manual retries.
    - Manual retries: Use a monotonically increasing sequence number as instance var. Increase it each time you commit, then check sequence number in the `CommitAsync` callback against the value received - If they're even, safe to retry. If higher, newer commit has already been sent.
    
### Combining Sync and Async Commits

- Occasional failures to commit generally aren't a problem, but they are just before we `Close` a consumer and before rebalancing, so use `CommitAsync` in normal `poll` loop, then `CommitSync` on shutdown.

### Commit Specified Offset

- Allows committing more frequently(e.g., middle of large batch) - Can't simply call `Commit...` because that will automatically use the offset returned by the last `poll`, which you haven't finished yet.
- `Commit...` accepts a map of Partitions and Offsets param to facilitate.

## Rebalance Listeners

- Consumer hook points for when partitions are added / removed from the consumer
  - `OnPartitionsAssigned(IEnumerable<TopicPartition>)`: After partitions have been assigned to the consumer but before it starts consuming messages. Used to prep state, seek to correct offset, etc.
    - If Cooperative: Invoked on every rebalance to notify consumers, but may pass an empty list if no partitions were assigned.
  - `OnPartitionsRevoked(IEnumerable<TopicPartition>)`: Used to commit offsets so that the next consumer to use the partition(s) picks up at the right place
    - If Eager rebalancing: Invoked before rebalancing, after consumer has stopped processing messages.
    - If Cooperative: Invoked at end of rebalance with only the subset of partitions that will be lost. Will only be invoked if the consumer gave up ownership, never with an empty collection.
  - `OnPartitionsLost(IEnumerable<TopicPartition>)`: *Only called if Cooperative rebalancing*. If not hooked, falls back to `OnPartitionsRevoked`. Exceptional case - Called when partitions were assigned to another consumer without being revoked from this one. Used to clean up state / resources used with these partitions, but the new owner may already have its own state so be careful.
  - If Cooperative: Implementing all 3 hooks guarantees that during a normal rebalance `OnPartitionsAssigned` will be called by a new owner only after previous owner has completed `OnPartitionsRevoked` and given up ownership.
  
## Consuming Records with Specific Offsets

- `SeekToBeginning(IEnumerable<TopicPartition>)` and `SeekToEnd(IEnumerable<TopicPartition>)`: To read from the beginning or end of partitions
- `Seek(TopicPartition, Offset)`: Facilitates exactly-once semantics when offsets are stored outside of Kafka in concert with `ConsumerRebalanceListener`
  - To seek back or forward a few messages(e.g., skip to catch up for non-critical messages)
  - When an offset is stored outside of Kafka(e.g., in a database). e.g., app wants to pull from Kafka, store to DB, write offset only once stored - For atomicity, useful to commit a TX to the database with both record and offset. On consumer start / rebalance, use `Seek` instead of default polling behavior to find correct offset.

## Exiting

- `Consumer.Wakeup()`: Should call in another thread when leaving `poll` loop. May be done from `ShutdownHook` if `poll` is running in the app's main thread. Causes `poll` to exit with `WakeupException` immediately if `poll` is processing, or on next iteration if not.
- `Consumer.Close()`: Called after `WakeupException` to commit offsets and notify Group Coordinator that consumer is leaving, triggering a rebalance.

## Deserializers

- Avro + SchemaRepository: Useful since producer serializers must match consumer deserializers - `AvroSerializer` can ensure schema compatibility by throwing an error if things don't match.
- Custom: Possible but not recommended - Risks tightly coupling producers / consumers, etc.

## Standalone Consumers(i.e., Consumers without Groups)

- Useful if you're sure there will only ever be one consumer that will need to read data from every partition in a topic, or from one specific partition. Groups and rebalances would be needless overhead here.
- Instead of `Subscribe`ing to a topic, consumer directly `Assign`s partitions. Can't subscribe and assign at the same time.

# Chapter 5: Kafka Internals

## Cluster Membership

- When a Broker starts, a unique id(either set or auto-gen'd) registers in Zookeeper by creating an ephemeral node. Kafka components subscribe to `/brokers/ids` to be notified when brokers add/remove. When a broker loses connectivity, node is automatically removed, subscribers notified. However, broker id stays in other places e.g., list of replicas for a topic, so that if you lose a broker and start a new one with the same id, it will join the cluster with the same partitions / topics.
  - Duplicate Ids: Trying to register with a duplicate id will error

## The Controller

- First broker started - Creates a node in Zookeeper at `/controller`. Subsequent brokers will try to create, error, create a [Zookeeper watch](https://zookeeper.apache.org/doc/r3.4.8/zookeeperProgrammers.html#ch_zkWatches) to detect changes. When `/controller` disappears, brokers will try to create that path, one will become new controller - The new controller receives an incremented _controller epoch_ number to prevent a _split brain_ scenario - messages received from an older controller can be ignored.
- When a broker leaves, controller determines partitions that need a new leader, determines who the new leader should be(the next replica in the partition's replica list), sends a request with info on new leader / followers to all brokers that contain the new leader or the existing followers for those partitions. New leaders begin serving produce / consume requests from clients, followers begin replicating messages.
- When a broker joins, broker id used to determine if there are replicas that exist on the new broker. If there are, controllers notifies new and existing brokers, replicas on the new broker begin replicating messages from existing leaders.

## Replication

- Topics have many Partitions, Partitions may have many replicas. Replicas are stored on broker - Generally a broker will store many(thousands) of replicas from different topics and partitions.
- Leader Replica: Each partition has a single leader that handles all produce / consume requests. Knows how up-to-date followers are(e.g., if follower falls behind due to network congestion or a broker crash, halting syncs on that broker's replicas).
  - Preferred Leader: The leader replica at the time when the topic was created, preferred because at creation time the leaders are evenly balanced across brokers. Default `auto.leader.rebalance.enable=true`, checks if preferred leader is not the current leader and trigger leader election to fix if so. Preferred leader can be determined by looking at the first replica in the list returned from kafka topic script(`kafka-topics.sh`). Recommended value = false?
- Follower Replica: Replicate messages from the leader and stay synced. After leader crash, one follower will become the new leader. Replicas send `Fetch` requests(same as Consumers) to Leader with offset wanted; Leader replies with messages - Fetch requests will always be in order, will never request message 4 before message 3 has been received, allowing Leader to know how far followers are behind by looking at the last offset requested by a given Follower.
  - Out of sync: Replicas that haven't requested a message for > 10s(`replica.lag.time.max.ms`) or that has requested messages but hasn't caught up to the most recent in that timeframe. Out of sync replicas can't be elected leaders b/c don't contain all messages. 
  
 ## Request Processing
 
 - Clients initiate TCP connections to brokers, brokers process those requests in order.
 - Standard header: Request type(i.e., API Key); Request version(to respond to different client version); Correlation ID; Client Id(identifies app)
 - Each listening port on the broker runs an _acceptor_ thread that creates a connection and hands it to a _processor_ thread(a.k.a, `network thread`) for handling. Number of processor threads is configurable.
  - Client request -> Acceptor -> Processor -> Request queue -> IO thread(a.k.a, request handler thread) -> Response queue -> Processor -> Client. 
- Common Requests
  - _Produce_, sent by producers with messages to be written; _Fetch_, sent by consumers / followers when they read.
    - _Produce_ and _Fetch_ requests must be sent to the broker with the leader replica of a partition, else receive _Not a Leader for Partition_ error.
  - _Metadata_, sent by clients with list of topics they want, response includes partitions in the topic, replicas for the partitions, and leader replica. Metadata requests can be served by any broker, as they all have a cache containing this info. Refreshes periodically(`metadata.max.age.ms`) to determine if topic metadata has changed(e.g., broker added, replicas moved to new broker); also if the _Not a Leader_ error occurs, since that signals stale info.
  - Produce
    - `acks` controls how many brokers need to process the message before client assumes it was written successfully.
    - On request receipt, broker confirms user has write privs on the topic; number of acks is valid(0, 1, "all"); if `acks=all`, are there enough replicas in-sync to consider the request safely written. If all's well, written to filesystem cache - No guarantee on lag time to disk, records are considered written once they hit the cache. Once leader has written, if `acks=0` or `acks=1`, broker responds immediately; else, stored in a _purgatory_ buffer until all replicas have written.
  - Fetch
    - Client asks broker to send messages from a list of topic + partition + offset with a limit on the max and min amount of data(or min with a time limit) they will accept(to reduce client memory load and broker cpu / network traffic, respectively). Broker determines whether the offset exists on the partition(i.e., is the client asking for a deleted or non-existent future message). If it does, broker reads up to client limit and sends using zero-copy - Messages are sent from the file(or filesystem cache) directly to the network channel without buffering to remove overhead of copying bytes into buffers, managing those buffers.
    - To guarantee consistency(e.g., if leader writes, crashes, new leader hasn't yet written), not all data is availbe to be read - Clients can only read messages that were written to all in-sync replicas. Attempting to fetch messages that haven't yet been written to all isrs will simply return empty, will not error. If replication is slow, new messages will take longer to appear, up to `replica.lag.time.max.ms`(which controls how long until a replica is considered out of sync).
- Other Requests
  - Protocol handles ~20 and expected to grow
  - Brokers should be upgraded before clients - Brokers will respond successfully to old clients, but old brokers may not be able to serve new client formats.
  - `LeaderAndIsr`: Controller announcing new partition leader

## Physical Storage

- Partition replica is base unit - Cannot be split across brokers or disks, limited to size of a single disk mount point.
- Stored in path specified by `log.dirs`.

### Partition Allocation

- On topic creation, Kafka distributes partitions evenly across brokers, with a replica on different brokers(and different racks if provided). Given e.g., 6 brokers, Partition 0 Leader(P0L) on Broker 4, P1L on broker 5, P2L on broker 0, etc -> Replicas placed at increasing offset from leader, P0 follower 1 on broker 5, P0 follower 2 on broker 0, etc. If rack aware, alternate across racks instead of incrementing broker. Once distributed across brokers, determine directory - Count how many partitions at each directory, add new to the one with the fewest(so if adding a new disk, all new partitions will go to that disk for some time). _Space, load, and existing partition size are not taken into account_, so if partitions vary greatly, be careful.

### File Management

- To accomodate expiring records per retention settings, partitions are split into file segments(default: 1GB or 1 week of data).
- Active segment is the segment currently being written to. Active segment will never be deleted, even if retention is shorter.
- Kafka will keep file handles open to both inactive and active segments, so lots of open handles.

### File Format

- Data format is same as message format to facilitate zero-copy.
- Message contains: Key, value, offset, message size, checksum, magic byte(message format version), compression codec(Snappy, GZip, LZ4), timestamp(given either by producer when sent or by broker on arrival, depending on cfg).
- If producer sends compressed messages, messages in a single producer batch are compressed together and sent as the value of a wrapper message, so broker receives the single wrapper message. When consumer decompresses message value, it will see all messages in the batch with their own timestamps, offsets. Thus, producer compression + large batches = better compression both over the network and on disk, but if the message format changes(e.g., we add a timestamp to the message), both the wire protocol and on-disk format must change and brokers must know how to handle files that contain both formats.
- `DumpLogSegments` tool(`bin/kafka-run-class.sh kafka.tools.DumpLogSegments --deep-iteration`) will show partition segment and content - Offset, checksum, etc.

### Indexes

- Kafka maintains an index for each partition to facilitate quickly seeking on offset requests - Maps offsets to segment files and positions within the files.
- Indexes are also segmented, deleted on message purge. Corrupt / manually deleted indexes will be regenerated from matching log segments by rereading messages and recording offsets/locations.

### Compaction

- e.g., Customer shipping addresses, where you want to retain each user's current address indefinitely, purge old; or that stores application state changes.
- In cases where you only care about latest state, topic retention policy can be set to `compact`.
- Compaction views each log as two portions
  - Clean: Messages that have been compacted in the past - Each key contains only the latest value
  - Dirty: Messages written since last compaction
- If compaction enabled when Kafka starts(via `log.cleaner.enabled` setting), each broker will start a _compaction manager_ thread and some _compaction_ threads responsible for managing and performing compaction. Compaction threads -> Read through partitions with the highest ratios of dirty records -> Get dirty section, map in-memory(16-byte hash of key + 8-byte offset from previous message with same key, so each map entry is 24 bytes) -> Read clean segments, starting at oldest, check against offset map - If key exists, we omit the message because an identical key with a newer value exists; else, the value of this message is the latest, so copy message to replacement suegment -> swap replacement segment for the original, move to next segment.
  - Memory compaction limits are set and summed across _all_ compaction threads, so 1GB will allow each compaction thread 200MB.
- Deleted Events
  - Since we always keep the latest message value, deletion must be triggered by the _application_ producing a message with the key and a `null` value. Cleaner will keep this _tombstone_ around for some amount of time, during which consumers will be able to read it and see it was deleted; then, cleaner will remove the tombstone. Important to give consumers enough time to see the tombstone, else will never see the record was deleted(e.g., if consumer needs to delete a database entry based on the tombstone).
- When are topics compacted?
  - Active segment is never compacted.
  - Old behavior: Kafka compacts when 50% of the topic contains dirty records(configurable). Coming soon / May exist: Grace period during which Kafka guarantees messages will not be compacted, allowing apps that need to see every message to do so.

# Chapter 6 - Reliable Data Delivery

## Reliability Guarantees

- Messages inside a partition will be ordered by time of receipt.
- Messages are considered committed when they have been written to all in-sync replicas, _not_ when they're flushed to disk(due to filesystem cache writes). `acks` setting determines when the client receives an ack of the write, not when it's considered committed; however, consumers can only read messages that are committed.
- Messages will not be lost as long as one replica is alive.

## Replication

- Partitions are stored on a single disk, which is replicated across multiple brokers. One replica will be the leader, which serves all requests; the others just stay in sync with that replica. If leader fails, one of the in-sync replicas will be promoted.
  - In-sync replica: Has an active heartbeat in Zookeeper; Has fetched messages in the last 10s(configurable); Has fetched the _most recent_ message from the leader in the last 10s(i.e., has low lag). A slow ISR will slow down producers(waiting for it to be written) and consumers(can't see messages until fully committed); an out of sync replica doesn't impact performance since leader no longer needs it to ack a write for commit.
  - **Warning**: Replicas flipping frequently out of sync signals something is wrong with the cluster - Often misconfiguration of Java's garbage collection on a given broker, forcing lengthy gc pauses which will kill the connection and push the broker out of sync.

## Broker Configuration

- Configured at the broker or topic level, so can have different guarantees for critical vs non-critical messages on a cluster.
- Replication Factor
  - Broker: `default.replication.factor`(default: 3); Topic: `replication.factor`.
  - At `1`, a topic will be entirely unavailable during a broker restart; `2` means able to lose a single broker and still function - but sometimes losing a broker will force a restart of another broker; etc.
- Unclean Leader Election
  - `unclean.leader.election.enable`: default=true. Broker-only cfg. Controls what happens when a leader becomes unavailable and no ISR exists which can happen when:
    - Leader has 2 followers, both go unavailable. Leader considers records committed because followers are out of sync. Leader becomes unavailable, an out of sync broker restarts before Leader -> out of sync broker will be only available replica for leader.
    - Partition has 3 replicas, but network issues force 2 followers out of sync(but still alive). Leader commits since it's the only isr. Leader drops.
  - If allow unclean leaders, we lose messages that were persisted only to leader; else, we must wait for original leader to return in order to come back online.
  - Disable if you'd rather be down than lose messages(e.g., financial transactions).
- Minimum In-Sync Replicas
  - `min.insync.replicas`: Broker or topic-level. If > 1, leader will reject Produce requests when there aren't enough isrs to satisfy min with `NotEnoughReplicasException`; Consume requests are unaffected.
  
## Using Producers in a Reliable System

- Even if brokers are cfg'd for reliability, Producers must be cfg'd also or risk data loss e.g., 
  - 3 replicas, `unclean.leader.election.enable=false` to prevent data loss; Producers send with `acks=1`; Leader responds after self-commit but crashes before replication, so other replicas are considered in-sync and promoted; Producer believes message was written, so from its perspective a message was lost even though consumers will never see it.
  - 3 replicas, `unclean.leader.election.enable=false` to prevent data loss; `acks=all`; Attempt to produce, but leader just crashed, new one isn't elected, so LeaderNotAvailableException; Producer doesn't handle error, doesn't retry; Broker performed correctly(never got the message) and consumers will never see message, but messages will be lost.
- Send Acknowledgements
  - `acks=0`: Producer considers message written as soon as it's transmitted, even if partition is offline. Even during clean leader election, Producer will lose messages because it won't deal with Leader Unavailable during election.
  - `acks=1`: Leader will ack or send an error as soon as it writes to its partition file(i.e., filesystem cache). Producer will receive LeaderNotAvailableException during election, must handle by retrying. Data loss if the leader crashes and data was written to the leader but not replicated.
  - `acks=all`: Leader waits for all isrs to write the message before replying with an error / ack. Works with `min.insync.replicas` to determine safety. Slow, but can mitigate by using async mode for the producer and sending larger batches.
- Configuring Retries
  - Producers can automatically handle _retriable errors_: `LeaderNotAvailableException`; but not _nonretriable_ e.g., `InvalidConfig`, message size, authorization.
  - Wnat to never lose a message? Let Producer retry indefinitely for retriable errors. Otherwise, need to consider what you plan to do when a producer gives up due to too many fails: Are you going to just retry some more in-process(i.e., should you just make it indefinite after all); write it somewhere else and retry later; or are you going to drop messages?
  - Auto-retries have a small risk of dupes e.g., network issue prevents broker ack from reaching producer but message was written and replicated -> Producer will retry as a temporary network issue and broker will have same message twice. **Mitigate** by applying uniqueid to messages and having consumers detect and clean dupes; or by making messages idempotent, so sending twice doesn't make a difference.
    - **Warning**: Auto-retries may also cause memory issues storing all of the messages that will need to be retried.
- Additional Error Handling
  - In addition to non-retriable errors, must handle errors that happen pre-send to the broker(e.g., serialization); and errors that happen if a producer "runs out" of retries or runs out of memory due to having to retry too many messages.
  - Options: Throw away "bad messages"; Log errors; Store messages locally for triage; Trigger a callback; etc.
  
## Using Consumers in a Reliable System

- Since messages aren't visible to consumers until fully committed, consumer's main responsibility is to keep track of what it has read. Periodic offset commits allow other consumers to pick up when one consumer goes down.
- Consumer Configuration for Reliable Processing
  - `group.id`: To receive only a subset of messages from topics.
  - `auto.offset.reset`: Behavior when no offsets were committed(e.g., first start) or when a requested offset doesn't exist. `earliest`: Start from the beginning, including reprocessing messages; `latest` start from the end, ensures no dupes.
  - `enable.auto.commit`: Allow consumers to auto-commit their offsets on a schedule or do it manually. Downsides to automatic is that you have no control over how many dupes you may need to process(e.g., when a consumer stops before committing its offsets); and if you're processing on a background thread, automatic may commit before you've actually processed the message.
  - `auto.commit.interval.ms`: If you auto-commit, this is how often it happens. More frequently adds overhead but reduces the number of dupes you'll process in a bad state.
- Explicitly Committing Offsets
  - Only commit after successfully processing
  - Commit frequency trades performance for dupes
  - Know what offsets are being committed: Don't commit the last offset _read_, commit the last offset _processed_.
  - Rebalances: Apps must commit before partitions are revoked, local state must be wiped and rebuilt based on new partitions.
  - Retries
    - Because you commit _offsets_ instead of _message ids_, even if you process message 2 before you've retried message 1, shouldn't commit unless willing to lose message 1. Instead
      - On retriable error, commit the last record processed, then store incoming records that need processing in a buffer and keep retrying the bad one. Alternatively `pause` the consumer so that future `poll`s won't return additional data.
      - Or, write retriable errors to a separate topic and move on. Have a separate consumer gruop handle retries from the retry topic; or a single consumer can subscribe to both the main and retry topics, but pause the retry topic between retries, like a dead letter queue.
  - Maintaining State: e.g., to calc a moving average after each poll, on restart you'll need to both consume from last offset and also recover the matching moving average. One option: Write the latest accumulated value to a `results` topic when you commit the offset, so when a new thread is starting up it can pick up the last accumulated value; but Kafka doesn't provide transactions, so can't gurantee offset commit + write to topic will be atomic. Hard to handle this inline - Easier to solve with Kafka Streams, which has APIs for aggregation, joins, windows, other analytics.
  - Long Processing Times: If a process blocks and fails to report heartbeat back to Kafka, Consumer will be considered offline. Must hand off to a thread-pool thread, pause Consumer(but continue polling, just receiving no data), then unpause when worker threads finish.
  - Exactly-Once Delivery
    - One option: Write results to a system that has support for unique keys - Either use an auto-gen'd unique key or create one using topic + partition + offset. If a record is reprocessed, will re-write the same key and value, so result will be the same.
    - Or, if you have a transaction-based system, write the records and their offsets in a single transaction to that system, retrieve offsets from there, use `consumer.seek` to manually start consuming from the tx-stored offset.
    
## Validating System Reliability

- Validating Configuration
  - Helps test that cfg implemented meets requirements and that system behaves as you expect in practice.
  - `org.apache.kafka.tools`: Includes `VerifiableProducer` and `VerifiableConsumer` classes that can be run at cli or embedded in automated testing.
    - Producer: Produces a seq from 1 to an arbitrary value, configured with all regular settings(acks, retries, etc). Each message will print a success or error.
    - Consumer: Consumes events, prints them out in order, and prints info re: commits and rebalances.
  - Potential scenarios
    - Leader Election: What happens if a leader dies? How long do producer / consumer take to recover?
    - Controller Election: How long to resume after controller restarts?
    - Rolling Restart: Can brokers be restarted one by one without losing messages?
    - Unclean Leader Election: What happens when we kill all replicas for a partition one by one(so each goes out of sync) and then start an out of sync broker? What needs to happen to resume operations and is this ok?
  - See also: [Kafka test suite](https://github.com/apache/kafka/tree/trunk/tests)
- Validating Applications
  - Custom error-handling, offset commits, rebalance listeners, etc.
  - Scenarios
    - Clients lose connectivity, briefly and extended.
    - Leader election triggered
    - Rolling restart of brokers / consumers / producers
- Monitoring Reliability in Production(Also in Chapter 9)
  - Kafka Java clients include JMX metrics.
  - Producer: Error rate, retry rate per record(among others) - Detect whether errors or number of retries is increasing. Also look for `WARN` lines in logs - If you see events with 0 attempts left, producer is running ot of retries.
  - Consumer: Consumer lag - How far behind the latest message is the consumer? [Burrow](https://github.com/linkedin/Burrow) can help with this.
  - E2E: How long from when a message is produced to when it's consumed? Produced records include a timestamp with when they were produced. Consumers need to record the number of events consumed(events / second) and record lag time between processed and produced.
  - Can also include a Monitoring consumer on critical topics that solely counts events consumed and produced to ensure they're even.

# Chapter 7 - Building Data Pipelines

- Two main use cases: Building a pipeline where Kafka is itself an endpoint(e.g., Kafka -> MongoDb, Kafka -> S3); or building a pipeline between two systems using Kafka as an intermediary(e.g., Twitter data -> Kafka -> ElasticSearch). *Kafka Connect* added to address these cases, effectively allowing Kafka act as a buffer.

## Considerations When Building Data Pipelines

- Timeliness: Do you need real-time delivery or hourly batches? Kafka can do either, and can also be used to apply back-pressure. Kafka itself back-pressures on Producers by delaying acks when needed.
- Reliability: Generally need at-least-once delivery at a minimum(basic), sometimes exactly-once(facilitated by using an external transactional data store and / or unique keys). *Connect* makes it easier for connectors to build e2e exactly-once with APIs for integrating with external systems when handling offsets - See open source connectors.
- High and Varying Throughput: Must be able to handle very high throughput and adapt if scale suddenly increases. Facilitated by decoupling producer / consumer and by Kafka's scale-out capabilities. *Connect* focuses on parallelizing work to further tweak throughput.
- Data Formats: Connect is data-agnostic, supports pluggable converters if you need to alter its own in-memory structures. Schema updates should be transparent, so a pipeline coming from SQL with a new column should transparently be added to the sink.
- Transformations
  - ETL(Extract-Transform-Load) -> Pipeline is repsonsible for making data modifications as the data passes through. No need to store data multiple times, but computation and storage must be done by the pipeline, and transformations within the pipeline limit what consumers are able to do(e.g., if a field is removed in the pipeline, consumer can't use it). If a missing field is added, pipeline must be rebuilt and reprocessed.
  - ELT(Extract-Load-Transform, also called high-fidelity pipelines, data-lake architecture) -> Pipeline does minimal transformation, data arrives close to how it came in. Maximum flexibility and potentially easier troubleshooting, since troubleshooting is at app layer. But increased CPU / storage resources for target system, where those may be pricey.
- Security: Encryption in transit; Authentication/Authorization; Audit log.
- Failure Handling: Gating faulty records, recovering from unparseable data, ability to fix bad records and reprocess.
- Coupling Concerns
  - Ad-hoc Pipelines: Different integrations bring different support / observability needs, and that new systems may need new custom pipelines, slowing speed to adopt new tech / innovate.
  - Metadata Loss: Pipelines that erase metadata or don't allow for schema evolution couple the producing software to the consuming. Without schema information, both producers and consumers need to know how to parse and interpret data(e.g., new field may break consumers). Evolution decouples to allow apps to modify over time.
  - Extreme Processing: Too much tailoring can reduce the flexibility of downstream systems by depriving them of data; and leads to frequent pipeline changes as those systems need additional data.

## When to use Connect vs Producers and Consumers

- Kafka clients are embedded into the app: Allow writing to or reading from Kafka, so useful when you have control of the app code.
- Connect for non-embedded connectors - Pull from an external store into Kafka, push from Kafka into a store. Many connectors already exist and only require cfg; If one doesn't exist, preferred to use the Connect API over traditional Produce / Consume because Connect provides cfg management, offset storage, parallelization, error handling, different data types, REST management APIs.

## Connect

- Provides APIs to run connector plugins, which move the data.
- Runs as a cluster of worker processes - Connector is installed on the workers -> Configured via REST API -> Connectors spin up tasks to move data in parallel. Source connector tasks just read and provide data objects to the workers; Sink connector tasks get data from the workers and write them to the target. `Convertors` support storing the data objects in json or Avro.
- Running Connect
  - Ships with Kafka, so `bin/connect-distributed.sh config/connect-distributed.properties`
  - Key configuration
    - `bootstrap.servers`: List of brokers that Connect will work with. Connectors will pipe to / from these brokers. Recommended at least 3.
    - `group.id`: All workers with the same group id are part of the same Connect cluster.
    - `key.converter`, `value.converter`: Default is JsonConverter, but also AvroConverter, etc.
      - Sometimes, specific cfg: `key.converter.schemas.enable=true` for json
    - `rest.host.name`, `rest.port`
      - Port for the configuration / management REST api.
  - Standalone Mode: All connectors and tasks run on one worker. Useful for dev, troubleshooting.
- Examples: See book: File src -> File sink, MySQL -> ElasticSearch
- Custom Connectors: Connector API is public, see docs.
- A Deeper Look at Connect
  - Connectors: Responsible for determining how many tasks will run for the connector; deciding how to split the data copying between tasks; Getting cfg for tasks and workers, passing it along.
  - Tasks: Responsible for actually getting data in and out of Kafka. Initialized by receiving context from worker. Source Context: Includes an object that lets source task store the offsets of the source records(e.g., file position; primary key id). Sink Context: Methods that allow connector to control the records it receives(e.g., for back-pressure, retrying, exactly-once id storage). After initialization, started with `properties`.
  - Workers: Container processes that execute connectors and tasks. Handle http requests that define connectors and cfg, store cfg, pass cfgs along. Worker crashes -> Other cluster workers will reassign connectors / tasks. Worker added -> Cluster workers will send it connectors / tasks. Auto-commit offsets for both source / sink. Retry when tasks throw.
  - Connectors and tasks move the data; Workers handle the REST API, cfg, reliability, HA, scaling, load balancing. The worker complexity isn't handled by the Producer / Consumer and may take substantial time under Producer / Consumer. Connect handles this for you.
  - Data Model
    - Connect's Data API includes data objects + schema to describe data.
    - Source connectors know how to generate objects based on the Data API -> Connect workers use `convertors` to store to Kafka; Connect workers then read from Kafka using that converter and then pass it to the sink connector.
  - Offset Management
    - Source connectors feed workers with a logical partition and offset(NOT Kafka partition / offset - these are in terms of the source system e.g., file is the partition, offset is the line number). Connectors return these to the worker, workers send to brokers, workers store offset once ack'd.
    - Sink connectors read from a stream that already has a topic, partition, and offsets, call connector's `put` method to store them to destination. If success, commit offset.

## Alternatives to Connect

- Ingest frameworks for other datastors: Hadoop + Flume; Logstash + ElasticSerach. If Kafka is the center of your system, use Connect; if something else, use theirs.
- GUI-based ETL: Some GUI-based tools support Kafka as source and sink. If you're already using them or team is used to a GUI-based approach to building ETL pipelines, no need to add Connect. However, these tend to be complex, suited toward heavy pipelines.
- Stream-Processing Framewoks: Most stream-processing frameworks consume from Kafka and can write elsewhere. If you're already using a framework, keep using it. May be more difficult to troubleshoot lost / corrupt messages.

# Chapter 8 - Cross-Cluster Data Mirroring

# Chapter 9 - Administering Kafka

# Chapter 10 - Monitoring Kafka

##  Metric Basics

- Exposed via JMX
- Internal vs External Measurements: Internal from JMX out of Kafka itself; External e.g., wrapping total endpoint latency for the web app consuming from Kafka.
- Application Health Checks: Preferred -> Use an external process that reports whether the broker is up or down; Else, alert on suspicious lack of metrics from the Kafka broker(i.e., stale metrics). Stale metrics method makes it hard to differentiate between failure of metric system and Kafka.
- Metric Coverage: Mind how many alerts you create, avoid alert fatigue.
- Rate Metric Attributes
  - EventType: Unit of measurement(e.g., bytes)
  - RateUnit: Time period for rate(e.g., seconds)
  - OneMinuteRate / FiveMinuteRate / FifteenMinuteRate: Avg over 1/5/15m.
  - MeanRate: Avg since broker started. Should not fluctuate much.
  - Count: Increasing value for the metric since broker started(e.g., bytes producted to the broker since process started).

## Broker Metrics

- What if Kafka is what you use to collect metrics? Could use a separate monitoring system for Kafka that doesn't depend on Kafka itself, or store metrics for Cluster A in the region for Cluster B and for Cluster B in the region for Cluster A.
- Under-replicated Partitions
  - Reported per-broker
  - The number of partitions for which the Broker is the leader replica and the follower replicas aren't caught up
  - Catches brokers down, resource exhaustion, etc.
  - How to React
    - If steady: One or more brokers is probably offline. Determine which and resolve, by restarting / replacing.
    - Ensure you've recently done a preferred leader election - Brokers don't automatically take leadership back unless `auto.leader.rebalance.enable=true`(not recommended), so easy for leader replicas to become unbalanced. Preferred leader election is safe and may fix things.
    - If fluctuating, or if steady but no brokers are offline: Probably a performance issue.
      - Determine whether a problem is isolated to a single broker or whole cluster e.g., by checking whether the under-replicated partitions are on a single broker. 
      - If not isolated, may still be a single broker having trouble replicating messages. Double-check whether a specific broker underlies all under-replicated partitions(e.g., using `kafka-topics.sh`). 
      - If not a single broker, cluster-level
        - Unbalanced Load / Leadership
          - Check Partition Count + Leader Partition Count + All topics message IN rate + All topics bytes IN rate + All topics bytes OUT rate. These should be even in a balanced cluster, especially after a preferred leader election. If _uneven_, may have to reassign partitions from loaded brokers to less loaded(e.g., via `kafka-reassing-partitions.sh`).
            - Kafka doesn't provide for automatic reassignment of partitions, so balancing can be a tedious manual process of reviewing all data and finding a good assignment. [`kafka-assigner` tool](https://github.com/linkedin/kafka-tools) can help automate this.
          - Check CPU + Inbound network throughput + outbound network throughput + Disk avg wait time + Disk % utilization. All of these will result in under-replicated partitions since the replication process acts the same as other kafka clients. Review trend regularly to catch underages early. At the broker level, check All Topics Bytes in Rate.
        - Host-level Problems - Not only crashing, but bad memory or cpu(causing the system to not use that resource, reducing available overall).
          - Disk failures: Producer performance is tied directly to disk performance due to commits. Slow commits -> Performance issues for Producers, Replica fetchers(thus, under-replicated partitions). Single disk failure can break a cluster b/c partitions will be evenly spread over a cluster -> One broker performs poorly, slows down Produce(which connects to all brokers that lead a partition for a topic) -> back-pressure in Producers.
          - Network failures: Bad cable, connectors, configuration(speed, duplex) at either server or in hardware; OS issues(buffers undersized, not enough memory for more connections). Check for # of errors detected on the network interfaces - If increasing, may be an underlying issue.
          - Contention: Another app competing for resources(perhaps misbehaving).
          - Configuration: If nothing else amiss, look for any misconfiguration of brokers or system.
- Other metrics
  - Active Controller Count(0 or 1): Shows whether the broker is currently the controller for the cluster(1). If two brokers report 1, a controller thread has hung -> Problems with admin tasks e.g., Partition moves. Must restart both brokers. If _no_ broker reports 1, cluster won't respond to state changes(e.g., topic / partition creation, broker failures). Must determine why controller threads aren't working - Has a network partition from the Zookeeper cluster occurred? After fixing, must restart all brokers to reset state for controller threads.
  - Request Handler Idle Ratio: % time that request handlers are idle. Warn < 20%, Alert < 10%. May be undersized cluster; insufficient threads in the pool(should = # CPUs); threads doing too much work per request(prior to Kafka 0.10, threads were responsible for decompressing message batches, validating and assigning offsets, recompressing -> high load).
    - Two thread pools - Network threads(read and write data to clients across the network, low processing overhead), IO Threads(i.e., request handler threads; responsible for servicing client requests, including read/write messages to disk, so load will impact thread pool).
    - Intelligent thread usage: Shouldn't need more threads than CPUs in the broker, as Kafka offloads long requests to a purgatory(e.g., when requests are awaiting acks on a Produce).
  - All Topics Bytes In: How much traffic brokers are receiving from producing clients. Predicts the need to expand cluster / bump resources. Shows whether one broker is taking more traffic than others, requiring rebalancing of partitions.
  - All Topics Bytes Out: How much consumers are reading. Generally higher than Produce(e.g, 6x higher). *Includes* replica fetcher traffic, so bytes out = (bytes in * replication factor - 1)(b/c each replication will consume all bytes in) + consumer bytes.
  - All Topics Messages In: Different view of Producer traffic. avg message size = All Topics Messages In / All Topics Bytes in.
    - No matching "Out" metric because the broker just sends the next batch to the consumer without determining how many messages are inside, so broker doesn't know outgoing count.
  - Partition Count: Shouldn't change much. Includes every replica the broker has, whether leader or follower. Useful when automatic topic creation is enabled.
  - Leader Count: Should be even across cluster. Should be checked regularly to determine whether cluster is imbalanced, even if replicas are balanced in count and size, because a broker can drop leadership for many reasons(e.g., zookeeper session expiry) and won't take leadership back unless you have automatic enabled. Leader count drops -> May have to do Preferred replica election.
  - Offline Partitions: Only provided by the broker acting as a controller(other brokers report 0). Shows # partitions in cluster without a leader due to either all brokers hosting a replica for the partition being down; or no in-sync replicas exist that can take leaderhip due to message count mismatches(when unclean election is disabled). Offline partitions can result in losing messages or causing back-pressure at the app level, so must address ASAP.
  - Request Metrics: ApiVersions, ControlledShutdown, CreateTopics, DeleteTopics, DescribeGroups, Fetch, FetchConsumer, FetchFollower, GroupCoordinator, Heartbeat, JoinGroup, LeaderAndIsr, LeaveGroup, ListGroups, Metadata, OffsetCommit, OffsetFetch, Offsets, Produce, SaslHandshake, StopReplica, SyncGroup, UpdateMetadata
    - Each records:
      - Requests per Second(Rate): Count of requests by type that have been received, processed
      - Total Time: From broker receipt to response
      - Request queue time: Time spent in the queue after receipt before response
      - Local Time: Time spent by partition leader before sending to disk buffer(not to disk)
      - Remote Time: Time spent waiting for followers before request can complete
      - Throttle Time: Time response is held to satisfy client quota settings
      - Response Queue Time: Time spent in the queue before sent to requestor
      - Response Send Time: Time spent actually sending the response
    - Watch: Avg, p95/99 Total Time for all request types(pref with drills to more granular measurements)
    - Alert: Some requests(e.g., `Fetch`) are dependent on client settings(e.g., how long client will wait), how busy a topic is, etc. May be able to develop a baseline for p9x and go from there or check anonalies.
- Topic and Partition Metrics
  - Per-Topic: Bytes In rate; Bytes Out rate; Failed Fetch rate; Failed Produce rate; Messages In rate; Fetch Request rate; Produce Request rate. These are difficult to monitor in aggregate, but may be useful passed to client users to evaluate their usage.
  - Per-Partition
    - Partition Size: Bytes retained on disk for a partition(sum to get total retained for a topic). A difference in partition size for a single topic may indicate that messages aren't evenly distributed across the key used for produce.
    - Log Segment Count: Number of log segment files on disk.
    - Log End Offset, Log Start Offset: Highest and lowest message offset in the partition. This _is not_ number of messages when compaction is on - in-between messages may be deleted when updated records are added. Potentially allows you to get an offset at a specific timestamp for rollback/replay/etc.
    - Under-Replicated Partition: Difficult to use at the partition level due to sheer amount of data - Easier to use broker level metrics + CLI tools to find under-replicated partitions.
- JVM Monitoring
  - Garbage Collection: Varies by JRE, but e.g., Full GC Cyles(G1 Old), Young GC Cycles(G1 Young).
    - CollectionCount: Counter. Number of GC cyles since JVM started
    - CollectionTime: Counter. Time(ms) spent in GC cycle since JVM started
      - Both include LastGCInfo: Info on the last GC cycle of either type. Duration(ms); GcThreadCount; id; startTime; endTime;
  - Java OS monitoring
    - MaxFileDescriptorCount: Max number file descriptors JVM may open
    - OpenFileDescriptorCount: Number of file descriptors currently open
    - File descriptors must be opened for every log segment and network connection - Issue closing network connections may cause quick resource exhaustion.
- OS Monitoring
  - Useful to watch cpu time spent by type: In user space; in kernel space; on low-priority processes; idle; waiting on disk; handling hardware interrupts; handling software interrupts; waiting on hypervisor.
  - System Load: Count of the number of processes runnable and waiting for a cpu to execute on - Linux also includes threads in uninterrable sleep state(e.g., waiting for disk). Gives avg over last 1m, 5m, 15m. Single CPU, value 1 => System fully loaded, always a thread waiting to execute. 24 CPUs, value 24 => System fully loaded.
  - Brokers will be CPU-heavy, but less memory hungry(e.g., for compression). Ensure that swap isn't being used.
  - Disk space, inodes(file and dir metadata objects): Ensure not running out of space.
  - Disk IO: Reads/Writes per sec, avg r/w queue size, avg wait time, % utilized
  - Network In / Out: Remember that outbound traffic is inbound bytes * replication factor.
- Logging
  - Useful to separate out these two:
    - `kafka.controller` logger: Messages regarding cluster controller. Since only one broker will be controller at a time, only one broker will be writing these logs. Includes topic creation / modification, broker status changes, cluster activities(e.g., preferred replica election, partition move)
    - `kafka.server.ClientQuotaManager`: Messages about produce/consume quota activities.
  - `kafka.log.LogCleaner`, `kafka.log.Cleaner`, `kafka.log.LogCleanerManager`: Enable, set to DEBUG to get status of compaction threads: Size and number of messages in each partition being compacted.. **Without these, compaction failure of a partition may silently halt log compaction threads**.
  - For debugging issues
    - `kafka.request.logger`: Records info on every request sent to the broker. DEBUG -> Connection endpoints, request timings, summary information; TRACE -> Topic and partition info. Both of these are very verbose at this level.

## Client Monitoring

- Producer Metrics
  - Overall Producer Metrics
    - Alert on
      - `record-error-rate`: >0 => Producer is dropping messages it's trying to send
      - `request-latency-avg`: Time to produce a request to a broker. Establish a baseline and alert above that. Increase in latency may be neworking issues or broker issues.
    - Monitor
      - `record-retry-rate`: Counterpart of `-error-rate`. Some retries are expected.
      - `outgoing-byte-rate`(bytes sent / s); `record-send-rate`(messages produced / s); `request-rate`(Produce requests sent to brokers / s). Remember a single request wil contain one or more batches, each of which will contain one or more messages.
      - `request-size-avg`(avg size in bytes of produce requests); `batch-size-avg`(avg size of a single batch); `record-size-avg`(avg size of a single record); `records-per-request-avg`(avg number of records in a single produce request). Useful if producing to a single topic; less if producing to many.
      - `record-queue-time-avg`: Avg time in ms a single message waits in the producer before being produced. After an app calls producer client using `Send`, producer will wait til it can either fill a batch(based on `batch.size`) or `linger.ms` expires. Fast topics will hit the first, slow the second. Can tune those settings to latency needs based on this metric.
    - `ProducerRequestMetrics`: Less useful because it's recorded per-thread, so the other metrics are easier to use.
    - All Overall metrics are also broken down per-broker and per-topic - Useful for debugging, less so for general watching
      - Broker
        - `request-latency-avg` may be useful because it should be stable as long as batches are stable and may show a problem with connections to a specific broker.
        - Other metrics vary based on what partitions each broker is leading, so measurement baselines can change depending on cluster state.
      - Topic
        - Only useful for producers working with >1 topic, but difficult to use / alert on if producer is working with _many_ topics.
        - `record-send-rate`, `record-error-rate`: May isolate droped messages to a specific topic
- Consumer Metrics
  - Overall: Overall has data on low-level network ops, but Fetch Manager has data on bytes, requests, record rates.
  - Fetch Manager Metrics
    - Maybe alert on
      - `fetch-latency-avg`: How long `fetch` requests take. Impacted by `fetch.min.bytes`, `fetch.max.wait.ms`. Slow topics will also have erratic latencies as sometimes the broker will respond quickly(messages are available) and sometimes not until `fetch.max.wait.ms`(nothing to consume). Useful when consuming is fairly regular.
    - Monitor
      - `records-lag-max`: How far behind the broker we are for the partition that is most behind. Hard to alert on this b/c only shows lag for one partition and relies on consumer operating correctly. Prefer alerting on external lag monitoring(described elsewhere).
      - `bytes-consumed-rate`, `records-consumed-rate`: Message traffic consumed by client. May alert on a lower bound to determine if one consumer isn't pulling its weight, but dependent on whether the producer is working properly so may be better to monitor there.
      - `fetch-rate`(fetch requests / s); `fetch-size-avg`(bytes / request); `records-per-request-avg`(number of messages / request). There is no `record-size-avg` - If needed, have to calculate from other metrics or grab it in app code.
  - Broker
    - `request-latency-avg`: Useful if traffic is steady
    - `incoming-byte-rate`, `request-rate`: Useful to isolate problems consuming from a specific broker
  - Topic
    - Useful only if more than one topic is being consumed; else, duplicate of Fetch Manager metrics. If _many_ topics consumed, may be tough to parse due to noise.
    - `bytes-consumed-raet`, `records-consumed-rate`, `fetch-size-avg`
  - Consumer Coordinator Metrics(group only)
    - `sync-time-avg`, `sync-rate`: Time paused while / Count of times a consumer group is synchronizing(Negotiating which partitions will go to which clients). Can be lengthy with many partitions.
    - `commit-latency-avg`: Time taken to by offset commit requests. May be worth establishing baseline and alerting above that.
    - `assigned-partitions`: Number of partitions assigned to the consumer. May surface load imbalance due to coordinator algorithm for partition distribution.
  - Quotas
    - Not enabled by default, but throttling can be cfg'd(bytes/s) at producer and consumer, plus default value.
    - Monitor `fetch-throttle-time-avg`, `produce-throttle-time-avg` to surface when throttling takes place.

## Lag Monitoring

- Prefer using an external process that watches state of partition on the broker + offset of most recently produced message + state of consumer + last offset committed for the partition. This will work even if consumer is down. Still may be huge for a large consumer of many partitions.
- Consumer Lag: Difference in number of messages between last message produced in a partition and last message processed by consumer. Less useful than external monitoring because at the consumer level, this represents only a single partition(the one furthest behind) and requires the consumer to be up and operating(calc'd on each fetch request).
- Via CLI tools: Must understand what represents reasonable lag for each partition(e.g., a slow topic will need different thresholds than a busy one).
- Consider [Burrow](https://github.com/linkedin/Burrow): Gathers lag info for all consumer groups and calculates a single status for each - Working, behind, stalled, stopped. Monitors progress that the consumer group is making on processing messages(though absolute message lag is available).
- `records-lag-max`: Last resort, client-level view of lag.

## End-to-End Monitoring

- i.e., "Can I produce", "Can I consume". Ideally per-topic, but injecting synthetic traffic to each one may not be great in prod.
- [Kafka Monitor](https://github.com/linkedin/kafka-monitor): Continually produces / consumes from a topic that's spread across all brokers in a cluster. Measures availabilty of produce / consume requests on each broker and total produce-to-consume latency.

# Chapter 11 - Stream Processing

## What is Stream Processsing?

- Data Stream(i.e. Event Stream, Streaming Data): Abstraction representing an unbounded(infinite) dataset, with new records continually arriving over time.
- Characteristics
  - Ordering: Event streams are ordered by time
  - Immutable data records: Events that have happened are fact, can't be modified. e.g., Cancelling a financial transaction would incur a new event recording the cancellation of that tx.
  - Replayable: Allows for error correction, new analysis methods, audits.
- Stream Processing: The ongoing processing of the events in a stream. Compare to...
  - Request-Response: Low latency aiming for high consistency in response times. Usually blocking - App sends request, processing system sends response. 
  - Batch Processing: High latency, high throughput. Processing system wakes up on a schedule, reads all input, write output, stops. Efficient, but may take too long to  make data available.
  - Stream Processing: Continuoue, non-blocking(async).
  
## Stream Processing Concepts

- Time
  - Complex - See [_There Is No Now_](https://queue.acm.org/detail.cfm?id=2745385)
  - Event time: Time the events occurred and record was created. Create time is automatically added to producer records.
  - Log append time: Time the events arrived and were stored at the Kafka broker. Brokers will automatically add if cfg'd to do so.
  - Processing time: Time at which stream processing app received the event for processing.
- State
  - Required to operate on data across multiple events(e.g., moving average).
  - Local / Internal state: Accessible only by one instance of the stream processing app. Usually managed with an embedded, in-memory database - Very fast but dependent on local memory available, so may need to come up with clever partitioning to create substreams of data that can be parceled out.
  - External state: Maintained in an external datastore(e.g., Cassandra, Redis). Not limited as much by space, accessible from many application instances, but extra latency and complexity to manage and interact with the external system.
- Stream-Table Duality
  - Tables: A collection of mutable records with a key and a schema-defined set of attributes, generally without a concept of time(e.g., `UPDATE` will overwrite records).
  - Streams: Number of events, each of which represents a change.
  - Table -> Stream: Capture changes in the table system, ship to stream(e.g., via CDC or Kafka Connect).
  - Stream -> Table(i.e., Materializing a stream): Create a table and replay all events, changing table state as we go.
- Time Windows
  - e.g., Operating on time slices of data(moving avg, orders/week, stream-join operations). Think about...
    - Size of the window: Do we want avg every 5m, 15m, daily, etc. Larger windows are smoother but take longer to be registered.
    - How often window moves(i.e., _Advance Interval_): 5m averages can update per minute, per second, etc.
      - _Hopping Windows_: Windows for which the size is a fixed time interval
      - _Tumbling Windows_: Windows for which the advance interval = window size. "5 minute window every 5 minutes" -> no overlap in windows.
      - _Sliding Windows_: Windows for which the window moves on every record. "5 minute window every 1 minute" -> Windows overlap, so events are in multiple windows.
    - How long the window remains updatable: If new events arrive after the window is closed, do you update the past window or drop the data?
  - Windows may be aligned to clock time(00:00-00:05) or unaligned(App start@03:17, first slice 03:17-03:22). Sliding windows are never aligned because they move on new records.
  
## Stream Processing Design Patterns

- Stateless, Single-Event Processing(i.e., map/filter)
  - Consume events -> Modify -> Produce to another stream(e.g., Log source topic -> App detects ERROR vs other -> Write to ERROR topic, other topic)
- Processing with Local State
  - Rolling state, i.e., `group by`
  - e.g., Minimum and rolling avg for a symbol: _Trades_ topic, partitioned by symbol(A-N, O-Z) -> Two processors, one for each partition, each with its own state that calculates a cumulative value -> _AggregatedTrades_ topic partitioned by symbol(A-N, O-Z)
  - Complexity(Some may be handled be the app framework chosen)
    - Memory Usage: Local state has to fit in local memory
    - Persistence: On app shutdown, must preserve rolling state or be able to recover it. Can be handled with Kafka Streams, where local state is stored in-memory in RocksDB(which flushes to disk) _and_ sent to log-compacted Kafka topic(compacted because rolling state is truth and can be re-calc'd, no need to store history). On node down, re-pop from topic.
    - Rebalancing: Partition reassignments -> Old app instance has to store current state, transfer to new instance    
- Multiphase Processing / Repartitioning
  - For when rolling state won't cut it - Need all info. Similar to map/reduce, where multiple steps are often required. Some app frameworks will allow including all steps in a single app, handing work to app instances or workers at each step.
  - e.g., top 10 stock movers: _Trades_ topic, partitioned by symbol(A-N, O-Z) -> Two processors, one per partition, that calculate a daily gain / loss for each symbol -> Write to much smaller _Daily Gain / Loss_ topic with one partition(because we need all data to find tops) -> Processor to find tops -> _Top 10_ topic
- Processing with External Lookup: Stream-Table Join
  - Naive: Topic -> Enrich processor that reads from external store -> Enriched topic. Latency of external store makes this hard to scale.
  - Better: Topic -> Enrich processor that caches data from external store -> Enrich topic. Have to deal with cache invalidation frequency(too short = hammer store, too long = stale data)
  - Much Better: Listen for changes in external store to update app cache(CDC, e.g., thru Kafka Connect). _Base_ topic(Kafka) + _Enrich_ topic(fed via CDC) -> Processor joining _Base_ with local cache data from _Enrich_ -> _Enriched_ topic
    - So, _stream-table_ where one stream represents changeset from external store
- Streaming Join(i.e., Windowed Join)
  - Joining two streams' entire history, matching by key over a time window
  - e.g., Matching user search queries with search click events(where click will happen shortly after search renders, so window must be a few seconds): _Clicks_ stream + _Searches_ stream -> 5s window on each, held in local state, joined by app -> _SearchClicked_ event
    - For _Kafka Streams_: Partition each stream on the same key(e.g., UserId in this example) so that the id will land in the same partition number for each stream. Streams will auto-assign matching partitions to the same task. Task maintains rolling window in RocksDB cache for join.
- Out-of-Sequence Events
  - e.g., one source drops network connection for some time, returns later to send data
  - Complexity
    - Must recognize an event is out of sequence, so app needs to know where / when it is
    - Decide on how long to accept events - Can you drop hour-old events, day-old, etc.
    - For events accepted, handle it. In a batch job, can just re-run; in streams, must be continuous.
    - Updating results is easy if stream is going to a db, but harder if you've already sent an email.
  - _Google Dataflow_, _Kafka Streams_, and other frameworks have support for event times independent of processing time, ability to handle event times older / newer than current processing time. Devs configure how long windows must be open -> Framework maintains multiple aggregation windows in local state that are within updatable time(Memory heavy). Kafka Streams writes aggregation results to result topics which are usually compacted, so updating simply writes the new result to the stream, old gets compacted.
- Reprocessing for Migration to a New Stream
  - e.g., new app runs on existing stream, produces new stream alongside existing, eventually clients migrate to new stream
  - Create new consumer group for new app that starts from the first offset of input topics -> allow it to catch up -> deprecate old
  - Easy to switch between multiple versions for comparing results, no risk of data loss or errors due to cleanup
- Reprocessing for an Existing Stream
  - e.g., existing app is reset to process from first offset of input topics. Must reset local state(e.g., using Kafka Streams tool), may need to clean previous output stream.
  - Prefer new stream when possible to reduce complexity and for safety
  
## Streams by Example using Kafka Streams

- Kafka Streams
  - Two APIs: Low-level Processor API, allowing custom transformations; High-level Streams DSL, defining a chain of transformations(e.g., filter or stream-join).
  - For DSL API: `StreamBuilder` to create a topology/DAG of transformations to apply -> `KafkaStreams` object to start threads that apply the topology to events -> Close `KafkaStreams` object to stop processing.
  - Basic Configuration
    - Application Id: Coordinates instances of the app, names local stores and topics related to the instances. Must be unique for each Streams app within the same cluster.
    - Bootstrap Servers: Streams always reads / writes from and to Kafka and use it for coordination, so must spec cluster.
    - Serializer / Deserializer
  - Kafka Streams app in local mode only require local Kafka, even if you have multiple partitions and multiple instances - KStreams will coordinate the work as you spin up additional instances.
- [Word Count example](https://github.com/gwenshap/kafka-streams-wordcount)
- [Stock Market Statistics example](https://github.com/gwenshap/kafka-streams-stockstats)
  - _Trades_ stream with symbol, ask price, ask size -> App -> 3 streams, 5s windows / 1s update freq, for Minimum Ask Price, Num Trades, Avg Ask Price
  - Uses a custom serializer since value isn't just a `string`
  - KStreams takes care of local state, failover, etc.
- [Click Stream Enrichment example](https://github.com/gwenshap/kafka-clickstream-enrich/)
  - _Clicks_ stream + _CDC_ stream for external store + _Searches_ stream -> App
  
## Kafka Streams: Architecture Overview

- _Topology_(a.k.a., DAG): The set of operations / transformations for each event, from Sources to Sinks
- For [Word Count example](https://github.com/gwenshap/kafka-streams-wordcount): _Lines_ topic -> Split to words -> Key = Word -> Filter `the` -> Group by key -> Re-partitioned topic -> Aggregate counts using local state -> _WordCount_ topic
- Scaling the Topology
  - Many instances with many threads(configurable), load balanced across the instances. Scale out with threads / instances.
  - KStreams engine splits topology into _Tasks_, dependent on number of partitions in processed topics -> Each task consumes from a subset of partitions, executes all steps that apply to the partition, writes to sink. e.g., _Topic_ with 2 partitions -> Task 1 + Task 2 -> Output topic.
    - If many threads / instances available, each one will execute some subset of the tasks created e.g. _Topic_ with 4 partitions -> Host 1 Thread 1 with 2 tasks consuming from 2 separate partitions + Host 2 Thread 2 with 2 tasks consuming from other 2 separate partitions
  - If reading from multiple partitions, KStreams assigns all partitions needed for a single join to the same task to avoid inter-task dependencies/waits, facilitated by all participating topics having same number of partitions based on the key used for joining.
  - If repartitioning on a new join key to slice a different way, KStreams creates two sub-topologies: One to write events to a new topic with new keys and partitions, another to read from that new topic and continue processing. Facilitates parallelism for each sub-topology and keeps them independent.
- Surviving Failures
  - Kafka is HA, app can always look up its last committed offset in the stream and continue processing from that, local state can be recreated its change log(also stored in Kafka)

## Choosing a Stream-Processing Framework

- Use Cases
  - Ingestion: Migrating data from one system to another with some transformations. Can use streams, but may be better relying on a purpose-built system like Connect.
  - Low ms latency app: Apps requiring very fast response(e.g., fraud detection). Can use streams, but may be better off with request-response; if streams, must opt for a system that supports event-by-event low-latency processing, not just microbatches.
  - Async microservices: Apps performing a simple action as part of a larger process, may require local state caching. Need a stream system that integrates well with message bus(e.g., Kafka), has change capture support to push changes to other microservice local caches, and supports a local store that can serve as a cache or materialized view of microservice data.
  - Near real-time analytics: Apps performing aggregations/joins to generate insights. Need a stream system that supports a local store for advanced aggregations, windows, and joins. APIs should include support for custom aggregations, window operations, multiple join types.
- Operability: Can you deploy, monitor, troubleshoot, secure it?
- API Usability and Debugging: Can you quickly write an app with it?
- Simplicity: Does the framework handle scale, recovery, advanced window aggregations, local caches in a clean, simple way that requires little from the dev?
- Community: Is it well-supported? Are bugs fixed quickly? Is there abundant info available via searches?
