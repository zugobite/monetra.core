# Best Practices

Recommended patterns and anti-patterns for building reliable applications with `monetra-core`.

---

## Table of Contents

- [Design Patterns](#patterns)
- [Anti-Patterns to Avoid](#anti-patterns)
- [Testing Money](#testing)
- [Performance Tips](#performance)

---

## Design Patterns {#patterns}

### 1. Centralise Currency Configuration

Define currencies and tokens in a single module:

```typescript
// config/currencies.ts
import { getCurrency, defineToken } from "monetra-core";

export const USD = getCurrency("USD");
export const EUR = getCurrency("EUR");
export const GBP = getCurrency("GBP");

export const REWARDS = defineToken({
  code: "REWARDS",
  symbol: "★",
  decimals: 0,
  type: "custom",
});

export type SupportedCurrency = "USD" | "EUR" | "GBP" | "REWARDS";
```

### 2. Create Money Factory Functions

Wrap common creation patterns:

```typescript
import { money, Money } from "monetra-core";

export function usd(amount: string): Money {
  return money(amount, "USD");
}

export function fromApiResponse(cents: number, currency: string): Money {
  return Money.fromMinor(cents, currency);
}
```

### 3. Use Allocation Instead of Division

When splitting money, prefer `allocate()` over `divide()` to avoid losing cents:

```typescript
// ❌ Division loses the remainder
const perPerson = total.divide(3, { rounding: RoundingMode.HALF_DOWN });
// 3 × $3.33 = $9.99 (lost $0.01)

// ✅ Allocation distributes the remainder
const [a, b, c] = total.allocate([1, 1, 1]);
// $3.34 + $3.33 + $3.33 = $10.00 (exact)
```

### 4. Explicit Rounding at Boundaries

Apply rounding at system boundaries (API responses, database writes, display), not in intermediate calculations:

```typescript
// ✅ Good: round once at the end
const subtotal = price.multiply(quantity);
const withTax = subtotal.addPercent(taxRate);
const display = withTax.format(); // rounding happens here

// ❌ Avoid: rounding at every step compounds errors
```

---

## Anti-Patterns to Avoid {#anti-patterns}

### ❌ Using `number` for Money

```typescript
// ❌ NEVER
const total = 19.99 + 5.01; // 25.000000000000004

// ✅ ALWAYS
const total = money("19.99", "USD").add(money("5.01", "USD"));
```

### ❌ Ignoring Currency

```typescript
// ❌ Loses currency context
function getTotal(amounts: number[]): number {
  return amounts.reduce((a, b) => a + b, 0);
}

// ✅ Currency-aware
function getTotal(amounts: Money[]): Money {
  return amounts.reduce((sum, m) => sum.add(m));
}
```

### ❌ Silent Rounding

```typescript
// ❌ Hides precision loss
Math.round(price * 100) / 100;

// ✅ Explicit and traceable
price.divide(3, { rounding: RoundingMode.HALF_UP });
```

### ❌ Mutating Money

```typescript
// ❌ Money is immutable — this does nothing
price.add(tax); // return value discarded

// ✅ Capture the result
const total = price.add(tax);
```

---

## Testing Money {#testing}

### Test with Exact Values

```typescript
import { money } from "monetra-core";

test("calculates order total correctly", () => {
  const price = money("19.99", "USD");
  const qty = 3;
  const total = price.multiply(qty);

  expect(total.equals(money("59.97", "USD"))).toBe(true);
});
```

### Test Allocation Invariants

```typescript
test("allocation preserves total", () => {
  const original = money("100.00", "USD");
  const parts = original.allocate([1, 1, 1]);

  const sum = parts.reduce((a, b) => a.add(b));
  expect(sum.equals(original)).toBe(true);
});
```

### Test Edge Cases

```typescript
test("handles zero amounts", () => {
  const zero = Money.zero("USD");
  expect(zero.isZero()).toBe(true);
  expect(zero.add(money("5.00", "USD")).format()).toBe("$5.00");
});

test("prevents currency mixing", () => {
  expect(() => {
    money("10", "USD").add(money("10", "EUR"));
  }).toThrow(CurrencyMismatchError);
});
```

### Property-Based Testing

Use `fast-check` to verify mathematical invariants:

```typescript
import fc from "fast-check";

test("addition is commutative", () => {
  fc.assert(
    fc.property(fc.bigInt(), fc.bigInt(), (a, b) => {
      const ma = Money.fromMinor(a, "USD");
      const mb = Money.fromMinor(b, "USD");
      expect(ma.add(mb).equals(mb.add(ma))).toBe(true);
    })
  );
});
```

---

## Performance Tips {#performance}

### Prefer `fromMinor` When Possible

`fromMinor` avoids string parsing and is the fastest way to create `Money`:

```typescript
// Fastest
Money.fromMinor(1999, "USD");

// Slower (parses decimal string)
money("19.99", "USD");
```

### Batch Operations

When processing many values, reduce object creation:

```typescript
// ✅ Use allocate for multi-way splits (one call)
const shares = total.allocate(ratios);

// ❌ Avoid: calling divide N times
ratios.map(r => total.divide(r, { rounding: RoundingMode.HALF_UP }));
```

### Tree-Shaking

Import only what you need for optimal bundle size:

```typescript
// ✅ Named imports — tree-shakeable
import { money, Money, RoundingMode } from "monetra-core";

// ❌ Avoid namespace imports
import * as Core from "monetra-core";
```
