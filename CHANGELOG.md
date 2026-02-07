# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-02-07

### Added

This is the initial release of `monetra-core`, extracted from the full [Monetra](https://github.com/zugobite/monetra) framework (v2.3.0). It contains only the correctness-focused foundation layer.

#### Core Money (`Money` class)

- Immutable, BigInt-based monetary value object
- `Money.fromMinor()`, `Money.fromMajor()`, `Money.fromCents()`, `Money.fromDecimal()`, `Money.fromFloat()`
- Arithmetic: `add`, `subtract`, `multiply`, `divide`, `abs`, `negate`, `clamp`
- Comparisons: `equals`, `greaterThan`, `lessThan`, `greaterThanOrEqual`, `lessThanOrEqual`, `compare`
- Financial helpers: `percentage`, `addPercent`, `subtractPercent`, `split`
- Static helpers: `Money.min()`, `Money.max()`, `Money.zero()`
- Serialization: `toJSON()`, `Money.reviver`, `toDecimalString()`
- `money()` convenience factory function

#### Currency & Registry

- ISO 4217 currency support with 60+ currencies
- Automatic decimal precision handling (0–18 decimal places)
- `Currency` metadata objects with code, name, symbol, and decimals
- Global currency registry with `getCurrency()` lookups

#### Custom Tokens

- `defineToken()` factory for custom currencies and crypto tokens
- High-precision support up to 18 decimals (ETH, BTC, USDC, USDT presets)
- Custom symbol support in formatting
- Token metadata: `chainId`, `contractAddress`, `standard`, `coingeckoId`

#### Rounding

- Six explicit rounding modes: `HALF_UP`, `HALF_DOWN`, `HALF_EVEN`, `FLOOR`, `CEIL`, `TRUNCATE`
- Required on lossy operations — no silent precision loss

#### Allocation

- Deterministic splitting across any number of ratios
- Lossless remainder distribution (total always equals original)
- GAAP-compliant splitting algorithms

#### Formatting & Parsing

- Locale-aware formatting via `Intl.NumberFormat`
- Accounting format support (parenthesised negatives)
- `parseLocaleString()` and `parseLocaleToMinor()` for parsing

#### Multi-Currency

- `Converter` class with exchange rate management
- Historical rate support via `addHistoricalRate()`
- `MoneyBag` for aggregating values across multiple currencies
- Type-safe currency mismatch prevention

#### Error Handling

- `CurrencyMismatchError` with actionable tips
- `InvalidArgumentError`, `OverflowError`, `RoundingRequiredError`
- `InsufficientFundsError`, `InvalidPrecisionError`
- Error codes on all error classes (`MonetraErrorCode` enum)

### Removed (vs. Monetra v2.3.0)

The following modules from the full Monetra framework are **not included** in `monetra-core`:

- **Financial Mathematics** — `pmt`, `npv`, `irr`, `futureValue`, `presentValue`, loan amortisation, compound/simple interest, depreciation, leverage ratios, ROI, `Rate` class
- **Ledger System** — `Ledger`, `DoubleEntryLedger`, `Account`, `JournalEntry`, chart-of-accounts templates, SHA-256 hash chain verification
- **Benchmarks** — micro and stress benchmark suites
- **Mutation Testing** — Stryker configuration and reports

These features remain available in the full [`monetra`](https://github.com/zugobite/monetra) package.
