# Core Concepts

Understanding the design decisions behind `monetra-core` will help you use the library effectively and build financially correct applications.

---

## Table of Contents

- [Integer-Based Arithmetic](#integer-arithmetic)
- [Minor Units](#minor-units)
- [Currency & Precision](#currency-precision)
- [Immutability](#immutability)
- [Rounding Strategies](#rounding)
- [Type Safety](#type-safety)

---

## Integer-Based Arithmetic {#integer-arithmetic}

The fundamental problem with using `number` for money:

```typescript
// ❌ JavaScript floating-point
0.1 + 0.2; // 0.30000000000000004

// ✅ monetra-core uses BigInt minor units
money("0.10", "USD").add(money("0.20", "USD"));
// Internally: 10n + 20n = 30n → $0.30 (exact)
```

`monetra-core` stores all amounts as `BigInt` values in **minor units** (the smallest denomination of a currency). This eliminates floating-point errors entirely.

### Why BigInt?

| Approach        | Precision      | Speed    | Native |
| --------------- | -------------- | -------- | ------ |
| `number`        | ~15 digits     | Fastest  | Yes    |
| Decimal library | Arbitrary      | Slower   | No     |
| **BigInt**      | **Unlimited**  | **Fast** | **Yes** |

`BigInt` is a JavaScript primitive with no precision limit and no external dependencies. It's the ideal foundation for monetary arithmetic.

---

## Minor Units {#minor-units}

A "minor unit" is the smallest denomination of a currency:

| Currency | Minor Unit | Factor |
| -------- | ---------- | ------ |
| USD      | cent       | 100    |
| JPY      | yen        | 1      |
| KWD      | fils       | 1000   |
| ETH      | wei        | 10^18  |

When you write `money("19.99", "USD")`, the library stores `1999n` internally. All arithmetic operates on this integer, and conversion back to major units happens only at display time.

```typescript
const m = money("19.99", "USD");
m.amount;   // 1999n (BigInt minor units)
m.format(); // "$19.99" (converted at display time)
```

---

## Currency & Precision {#currency-precision}

Every `Money` instance is bound to a currency. The currency determines:

- **Decimal places** — USD has 2, JPY has 0, ETH has 18
- **Symbol** — $, ¥, Ξ
- **Formatting** — locale-specific display rules

Arithmetic between different currencies is not allowed and throws a `CurrencyMismatchError`:

```typescript
const usd = money("10.00", "USD");
const eur = money("10.00", "EUR");

usd.add(eur); // ❌ Throws CurrencyMismatchError

// Use a Converter for cross-currency operations
import { Converter } from "monetra-core";
const converter = new Converter("USD");
converter.setRate("EUR", 0.92);
const converted = converter.convert(eur, "USD");
usd.add(converted); // ✅ Both are USD now
```

---

## Immutability {#immutability}

Every operation on a `Money` instance returns a **new** instance. The original is never modified:

```typescript
const price = money("10.00", "USD");
const taxed = price.addPercent(8.5);

console.log(price.format()); // "$10.00" — unchanged
console.log(taxed.format()); // "$10.85" — new instance
```

This makes `Money` safe to share across components, functions, and threads without defensive copying.

---

## Rounding Strategies {#rounding}

When an operation cannot produce an exact result (like division), `monetra-core` requires you to specify a rounding mode. This prevents silent precision loss:

```typescript
import { money, RoundingMode } from "monetra-core";

const total = money("10.00", "USD");

// ❌ This would throw RoundingRequiredError
// total.divide(3);

// ✅ Explicit rounding
total.divide(3, { rounding: RoundingMode.HALF_UP });     // $3.33
total.divide(3, { rounding: RoundingMode.HALF_EVEN });   // $3.33 (banker's)
total.divide(3, { rounding: RoundingMode.CEIL });        // $3.34
```

### Available Modes

| Mode        | Description                          | 2.5 → | -2.5 → |
| ----------- | ------------------------------------ | ----- | ------- |
| `HALF_UP`   | Round towards nearest, ties up       | 3     | -3      |
| `HALF_DOWN` | Round towards nearest, ties down     | 2     | -2      |
| `HALF_EVEN` | Round towards nearest, ties to even  | 2     | -2      |
| `FLOOR`     | Round towards negative infinity      | 2     | -3      |
| `CEIL`      | Round towards positive infinity      | 3     | -2      |
| `TRUNCATE`  | Truncate towards zero                | 2     | -2      |

**When to use which:**

- **`HALF_UP`** — Most common, good default for invoices and retail
- **`HALF_EVEN`** — Banker's rounding, minimises cumulative bias over many operations
- **`FLOOR` / `CEIL`** — When you need conservative or aggressive estimates
- **`TRUNCATE`** — When you want to discard fractional amounts

---

## Type Safety {#type-safety}

`monetra-core` is written in strict TypeScript. Currency mismatches, invalid arguments, and missing rounding modes are all caught at the type level or with descriptive runtime errors:

```typescript
import { Money, CurrencyMismatchError, RoundingRequiredError } from "monetra-core";

try {
  usd.add(eur);
} catch (e) {
  if (e instanceof CurrencyMismatchError) {
    console.log(e.expected); // "USD"
    console.log(e.received); // "EUR"
    console.log(e.code);     // "CURRENCY_MISMATCH"
  }
}
```

All error classes include:

- A machine-readable `code` from `MonetraErrorCode`
- Human-readable `message` with actionable tips
- Contextual properties (e.g., `expected`, `received`)

---

## Next Steps

- [Getting Started](./getting-started.md) — Installation and quick start
- [Allocation Guide](./guides/allocation.md) — Lossless splitting
- [Custom Tokens Guide](./guides/custom-tokens.md) — Crypto and loyalty points
