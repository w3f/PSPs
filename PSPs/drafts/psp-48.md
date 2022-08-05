# Node Telemetry Protocol

- **PSP Number:** 48
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
The payloads are documented in [JSON Messagew](#json-messages).

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

Information about the system environment.

| Name          | Type          | Required | Description                     |
|---------------|---------------|----------|---------------------------------|
| msg           | STRING        | YES      | Constant "**system.connected**" |
| genesis_hash  | HEX_32        | YES      | Genesis hash of the chain       |
| chain         | STRING        | YES      | Name of the chain               |
| name          | STRING        | YES      | Name of the node                |
| implemenation | STRING        | YES      | Name of the node implemenation  |
| version       | STRING        | YES      | Node version, e.g. `0.9.17-75dd6c7d0`|
| validator     | STRING        | NO       |                                 |
| network_id    | STRING_64     | YES      | Network Id, e.g. `polkadot` or `ksmcc3`|
| startup_time  | STRING        | NO       | Startup time of the node        |
| target_os     | STRING        | NO       | Operating system, e.g. `linux`  |
| target_arch   | STRING        | NO       | CPU architecture, e.g `x86_64`  |
| target_env    | STRING        | NO       | OS environment, e.g. `gnu`      |
| sysinfo       | _NodeSysInfo_ | NO       |                                 |

The fields `version`, `target_arch`, `target_os` and `target_env` concatenate
to, for example, `0.9.17-75dd6c7d0-x86-linux-gnu`.

The structure _NodeSysInfo_ is structured as:

| Name               | Type    | Required | Description                                      |
|--------------------|---------|----------|--------------------------------------------------|
| cpu                | STRING  | NO       | Name of the CPU                                  |
| memory             | UINT    | NO       | Memory in bytes                                  |
| core_count         | UINT    | NO       | Number of cores                                  |
| linux_kernel       | STRING  | NO       | Name of the Linux kernel                         |
| linux_distro       | STRING  | NO       | Name of the Linux distro                         |
| is_virtual_machine | BOOLEAN | NO       | Whether the node is running in a virtual machine |

### System Interval

Information about the state of the system.

| Name                  | Type   | Required | Description                       |
|-----------------------|--------|----------|-----------------------------------|
| msg                   | STRING | YES      | Constant "**system.interval**"    |
| peers                 | UINT   | NO       | Number of connected peers         |
| txcount               | UINT   | NO       | Number of pending transactions    |
| bandwidth_upload      | FLOAT  | NO       | Upload speed in kB/s              |
| bandwith_download     | FLOAT  | NO       | Download speed in kB/s            |
| finalized_height      | UINT   | NO       | Latest finalized block number     |
| finalized_hash        | HEX_32 | NO       | Latest finalized block hash       |
| height                | UINT   |          | Latest non-finalized block number |
| best                  | HEX_32 | NO       | Latest non-finalized block hash   |
| used_state_cache_size | FLOAT  | NO       | Size of the node's state cache in MB/s |

### Block Import

Information about an imported block.

| Name   | Type   | Required | Description                 |
|--------|--------|----------|-----------------------------|
| msg    | STRING | YES      | Constant "**block.import**" |
| height | UINT   | YES      | Block number                |
| best   | HEX_32 | YES      | Block hash                  |

### Notify Finalized

Information about an imported, finalized block.

| Name   | Type   | Required | Description                     |
|--------|--------|----------|---------------------------------|
| msg    | STRING | YES      | Constant "**notify.finalized**" |
| height | STRING | YES      | Block number                    |
| hash   | HEX_32 | YES      | Block hash                      |

### (Afg) Authority Set

Information about the GRANDPA authority set.

| Name             | Type   | Required | Description                      |
|------------------|--------|----------|----------------------------------|
| msg              | STRING | YES      | Constant "**afg.authority_set**" |
| authority_id     | STRING | YES      | The public key if the local node is an elected authority, empty value otherwise |
| authorities      | STRING | YES      |                                  |
| authority_set_id | STRING | YES      | The Set Id of the current authority list |

### Hardware Bench

Information about the hardware performance of the system.

| Name                        | Type   | Required | Description                    |
|-----------------------------|--------|----------|--------------------------------|
| msg                         | STRING | YES      | Constant "**sysinfo.hwbench**" |
| cpu_hashrate_score          | UINT   | YES      |                                |
| memory_memcpy_score         | UINT   | YES      |                                |
| disk_sequential_write_score | UINT   | NO       |                                |
| disk_random_write_score     | UINT   | NO       |                                |

## Recommended Behavior

The telemetry protocol of a Polkadot Host implementation can behave according to
its own rules. However, this section describes the recommended behavior that the
implementation _should_ abide by, as is reflected in the substrate
implementation.

* The [`system.connected`](#system-connected) message should be sent once when
the telemetry client starts, respectively on (re-)connection to the telemetry
server.
* The [`system.interval`](#system-interval) message should be sent every five
  seconds.
* The [`block.import`](#block-import) message should be sent everytime a block
  is imported when the client is _fully synced_. The message should **NOT** be
  sent during sync in order not to spam the telemetry server. While the client
  is syncing, this message should be sent every 10 blocks.
* The [`notify.finalized`](#notify-finalized) message should be sent everytime a
  _finalized_ block is imported when the client is _fully synced_. This message
  should **NOT** be sent during sync in order not to spam the telemetry server.
  This message should also trigger a [`block.import`](#block-import) message.
* The [`afg.authority_set`](#afg-authority-set) message should be sent everytime
  the telemetry client starts, respectively on (re-)connection to the telemetry
  server and everytime the authority set changes (start of a new Era).
* The [`sysinfo.hwbench`](#hardware-bench) message should be sent everytime the
  telemetry client starts, respectively on (re-)connection to the telemetry
  server.

## Copyright

This PSP is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
