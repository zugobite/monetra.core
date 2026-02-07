# Performance

`monetra-core` uses native `BigInt` arithmetic for maximum speed. All monetary values are stored as integer minor units, avoiding the overhead of decimal string parsing during computation.

---

## Core Operations

| Operation              | Throughput              | Notes                                    |
| ---------------------- | ----------------------- | ---------------------------------------- |
| Addition               | ~38 million ops/sec     | BigInt addition on minor units           |
| Subtraction            | ~40 million ops/sec     | BigInt subtraction on minor units        |
| Multiplication         | ~8 million ops/sec      | BigInt multiplication with scaling       |
| Division               | ~8 million ops/sec      | BigInt division with rounding            |
| Equality check         | ~50 million ops/sec     | BigInt comparison                        |
| Object creation        | ~37 million ops/sec     | `Money.fromMinor()`                      |
| String parsing         | ~10 million ops/sec     | `money("19.99", "USD")`                  |

---

## Allocation

| Recipients | Throughput                    |
| ---------- | ----------------------------- |
| 3          | ~2.1 million splits/sec       |
| 10         | ~819,000 splits/sec           |
| 1,000      | ~8,900 splits/sec             |
| 10,000     | ~884 splits/sec               |

---

## Why BigInt is Fast

1. **Native runtime primitive** — no library overhead, no object allocation for arithmetic
2. **Integer-only arithmetic** — no decimal point tracking or string manipulation during computation
3. **Single representation** — amounts are always in minor units; conversion happens only at display time

---

## Tips

- Prefer `Money.fromMinor()` over `money("…")` when you already have integer amounts (avoids string parsing)
- Use `allocate()` instead of repeated `divide()` calls for multi-way splits
- Import only what you need — the package is fully tree-shakeable
