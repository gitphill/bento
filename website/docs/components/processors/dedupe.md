---
title: dedupe
slug: dedupe
type: processor
status: stable
categories: ["Utility"]
---

<!--
     THIS FILE IS AUTOGENERATED!

     To make changes please edit the corresponding source file under internal/impl/<provider>.
-->

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

Deduplicates messages by storing a key value in a cache using the `add` operator. If the key already exists within the cache it is dropped.


<Tabs defaultValue="common" values={[
  { label: 'Common', value: 'common', },
  { label: 'Advanced', value: 'advanced', },
]}>

<TabItem value="common">

```yml
# Common config fields, showing default values
label: ""
dedupe:
  cache: "" # No default (required)
  key: ${! metadata("kafka_key") } # No default (required)
  drop_on_err: true
```

</TabItem>
<TabItem value="advanced">

```yml
# All config fields, showing default values
label: ""
dedupe:
  cache: "" # No default (required)
  key: ${! metadata("kafka_key") } # No default (required)
  drop_on_err: true
  strategy: FIFO
```

</TabItem>
</Tabs>

Caches must be configured as resources, for more information check out the [cache documentation here](/docs/components/caches/about).

When using this processor with an output target that might fail you should always wrap the output within an indefinite [`retry`](/docs/components/outputs/retry) block. This ensures that during outages your messages aren't reprocessed after failures, which would result in messages being dropped.

## Batch Deduplication

This processor enacts on individual messages only, in order to perform a deduplication on behalf of a batch (or window) of messages instead use the [`cache` processor](/docs/components/processors/cache#examples).

## Delivery Guarantees

Performing deduplication on a stream using a distributed cache voids any at-least-once guarantees that it previously had. This is because the cache will preserve message signatures even if the message fails to leave the Bento pipeline, which would cause message loss in the event of an outage at the output sink followed by a restart of the Bento instance (or a server crash, etc).

This problem can be mitigated by using an in-memory cache and distributing messages to horizontally scaled Bento pipelines partitioned by the deduplication key. However, in situations where at-least-once delivery guarantees are important it is worth avoiding deduplication in favour of implementing idempotent behaviour at the edge of your stream pipelines.

## Fields

### `cache`

The [`cache` resource](/docs/components/caches/about) to target with this processor.


Type: `string`  

### `key`

An interpolated string yielding the key to deduplicate by for each message.
This field supports [interpolation functions](/docs/configuration/interpolation#bloblang-queries).


Type: `string`  

```yml
# Examples

key: ${! metadata("kafka_key") }

key: ${! content().hash("xxhash64") }
```

### `drop_on_err`

Whether messages should be dropped when the cache returns a general error such as a network issue.


Type: `bool`  
Default: `true`  

### `strategy`

Controls how to handle duplicate values.


Type: `string`  
Default: `"FIFO"`  
Requires version 1.4.0 or newer  

| Option | Summary |
|---|---|
| `FIFO` | Keeps the first value seen for each key. |
| `LIFO` | Keeps the last value seen for each key. |


## Examples

<Tabs defaultValue="Deduplicate based on Kafka key" values={[
{ label: 'Deduplicate based on Kafka key', value: 'Deduplicate based on Kafka key', },
]}>

<TabItem value="Deduplicate based on Kafka key">

The following configuration demonstrates a pipeline that deduplicates messages based on the Kafka key.

```yaml
pipeline:
  processors:
    - dedupe:
        cache: keycache
        key: ${! metadata("kafka_key") }

cache_resources:
  - label: keycache
    memory:
      default_ttl: 60s
```

</TabItem>
</Tabs>


