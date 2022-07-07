# Node Telemetry Protocol

- **PSP Number:** [To be assigned (=number of the initial PR to the PSPs repo)]
- **Authors:** Fabio Lama <fabio.lama@pm.me>
- **Status:** Draft
- **Created:** 2022-07-06
- **Reference Implementation** https://github.com/paritytech/substrate-telemetry

## Summary

This document described the protocol to send and receive telemetry information
produced by Polkadot Host implemenations. This information can be processed and
displayed as seen on https://telemetry.polkadot.io, for example.

## Motivation

Polkadot Host implemenations can send telemetry information about their current
state and activity to one or more endpoints that collect and process that
information. This gives observers an insight on the current state of the
network, which includes block imports, finalization, ressource requirements and
so on. Sending telemetry information is a voluntary act and is not required for Polkadot
and Kusama to function properly, respectively it's not part of the consensus protocol.

All telemetry messges are serialized in JSON format.

## Note on Security

All telemetry information sent or collected is entirely **subjective** and its
origin is untrusted. The telemetry information should offer an insight into the
network and should not be interpreted at face value. Impersonations of
participants cannot be prevented.

## Transport

Telemetry messages are send over the websocket procotol. Conventially, the
websocket stream connects on port `8000` to the endpoint `/submit`, which is not
a requirement. Servers that collect telemetry data might have different
requirements.

An example of such an endpoint might look like `wss://telemetry.example.com:8000/submit`.

## Versioning

All telemetry messages are versioned, allowing for more message types in the future.
This standard introduces two versions represented as JSON messages, `V1` and `V2`.
The payload is described in [JSON Messagew](#json-messages).

### V2

| Name    | Type | Required | Description         |
|---------|------|----------|---------------------|
| id      | UINT | YES      |                     |
| payload | ANY  | YES      | The payload message |

Example:

```json
{
	"id": 1,
	"payload": {
		"msg":"notify.finalized",
		"best":"0x031c3521ca2f9c673812d692fc330b9a18e18a2781e3f9976992f861fd3ea0cb",
		"height":"50"
	}
}
```

### V1

In this version, the message payload is sent directly. This version can be
detected if it cannot be decoded as `V2`. This version serves primarily for
backwards-compatibility reasons.

| Name | Type | Required | Description                |
|------|------|----------|----------------------------|
| -    | ANY  | YES      | Message is passed directly |

Example:

```json
{
	"msg":"notify.finalized",
	"best":"0x031c3521ca2f9c673812d692fc330b9a18e18a2781e3f9976992f861fd3ea0cb",
	"height":"50"
}
```

## JSON Messages

All passed on information is subjective and in accordance with the implemenator
of this protocol.

### System Connected

| Name          | Type          | Required | Description                     |
|---------------|---------------|----------|---------------------------------|
| msg           | STRING        | YES      | Constant "**system.connected**" |
| genesis_hash  | HEX_32        | YES      | Genesis hash of the chain       |
| chain         | STRING        | YES      | Name of the chain               |
| name          | STRING        | YES      | Name of the node                |
| implemenation | STRING        | YES      |                                 |
| version       | STRING        | YES      |                                 |
| validator     | STRING        | NO       |                                 |
| network_id    | STRING_64     | YES      |                                 |
| startup_time  | STRING        | NO       |                                 |
| target_os     | STRING        | NO       |                                 |
| target_arch   | STRING        | NO       |                                 |
| target_env    | STRING        | NO       |                                 |
| sysinfo       | NODE_SYS_INFO | NO       |                                 |

NODE_SYS_INFO:

| Name               | Type    | Required | Description                 |
|--------------------|---------|----------|-----------------------------|
| cpu                | STRING  | NO       | Name of the CPU             |
| memory             | UINT    | NO       | Memory in bytes             |
| core_count         | UINT    | NO       | Number of cores             |
| linux_kernel       | STRING  | NO       | Name of the Linux kernel    |
| linux_distro       | STRING  | NO       | Name of the Linux distro    |
| is_virtual_machine | BOOLEAN | NO       | Whether the node is running |
|                    |         |          | in a virtual machine        |

### System Interval

| Name                  | Type   | Required | Description                       |
|-----------------------|--------|----------|-----------------------------------|
| msg                   | STRING | YES      | Constant "**system.interval**"    |
| peers                 | UINT   | NO       | Number of connected peers         |
| txcount               | UINT   | NO       | Number of pending transactions    |
| bandwidth_upload      | FLOAT  | NO       |                                   |
| bandwith_download     | FLOAT  | NO       |                                   |
| finalized_height      | UINT   | NO       | Latest finalized block number     |
| finalized_hash        | HEX_32 | NO       | Latest finalized block hash       |
| height                | UINT   |          | Latest non-finalized block number |
| best                  | HEX_32 | NO       | Latest non-finalized block hash   |
| used_state_cache_size | FLOAT  | NO       |                                   |

### Block Import

| Name   | Type   | Required | Description                 |
|--------|--------|----------|-----------------------------|
| msg    | STRING | YES      | Constant "**block.import**" |
| height | UINT   | YES      | Block number                |
| best   | HEX_32 | YES      | Block hash                  |

### Notify Finalized

| Name   | Type   | Required | Description                     |
|--------|--------|----------|---------------------------------|
| msg    | STRING | YES      | Constant "**notify.finalized**" |
| height | STRING | YES      | Block number                    |
| hash   | HEX_32 | YES      | Block hash                      |

### (Afg) Authority Set

| Name             | Type   | Required | Description                      |
|------------------|--------|----------|----------------------------------|
| msg              | STRING | YES      | Constant "**afg.authority_set**" |
| authority_id     | STRING | YES      |                                  |
| authorities      | STRING | YES      |                                  |
| authority_set_id | STRING | YES      |                                  |

### Hardware Bench

| Name                        | Type   | Required | Description                    |
|-----------------------------|--------|----------|--------------------------------|
| msg                         | STRING | YES      | Constant "**sysinfo.hwbench**" |
| cpu_hashrate_score          | UINT   | YES      |                                |
| memory_memcpy_score         | UINT   | YES      |                                |
| disk_sequential_write_score | UINT   | NO       |                                |
| disk_random_write_score     | UINT   | NO       |                                |

## Copyright

This PSP is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
