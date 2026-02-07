# @zugobite/monetra-core Documentation

Precision BigInt money for TypeScript. Zero dependencies.

---

## What is @zugobite/monetra-core?

`@zugobite/monetra-core` is a zero-dependency TypeScript library for handling monetary values with absolute correctness. It stores all amounts as `BigInt` minor units (cents, satoshis, wei) to eliminate the floating-point errors that plague traditional `number`-based approaches.

It provides everything you need for correct money handling:

- **Money** — Immutable, integer-based monetary value object
- **Currency** — ISO 4217 registry with 60+ built-in currencies
- **Tokens** — Custom currency definitions for crypto, loyalty points, etc.
- **Allocation** — Lossless splitting of monetary values
- **Rounding** — Six explicit rounding modes with no silent precision loss
- **Formatting** — Locale-aware display and parsing
- **Conversion** — Multi-currency exchange with rate management
- **Errors** — Typed error classes with codes for programmatic handling

---

## Getting Started

1. [Installation & Setup](./getting-started.md) — Add `@zugobite/monetra-core` to your project
2. [Core Concepts](./core-concepts.md) — Understand BigInt, minor units, and rounding
3. [Guides](./guides/allocation.md) — Learn common patterns

---

## API Reference

- [Money](./core/money.md) — Core monetary value class
- [Currency](./core/currency.md) — ISO 4217 currencies and registry

---

## Guides

- [Allocation & Splitting](./guides/allocation.md)
- [Formatting & Parsing](./guides/formatting.md)
- [Custom Tokens](./guides/custom-tokens.md)
- [Error Handling](./guides/error-handling.md)

---

## Best Practices

- [Patterns & Anti-patterns](./best-practices.md)

---

## Comparison

| Challenge             | Traditional Libraries | @zugobite/monetra-core                        |
| --------------------- | --------------------- | ------------------------------------ |
| Floating-point errors | `0.1 + 0.2 !== 0.3`  | Integer-based arithmetic with BigInt |
| Currency mixing       | Silent bugs           | Type-safe currency enforcement       |
| Rounding mistakes     | Ad-hoc rounding       | Explicit rounding strategies         |
| Crypto precision      | Limited to ~15 digits | Up to 18 decimal places             |
| Multi-currency        | Manual conversion     | Built-in converter with rates        |

See the full [Library Comparison](./comparison.md).

---

## Relationship to Monetra

`@zugobite/monetra-core` is the foundation layer extracted from the [Monetra](https://github.com/zugobite/monetra) framework. Higher-level features (financial math, ledgers, double-entry bookkeeping) live in the full `monetra` package.

---

## Browser & Node.js Support

- **Node.js** 18+ (ESM and CommonJS)
- **Modern Browsers** (ES2020+)
- **TypeScript** 5.0+
- **Bundlers**: Vite, Webpack, Rollup, esbuild

---

## License

MIT © [Monetra Contributors](https://github.com/zugobite/monetra.core)

---

**Next:** [Getting Started →](./getting-started.md)
