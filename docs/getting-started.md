# Getting Started

This guide will help you install `monetra-core` and start handling monetary values with precision.

---

## Installation

Install using your preferred package manager:

```bash
# npm
npm install monetra-core

# yarn
yarn add monetra-core

# pnpm
pnpm add monetra-core
```

**Requirements:** Node.js 18+ or modern browsers with `BigInt` support.

---

## Quick Start

### Creating Money

```typescript
import { money, Money } from "monetra-core";

// From a string (major units) — safest approach
const price = money("19.99", "USD");

// From minor units (cents)
const tax = Money.fromMinor(199, "USD");

// Aliases
const fromCents = Money.fromCents(500, "USD");     // same as fromMinor
const fromDecimal = Money.fromDecimal("5.00", "USD"); // same as fromMajor
```

### Arithmetic

```typescript
const a = money("10.00", "USD");
const b = money("3.50", "USD");

const sum = a.add(b);           // $13.50
const diff = a.subtract(b);     // $6.50
const doubled = a.multiply(2);  // $20.00
```

### Division (Requires Rounding)

Division can produce non-terminating decimals, so a rounding mode is required:

```typescript
import { money, RoundingMode } from "monetra-core";

const total = money("10.00", "USD");
const perPerson = total.divide(3, { rounding: RoundingMode.HALF_UP });
// $3.33
```

### Allocation

For lossless splitting (no cents lost):

```typescript
const bill = money("100.00", "USD");
const [a, b, c] = bill.allocate([1, 1, 1]);
// $33.34, $33.33, $33.33 — total is still $100.00
```

### Comparisons

```typescript
const x = money("10.00", "USD");
const y = money("20.00", "USD");

x.lessThan(y);          // true
x.greaterThan(y);       // false
x.equals(money("10.00", "USD")); // true

Money.min(x, y);  // $10.00
Money.max(x, y);  // $20.00
```

### Formatting

```typescript
const amount = money("1234.56", "USD");

amount.format();                    // "$1,234.56"
amount.format({ locale: "de-DE" }); // "1.234,56 $"
amount.format({ accounting: true }); // "$1,234.56" (negatives in parentheses)
amount.toDecimalString();           // "1234.56"
```

### Currency Conversion

```typescript
import { Money, Converter } from "monetra-core";

const converter = new Converter("USD");
converter.setRate("EUR", 0.92);

const usd = Money.fromMajor("100", "USD");
const eur = converter.convert(usd, "EUR");
console.log(eur.format()); // "€92.00"
```

### Custom Tokens

```typescript
import { defineToken, money } from "monetra-core";

defineToken({
  code: "LOYALTY",
  name: "Loyalty Points",
  decimals: 0,
  symbol: "★",
});

const points = money("500", "LOYALTY");
console.log(points.format()); // "★500"
```

### JSON Serialization

```typescript
const m = money("42.00", "USD");

const json = JSON.stringify(m); // includes amount and currency
const parsed = JSON.parse(json, Money.reviver); // restores Money instance
```

---

## TypeScript

`monetra-core` is written in TypeScript with strict types. All public APIs are fully typed.

```typescript
import { Money, Currency, RoundingMode } from "monetra-core";

function applyDiscount(price: Money, percent: number): Money {
  return price.subtractPercent(percent);
}
```

---

## Next Steps

- [Core Concepts](./core-concepts.md) — Understand BigInt, minor units, and rounding
- [Allocation Guide](./guides/allocation.md) — Lossless splitting patterns
- [Custom Tokens Guide](./guides/custom-tokens.md) — Crypto and loyalty points
- [Error Handling Guide](./guides/error-handling.md) — Typed errors and recovery
