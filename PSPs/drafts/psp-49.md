# Base Weights

- **PSP Number:** 49
- **Authors:** Fabio Lama <fabio.lama@pm.me>
- **Status:** Draft
- **Created:** 2022-07-21
- **Reference Implementation:** https://github.com/paritytech/substrate/pull/10918

## Summary

This standard sets weights on expensive resources such as computation, database
access and bandwidth. The weights are determined by benchmarking the Substrate
implementation that provides those resources to the Runtime and displays the
recommended values that alternative implemenations to Substrate should adhere to
in order to unify the cost of those resources and maintain a sustainable fee
structure.

## Motivation

The Substrate implementation determined weights based on the resources that it
provides to the Runtime for consumption, such as I/O, computation and network
bandwidth. Given that alternative implementations are available and can be
developed in the future, those weights should be reflected by those alternative
implemenations so that the fee structure is justifiable for the diverse network
of participants at large. The goal is to avoid situations where alternative
implemenations are too strict or too loose in terms of costs of their resource
requirements which could lead to bad economic outcomes and a fractured network
of implemenations.

...

## Constants

A weight of _1 unit_ corresponds to one picosecond. We define a list of
constants to simplify the declaration of weights.

| Symbol  | Weight                  | Description            |
|---------|-------------------------|------------------------|
| $w_s$   | 1'000'000'000'000       | Weight per second      |
| $w_m$   | $$\frac{w_s}{1'000}$$   | Weight per millisecond |
| $w_\mu$ | $$\frac{w_m}{1'000}$$   | Weight per microsecond |
| $w_n$   | $$\frac{w_\mu}{1'000}$$ | Weight per nanosecond  |

## Extrinsic

The weight reflects the time it takes to execute a no-op extrinsic, such as
`system_remark`, respectively an extrinsic that does not perform an operation.
This weight indicates the _minimum (or base) requirement_ to execute an
extrinsic.

* `ExtrinsicBaseWeight` = $85'212 \times w_n$
	*	| Stats   | Nanoseconds |
		|---------|-------------|
		| Min     | $84'940$    |
		| Max     | $86'590$    |
		| Average | $85'212$    |
		| Median  | $85'156$    |
		| Std-Dev | $243.25$    |

	*	| Percentiles | Nanoseconds |
		|-------------|-------------|
		| 99th        | $86'269$    |
		| 95th        | $85'510$    |
		| 75th        | $85'216$    |

# Block

* `BlockExecutionWeight` = $5'797'172 \times w_n$
	*	| Stats   | Nanoseconds |
		|---------|-------------|
		| Min     | $5'722'294$ |
		| Max     | $5'960'144$ |
		| Average | $5'797'172$ |
		| Median  | $5'786'450$ |
		| Std-Dev | $48'146.81$ |

	*	| Percentiles | Nanoseconds |
		|-------------|-------------|
		| 99th        | $5'940'244$ |
		| 95th        | $5'890'113$ |
		| 75th        | $5'824'797$ |

## Database

...

## RocksDb

The weights reflect the time it takes to read and write one storage item. The
value is calculated by multiplying the *Average* of all values with `1.1` and
adding `0`.

* `read` = $20'499 \times w_n$
	*	| Stats   | Nanoseconds |
		|---------|-------------|
		| Min     | $5'015$     |
		| Max     | $1'441'022$ |
		| Average | $18'635$    |
		| Median  | $17'795$    |
		| Std-Dev | $4'829.75$  |

	*	| Percentiles | Nanoseconds |
		|-------------|-------------|
		| 99th        | $32'074$    |
		| 95th        | $26'658$    |
		| 75th        | $19'363$    |


* `write` = $83'471 \times w_n$
	*	| Stats   | Nanoseconds  |
		|---------|--------------|
		| Min     | $16'368$     |
		| Max     | $34'500'937$ |
		| Average | $75'882$     |
		| Median  | $74'236$     |
		| Std-Dev | $64'706.41$  |

	*	| Percentiles | Nanoseconds |
		|-------------|-------------|
		| 99th        | $111'151$   |
		| 95th        | $92'666$    |
		| 75th        | $80'297$    |

## ParityDb

The weights reflect the time it takes to read and write one storage item. The
value is calculated by multiplying the *Average* of all values with `1.1` and
adding `0`.

* `read`: $11'826 \times w_n$
	*	| Stats   | Nanoseconds  |
		|---------|--------------|
		| Min     | $4'611$      |
		| Max     | $13'478'005$ |
		| Average | $10'750$     |
		| Median  | $10'655$     |
		| Std-Dev | $12'214.49$  |

	*	| Percentiles | Nanoseconds |
		|-------------|-------------|
		| 99th        | $14'451$    |
		| 95th        | $12'588$    |
		| 75th        | $11'200$    |


* `write`: $38'053 \times w_n$
	*	| Stats   | Nanoseconds  |
		|---------|--------------|
		| Min     | $8'023$      |
		| Max     | $47'367'740$ |
		| Average | $34'592$     |
		| Median  | $32'703$     |
		| Std-Dev | $49'417.24$  |

	*	| Percentiles | Nanoseconds |
		|-------------|-------------|
		| 99th        | $69'379$    |
		| 95th        | $47'168$    |
		| 75th        | $35'252$    |

## Copyright

This document is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
