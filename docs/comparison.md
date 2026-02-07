# Library Comparison

A technical comparison between `@monetra/core` and other JavaScript/TypeScript money handling libraries.

---

## Overview

| Library           | TypeScript | Decimal Approach               | Scope   |
| ----------------- | ---------- | ------------------------------ | ------- |
| **@monetra/core** | Native     | Integer (BigInt)               | Library |
| Dinero.js 2.x     | Native     | Integer (BigInt)               | Library |
| currency.js 2.x   | Types      | Floating-point with correction | Library |
| big.js 6.x        | Types      | Arbitrary precision decimal    | Library |
| decimal.js 10.x   | Types      | Arbitrary precision decimal    | Library |
| js-money 0.7.x    | Types      | Floating-point                 | Library |

---

## Feature Comparison

| Feature                                | @monetra/core | Dinero.js | currency.js | big.js | decimal.js | js-money |
| -------------------------------------- | ------------- | --------- | ----------- | ------ | ---------- | -------- |
| Integer-based storage                  | Yes (BigInt)  | Yes       | No          | No     | No         | No       |
| Immutable objects                      | Yes           | Yes       | Yes         | Yes    | Yes        | Yes      |
| ISO 4217 currencies                    | Built-in      | Separate  | Manual      | N/A    | N/A        | Built-in |
| Custom token support                   | Yes           | No        | No          | N/A    | N/A        | No       |
| Cryptocurrency precision (18 decimals) | Yes           | Yes       | No          | Yes    | Yes        | No       |
| Formatting with locales                | Yes           | Yes       | Limited     | No     | No         | Yes      |
| Allocation/splitting                   | Yes           | Yes       | No          | No     | No         | No       |
| Currency conversion                    | Yes           | No        | No          | No     | No         | No       |
| Zero dependencies                      | Yes           | Yes       | Yes         | Yes    | Yes        | Yes      |
| Tree-shakeable                         | Yes           | Yes       | Yes         | N/A    | N/A        | Yes      |

---

## When to Use Each Library

### Use @monetra/core when:

- You need correct integer-based money handling with BigInt
- You want built-in ISO 4217 currencies with no configuration
- You need lossless allocation (splitting without losing cents)
- You work with crypto tokens requiring up to 18 decimals
- You want explicit rounding with no silent precision loss
- You need currency conversion with rate management
- Zero dependencies is a requirement

### Use Dinero.js when:

- You need a well-established, battle-tested money library
- You prefer a functional API style
- You don't need built-in conversion or allocation

### Use currency.js when:

- You need a lightweight, simple solution
- Floating-point precision is acceptable for your use case
- You don't need multi-currency support

### Use big.js / decimal.js when:

- You need arbitrary-precision decimal arithmetic (not money-specific)
- You're building scientific or mathematical applications
- You don't need currency awareness

---

## Code Comparison

### Creating Money

```typescript
// @monetra/core
import { money } from "@monetra/core";
const price = money("19.99", "USD");

// Dinero.js
import { dinero, USD } from "dinero.js";
const price = dinero({ amount: 1999, currency: USD });

// currency.js
import currency from "currency.js";
const price = currency(19.99);

// big.js
import Big from "big.js";
const price = new Big("19.99");
```

### Splitting

```typescript
// @monetra/core — built-in, lossless
const [a, b, c] = money("10.00", "USD").allocate([1, 1, 1]);
// $3.34, $3.33, $3.33

// Dinero.js — built-in
import { allocate } from "dinero.js";
const parts = allocate(price, [1, 1, 1]);

// currency.js — not built-in, manual implementation needed
// big.js — not built-in, manual implementation needed
```
