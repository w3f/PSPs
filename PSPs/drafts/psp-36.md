# Title

- **PSP Number:** [To be assigned (=number of the initial PR to the PSPs repo)]
- **Authors:** Fabio Lama <fabio.lama@pm.me>
- **Status:** Draft
- **Created:** 2022-07-06
- **Reference Implementation** https://github.com/paritytech/substrate-telemetry

## Summary

TODO

## Motivation

TODO

## Specification

TODO

### JSON Messages

#### System Connected

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

#### System Interval

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

#### Block Import

| Name   | Type   | Required | Description                 |
|--------|--------|----------|-----------------------------|
| msg    | STRING | YES      | Constant "**block.import**" |
| height | UINT   | YES      | Block number                |
| best   | HEX_32 | YES      | Block hash                  |

#### Notify Finalized

| Name   | Type   | Required | Description                     |
|--------|--------|----------|---------------------------------|
| msg    | STRING | YES      | Constant "**notify.finalized**" |
| height | STRING | YES      | Block number                    |
| hash   | HEX_32 | YES      | Block hash                      |

#### (Afg) Authority Set

| Name             | Type   | Required | Description                      |
|------------------|--------|----------|----------------------------------|
| msg              | STRING | YES      | Constant "**afg.authority_set**" |
| authority_id     | STRING | YES      |                                  |
| authorities      | STRING | YES      |                                  |
| authority_set_id | STRING | YES      |                                  |

#### Hardware Bench

| Name                        | Type   | Required | Description                    |
|-----------------------------|--------|----------|--------------------------------|
| msg                         | STRING | YES      | Constant "**sysinfo.hwbench**" |
| cpu_hashrate_score          | UINT   | YES      |                                |
| memory_memcpy_score         | UINT   | YES      |                                |
| disk_sequential_write_score | UINT   | NO       |                                |
| disk_random_write_score     | UINT   | NO       |                                |

## Tests

TODO

## Copyright

TODO

Each PSP must be labeled as placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).
