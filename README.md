![Monetra](./monetra_core.png)

# monetra-core

Precision BigInt money for TypeScript. Immutable, integer-based monetary values with ISO 4217 currencies, allocation, rounding, formatting, and custom token support. Zero dependencies.

[![CI](https://github.com/zugobite/monetra.core/actions/workflows/ci.yml/badge.svg)](https://github.com/zugobite/monetra.core/actions/workflows/ci.yml)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-0-brightgreen.svg)]()

### Package Information

[![npm version](https://img.shields.io/npm/v/monetra-core.svg)](https://www.npmjs.com/package/monetra-core)
[![npm downloads](https://img.shields.io/npm/dm/monetra-core.svg)](https://www.npmjs.com/package/monetra-core)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-blue.svg)](https://www.typescriptlang.org/)

---

## Why monetra-core?

**💰 BigInt Precision** — No floating-point surprises. Ever. Monetary values are stored in minor units as `BigInt`, eliminating the classic `0.1 + 0.2` problem.

**🔒 Immutable by Design** — Every operation returns a new `Money` instance. No mutation, no surprises.

**🌍 ISO 4217 Built-in** — 60+ currencies with correct decimal precision out of the box, plus custom token support for crypto (up to 18 decimals).

**⚖️ Lossless Allocation** — Split money across recipients without losing a single cent. Remainder is distributed deterministically.

**🎯 Explicit Rounding** — Six rounding modes (`HALF_UP`, `HALF_DOWN`, `HALF_EVEN`, `FLOOR`, `CEIL`, `TRUNCATE`). No silent precision loss.

**📦 Zero Dependencies** — No supply chain risks. No transitive vulnerabilities. Just TypeScript.

```typescript
import { money, Money } from "monetra-core";

// Create from major units (string) or minor units (number/bigint)
const price = money("19.99", "USD");
const tax = Money.fromMinor(199, "USD");
const total = price.add(tax);

console.log(total.format()); // "$21.98"

// Lossless allocation
const [a, b, c] = money("10.00", "USD").allocate([1, 1, 1]);
// $3.34, $3.33, $3.33 — no cents lost
```

---

## Installation

```bash
# npm
npm install monetra-core

# yarn
yarn add monetra-core

# pnpm
pnpm add monetra-core
```

**Requirements:** Node.js 18+ or any modern runtime/browser with `BigInt` support.

---

## Features

### Core Money Operations

- Integer-based storage (`BigInt`) eliminates floating-point errors
- Immutable API — all operations return new instances
- Full arithmetic: `add`, `subtract`, `multiply`, `divide`, `abs`, `negate`, `clamp`
- Comparisons: `equals`, `greaterThan`, `lessThan`, `greaterThanOrEqual`, `lessThanOrEqual`, `compare`
- Static helpers: `Money.min()`, `Money.max()`, `Money.zero()`
- JSON serialization with `toJSON()` and `Money.reviver`

### Currency & Tokens

- ISO 4217 currency registry with 60+ currencies
- Automatic decimal precision handling (0–18 decimals)
- Custom token definitions via `defineToken()` for crypto and loyalty points
- Built-in presets: ETH (18 decimals), BTC (8), USDC (6), USDT (6)

### Allocation

- Deterministic splitting across any number of ratios
- Remainder distributed fairly — total always equals the original amount
- Percentage helpers: `percentage()`, `addPercent()`, `subtractPercent()`, `split()`

### Rounding

- Six explicit modes: `HALF_UP`, `HALF_DOWN`, `HALF_EVEN`, `FLOOR`, `CEIL`, `TRUNCATE`
- Required on lossy operations (division) — no silent precision loss

### Formatting & Parsing

- Locale-aware formatting via `Intl.NumberFormat`
- Accounting format with parenthesised negatives
- Currency code/name display options
- `parseLocaleString()` and `parseLocaleToMinor()` for parsing

### Multi-Currency

- `Converter` class with exchange rate management
- Historical rate support via `addHistoricalRate()`
- `MoneyBag` for aggregating values across currencies
- Type-safe currency mismatch prevention

### Error Handling

- Custom error classes with error codes for programmatic handling
- `CurrencyMismatchError`, `InvalidArgumentError`, `OverflowError`, `RoundingRequiredError`, `InsufficientFundsError`, `InvalidPrecisionError`

---

## Quick Start

### Safe Money Handling

```typescript
import { money, Money } from "monetra-core";

const price = money("19.99", "USD");
const quantity = 3;
const subtotal = price.multiply(quantity);
const discount = subtotal.percentage(10); // 10% of subtotal
const total = subtotal.subtract(discount);

console.log(total.format()); // "$53.97"
```

### Currency Conversion

```typescript
import { Money, Converter } from "monetra-core";

const converter = new Converter("USD");
converter.setRate("EUR", 0.92);
converter.setRate("GBP", 0.79);

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

### Allocation

```typescript
import { money } from "monetra-core";

// Split a bill 3 ways
const bill = money("100.00", "USD");
const [share1, share2, share3] = bill.allocate([1, 1, 1]);
// $33.34, $33.33, $33.33 — no cents lost

// Weighted split (50%, 30%, 20%)
const [a, b, c] = bill.allocate([50, 30, 20]);
// $50.00, $30.00, $20.00
```

---

## Documentation

Full documentation is available in the [docs](docs/index.md) directory:

**Getting Started**

- [Core Concepts](docs/core-concepts.md)
- [Installation & Setup](docs/getting-started.md)

**API Reference**

- [Money & Currency](docs/core/money.md)
- [Currency Registry](docs/core/currency.md)

**Guides**

- [Precise Allocation](docs/guides/allocation.md)
- [Custom Tokens](docs/guides/custom-tokens.md)
- [Error Handling](docs/guides/error-handling.md)
- [Formatting](docs/guides/formatting.md)

---

## Testing

```bash
# Run the test suite
pnpm test

# Run property-based verification
pnpm test:property

# Run coverage report
pnpm test:coverage
```

---

## Relationship to Monetra

`monetra-core` is the foundation layer extracted from the [Monetra](https://github.com/zugobite/monetra) framework. It contains only the correctness primitives — BigInt-based money, currencies, allocation, rounding, and formatting.

Higher-level features (financial math, ledgers, double-entry bookkeeping) live in the full `monetra` package.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

Please review our [Code of Conduct](CODE_OF_CONDUCT.md) before contributing.

---

## Security

Report security vulnerabilities according to our [Security Policy](SECURITY.md).

---

## License

MIT — see [LICENSE](LICENSE) for details.
