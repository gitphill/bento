---
title: kafka_franz
slug: kafka_franz
type: output
status: beta
categories: ["Services"]
---

<!--
     THIS FILE IS AUTOGENERATED!

     To make changes please edit the corresponding source file under internal/impl/<provider>.
-->

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

:::caution BETA
This component is mostly stable but breaking changes could still be made outside of major version releases if a fundamental problem with the component is found.
:::
A Kafka output using the [Franz Kafka client library](https://github.com/twmb/franz-go).

Introduced in version 1.0.0.


<Tabs defaultValue="common" values={[
  { label: 'Common', value: 'common', },
  { label: 'Advanced', value: 'advanced', },
]}>

<TabItem value="common">

```yml
# Common config fields, showing default values
output:
  label: ""
  kafka_franz:
    seed_brokers: [] # No default (required)
    topic: "" # No default (required)
    key: "" # No default (optional)
    partition: ${! metadata("partition") } # No default (optional)
    metadata:
      include_prefixes: []
      include_patterns: []
    max_in_flight: 10
    batching:
      count: 0
      byte_size: 0
      period: ""
      jitter: 0
      check: ""
```

</TabItem>
<TabItem value="advanced">

```yml
# All config fields, showing default values
output:
  label: ""
  kafka_franz:
    seed_brokers: [] # No default (required)
    topic: "" # No default (required)
    key: "" # No default (optional)
    partitioner: "" # No default (optional)
    uniform_bytes_options:
      bytes: 1MB
      adaptive: false
      keys: false
    partition: ${! metadata("partition") } # No default (optional)
    client_id: bento
    rack_id: ""
    idempotent_write: true
    metadata:
      include_prefixes: []
      include_patterns: []
    max_in_flight: 10
    timeout: 10s
    batching:
      count: 0
      byte_size: 0
      period: ""
      jitter: 0
      check: ""
      processors: [] # No default (optional)
    max_message_bytes: 1MB
    max_buffered_records: 10000
    metadata_max_age: 5m
    compression: "" # No default (optional)
    tls:
      enabled: false
      skip_cert_verify: false
      enable_renegotiation: false
      root_cas: ""
      root_cas_file: ""
      client_certs: []
    sasl: [] # No default (optional)
```

</TabItem>
</Tabs>

Writes a batch of messages to Kafka brokers and waits for acknowledgement before propagating it back to the input.

This output often out-performs the traditional `kafka` output as well as providing more useful logs and error messages.


## Fields

### `seed_brokers`

A list of broker addresses to connect to in order to establish connections. If an item of the list contains commas it will be expanded into multiple addresses.


Type: `array`  

```yml
# Examples

seed_brokers:
  - localhost:9092

seed_brokers:
  - foo:9092
  - bar:9092

seed_brokers:
  - foo:9092,bar:9092
```

### `topic`

A topic to write messages to.
This field supports [interpolation functions](/docs/configuration/interpolation#bloblang-queries).


Type: `string`  

### `key`

An optional key to populate for each message.
This field supports [interpolation functions](/docs/configuration/interpolation#bloblang-queries).


Type: `string`  

### `partitioner`

Override the default murmur2 hashing partitioner.


Type: `string`  

| Option | Summary |
|---|---|
| `least_backup` | Chooses the least backed up partition (the partition with the fewest amount of buffered records). Partitions are selected per batch. |
| `manual` | Manually select a partition for each message, requires the field `partition` to be specified. |
| `murmur2_hash` | Kafka's default hash algorithm that uses a 32-bit murmur2 hash of the key to compute which partition the record will be on. |
| `round_robin` | Round-robin's messages through all available partitions. This algorithm has lower throughput and causes higher CPU load on brokers, but can be useful if you want to ensure an even distribution of records to partitions. |
| `uniform_bytes` | Partitions based on byte size, with options for adaptive partitioning and key-based hashing in the `uniform_bytes_options` component. |


### `uniform_bytes_options`

Sets partitioner options when `partitioner` is of type `uniform_bytes`. These values will otherwise be ignored. Note, that future versions will likely see this approach reworked.


Type: `object`  

### `uniform_bytes_options.bytes`

The number of bytes the partitioner will return the same partition for.


Type: `string`  
Default: `"1MB"`  

### `uniform_bytes_options.adaptive`

Sets a slight imbalance so that the partitioner can produce more to brokers that are less loaded.


Type: `bool`  
Default: `false`  

### `uniform_bytes_options.keys`

If `true`, uses standard hashing based on record key for records with non-nil keys.


Type: `bool`  
Default: `false`  

### `partition`

An optional explicit partition to set for each message. This field is only relevant when the `partitioner` is set to `manual`. The provided interpolation string must be a valid integer.
This field supports [interpolation functions](/docs/configuration/interpolation#bloblang-queries).


Type: `string`  

```yml
# Examples

partition: ${! metadata("partition") }
```

### `client_id`

An identifier for the client connection.


Type: `string`  
Default: `"bento"`  

### `rack_id`

A rack identifier for this client.


Type: `string`  
Default: `""`  

### `idempotent_write`

Enable the idempotent write producer option. This requires the `IDEMPOTENT_WRITE` permission on `CLUSTER` and can be disabled if this permission is not available.


Type: `bool`  
Default: `true`  

### `metadata`

Determine which (if any) metadata values should be added to messages as headers.


Type: `object`  

### `metadata.include_prefixes`

Provide a list of explicit metadata key prefixes to match against.


Type: `array`  
Default: `[]`  

```yml
# Examples

include_prefixes:
  - foo_
  - bar_

include_prefixes:
  - kafka_

include_prefixes:
  - content-
```

### `metadata.include_patterns`

Provide a list of explicit metadata key regular expression (re2) patterns to match against.


Type: `array`  
Default: `[]`  

```yml
# Examples

include_patterns:
  - .*

include_patterns:
  - _timestamp_unix$
```

### `max_in_flight`

The maximum number of batches to be sending in parallel at any given time.


Type: `int`  
Default: `10`  

### `timeout`

The maximum period of time to wait for message sends before abandoning the request and retrying


Type: `string`  
Default: `"10s"`  

### `batching`

Allows you to configure a [batching policy](/docs/configuration/batching).


Type: `object`  

```yml
# Examples

batching:
  byte_size: 5000
  count: 0
  period: 1s

batching:
  count: 10
  period: 1s

batching:
  check: this.contains("END BATCH")
  count: 0
  period: 1m

batching:
  count: 10
  jitter: 0.1
  period: 10s
```

### `batching.count`

A number of messages at which the batch should be flushed. If `0` disables count based batching.


Type: `int`  
Default: `0`  

### `batching.byte_size`

An amount of bytes at which the batch should be flushed. If `0` disables size based batching.


Type: `int`  
Default: `0`  

### `batching.period`

A period in which an incomplete batch should be flushed regardless of its size.


Type: `string`  
Default: `""`  

```yml
# Examples

period: 1s

period: 1m

period: 500ms
```

### `batching.jitter`

A non-negative factor that adds random delay to batch flush intervals, where delay is determined uniformly at random between `0` and `jitter * period`. For example, with `period: 100ms` and `jitter: 0.1`, each flush will be delayed by a random duration between `0-10ms`.


Type: `float`  
Default: `0`  

```yml
# Examples

jitter: 0.01

jitter: 0.1

jitter: 1
```

### `batching.check`

A [Bloblang query](/docs/guides/bloblang/about/) that should return a boolean value indicating whether a message should end a batch.


Type: `string`  
Default: `""`  

```yml
# Examples

check: this.type == "end_of_transaction"
```

### `batching.processors`

A list of [processors](/docs/components/processors/about) to apply to a batch as it is flushed. This allows you to aggregate and archive the batch however you see fit. Please note that all resulting messages are flushed as a single batch, therefore splitting the batch into smaller batches using these processors is a no-op.


Type: `array`  

```yml
# Examples

processors:
  - archive:
      format: concatenate

processors:
  - archive:
      format: lines

processors:
  - archive:
      format: json_array
```

### `max_message_bytes`

The maximum space in bytes than an individual message may take, messages larger than this value will be rejected. This field corresponds to Kafka's `max.message.bytes`.


Type: `string`  
Default: `"1MB"`  

```yml
# Examples

max_message_bytes: 100MB

max_message_bytes: 50mib
```

### `max_buffered_records`

Sets the max amount of records the client will buffer, blocking produces until records are finished if this limit is reached. This overrides the `franz-kafka` default of 10,000.


Type: `int`  
Default: `10000`  

### `metadata_max_age`

This sets the maximum age for the client's cached metadata, to allow detection of new topics, partitions, etc.


Type: `string`  
Default: `"5m"`  

### `compression`

Optionally set an explicit compression type. The default preference is to use snappy when the broker supports it, and fall back to none if not.


Type: `string`  
Options: `lz4`, `snappy`, `gzip`, `none`, `zstd`.

### `tls`

Custom TLS settings can be used to override system defaults.


Type: `object`  

### `tls.enabled`

Whether custom TLS settings are enabled.


Type: `bool`  
Default: `false`  

### `tls.skip_cert_verify`

Whether to skip server side certificate verification.


Type: `bool`  
Default: `false`  

### `tls.enable_renegotiation`

Whether to allow the remote server to repeatedly request renegotiation. Enable this option if you're seeing the error message `local error: tls: no renegotiation`.


Type: `bool`  
Default: `false`  
Requires version 1.0.0 or newer  

### `tls.root_cas`

An optional root certificate authority to use. This is a string, representing a certificate chain from the parent trusted root certificate, to possible intermediate signing certificates, to the host certificate.
:::warning Secret
This field contains sensitive information that usually shouldn't be added to a config directly, read our [secrets page for more info](/docs/configuration/secrets).
:::


Type: `string`  
Default: `""`  

```yml
# Examples

root_cas: |-
  -----BEGIN CERTIFICATE-----
  ...
  -----END CERTIFICATE-----
```

### `tls.root_cas_file`

An optional path of a root certificate authority file to use. This is a file, often with a .pem extension, containing a certificate chain from the parent trusted root certificate, to possible intermediate signing certificates, to the host certificate.


Type: `string`  
Default: `""`  

```yml
# Examples

root_cas_file: ./root_cas.pem
```

### `tls.client_certs`

A list of client certificates to use. For each certificate either the fields `cert` and `key`, or `cert_file` and `key_file` should be specified, but not both.


Type: `array`  
Default: `[]`  

```yml
# Examples

client_certs:
  - cert: foo
    key: bar

client_certs:
  - cert_file: ./example.pem
    key_file: ./example.key
```

### `tls.client_certs[].cert`

A plain text certificate to use.


Type: `string`  
Default: `""`  

### `tls.client_certs[].key`

A plain text certificate key to use.
:::warning Secret
This field contains sensitive information that usually shouldn't be added to a config directly, read our [secrets page for more info](/docs/configuration/secrets).
:::


Type: `string`  
Default: `""`  

### `tls.client_certs[].cert_file`

The path of a certificate to use.


Type: `string`  
Default: `""`  

### `tls.client_certs[].key_file`

The path of a certificate key to use.


Type: `string`  
Default: `""`  

### `tls.client_certs[].password`

A plain text password for when the private key is password encrypted in PKCS#1 or PKCS#8 format. The obsolete `pbeWithMD5AndDES-CBC` algorithm is not supported for the PKCS#8 format. Warning: Since it does not authenticate the ciphertext, it is vulnerable to padding oracle attacks that can let an attacker recover the plaintext.
:::warning Secret
This field contains sensitive information that usually shouldn't be added to a config directly, read our [secrets page for more info](/docs/configuration/secrets).
:::


Type: `string`  
Default: `""`  

```yml
# Examples

password: foo

password: ${KEY_PASSWORD}
```

### `sasl`

Specify one or more methods of SASL authentication. SASL is tried in order; if the broker supports the first mechanism, all connections will use that mechanism. If the first mechanism fails, the client will pick the first supported mechanism. If the broker does not support any client mechanisms, connections will fail.


Type: `array`  

```yml
# Examples

sasl:
  - mechanism: SCRAM-SHA-512
    password: bar
    username: foo
```

### `sasl[].mechanism`

The SASL mechanism to use.


Type: `string`  

| Option | Summary |
|---|---|
| `AWS_MSK_IAM` | AWS IAM based authentication as specified by the 'aws-msk-iam-auth' java library. |
| `OAUTHBEARER` | OAuth Bearer based authentication. |
| `PLAIN` | Plain text authentication. |
| `SCRAM-SHA-256` | SCRAM based authentication as specified in RFC5802. |
| `SCRAM-SHA-512` | SCRAM based authentication as specified in RFC5802. |
| `none` | Disable sasl authentication |


### `sasl[].username`

A username to provide for PLAIN or SCRAM-* authentication.


Type: `string`  
Default: `""`  

### `sasl[].password`

A password to provide for PLAIN or SCRAM-* authentication.
:::warning Secret
This field contains sensitive information that usually shouldn't be added to a config directly, read our [secrets page for more info](/docs/configuration/secrets).
:::


Type: `string`  
Default: `""`  

### `sasl[].token`

The token to use for a single session's OAUTHBEARER authentication.


Type: `string`  
Default: `""`  

### `sasl[].extensions`

Key/value pairs to add to OAUTHBEARER authentication requests.


Type: `object`  

### `sasl[].aws`

Contains AWS specific fields for when the `mechanism` is set to `AWS_MSK_IAM`.


Type: `object`  

### `sasl[].aws.region`

The AWS region to target.


Type: `string`  
Default: `""`  

### `sasl[].aws.endpoint`

Allows you to specify a custom endpoint for the AWS API.


Type: `string`  
Default: `""`  

### `sasl[].aws.credentials`

Optional manual configuration of AWS credentials to use. More information can be found [in this document](/docs/guides/cloud/aws).


Type: `object`  

### `sasl[].aws.credentials.profile`

A profile from `~/.aws/credentials` to use.


Type: `string`  
Default: `""`  

### `sasl[].aws.credentials.id`

The ID of credentials to use.


Type: `string`  
Default: `""`  

### `sasl[].aws.credentials.secret`

The secret for the credentials being used.
:::warning Secret
This field contains sensitive information that usually shouldn't be added to a config directly, read our [secrets page for more info](/docs/configuration/secrets).
:::


Type: `string`  
Default: `""`  

### `sasl[].aws.credentials.token`

The token for the credentials being used, required when using short term credentials.


Type: `string`  
Default: `""`  

### `sasl[].aws.credentials.from_ec2_role`

Use the credentials of a host EC2 machine configured to assume [an IAM role associated with the instance](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html).


Type: `bool`  
Default: `false`  
Requires version 1.0.0 or newer  

### `sasl[].aws.credentials.role`

A role ARN to assume.


Type: `string`  
Default: `""`  

### `sasl[].aws.credentials.role_external_id`

An external ID to provide when assuming a role.


Type: `string`  
Default: `""`  


