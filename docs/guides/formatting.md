# Formatting & Parsing Guide

`@monetra/core` provides flexible formatting options for displaying monetary values and robust parsing for converting user input back to Money objects.

---

## Table of Contents

- [Formatting Basics](#formatting-basics)
- [Locale Support](#locale-support)
- [Display Options](#display-options)
- [Parsing Input](#parsing)
- [Serialization](#serialization)

---

## Formatting Basics {#formatting-basics}

Every `Money` object can be formatted using the `format()` method:

```typescript
import { money } from "@monetra/core";

const amount = money("1234567.89", "USD");

// Default formatting
console.log(amount.format()); // "$1,234,567.89"

// Currency determines default format
const yen = money("123456", "JPY");
console.log(yen.format()); // "¥123,456"

const euro = money("1234.56", "EUR");
console.log(euro.format()); // "€1,234.56"
```

### Format Signature

```typescript
format(options?: {
  locale?: string;           // BCP 47 locale (e.g., "en-US", "de-DE")
  symbol?: boolean;          // Include currency symbol (default: true)
  display?: 'symbol' | 'code' | 'name';  // How to show currency
}): string
```

---

## Locale Support {#locale-support}

Format money according to different regional conventions:

```typescript
import { money } from "@monetra/core";

const amount = money("1234567.89", "USD");

// Different locales
console.log(amount.format({ locale: "en-US" })); // "$1,234,567.89"
console.log(amount.format({ locale: "en-GB" })); // "US$1,234,567.89"
console.log(amount.format({ locale: "de-DE" })); // "1.234.567,89 $"
console.log(amount.format({ locale: "fr-FR" })); // "1 234 567,89 $US"
console.log(amount.format({ locale: "ja-JP" })); // "$1,234,567.89"
console.log(amount.format({ locale: "ar-SA" })); // "١٬٢٣٤٬٥٦٧٫٨٩ US$"

// Euro with different locales
const euro = money("1234.56", "EUR");
console.log(euro.format({ locale: "de-DE" })); // "1.234,56 €"
console.log(euro.format({ locale: "fr-FR" })); // "1 234,56 €"
console.log(euro.format({ locale: "en-IE" })); // "€1,234.56"
```

### Common Locale Patterns

| Locale | Number Format | Example      |
| ------ | ------------- | ------------ |
| en-US  | 1,234.56      | $1,234.56    |
| de-DE  | 1.234,56      | 1.234,56 €   |
| fr-FR  | 1 234,56      | 1 234,56 €   |
| ja-JP  | 1,234         | ¥1,234       |
| ar-SA  | ١٬٢٣٤         | ١٬٢٣٤ ر.س    |
| hi-IN  | 1,23,456      | ₹1,23,456.00 |
| pt-BR  | 1.234,56      | R$ 1.234,56  |

---

## Display Options {#display-options}

### Currency Display

Control how the currency identifier appears:

```typescript
import { money } from "@monetra/core";

const amount = money("99.99", "USD");

// Symbol (default)
console.log(amount.format({ display: "symbol" }));
// "$99.99"

// ISO code
console.log(amount.format({ display: "code" }));
// "USD 99.99"

// Full name
console.log(amount.format({ display: "name" }));
// "99.99 US dollars"
```

### Without Symbol

```typescript
import { money } from "@monetra/core";

const amount = money("1234.56", "USD");

// No symbol - just the number
console.log(amount.format({ symbol: false }));
// "1,234.56"

// Useful for tabular data
const prices = [
  money("29.99", "USD"),
  money("149.99", "USD"),
  money("9.99", "USD"),
];

console.log("USD");
prices.forEach((p) => {
  console.log(p.format({ symbol: false }).padStart(10));
});
// USD
//      29.99
//     149.99
//       9.99
```

### Combining Options

```typescript
import { money } from "@monetra/core";

const amount = money("1234.56", "EUR");

// German format with code
console.log(
  amount.format({
    locale: "de-DE",
    display: "code",
  })
);
// "1.234,56 EUR"

// French format with name
console.log(
  amount.format({
    locale: "fr-FR",
    display: "name",
  })
);
// "1 234,56 euros"
```

---

## Parsing Input {#parsing}

### From Strings (Recommended)

The safest way to parse user input:

```typescript
import { money, Money } from "@monetra/core";

// Parse from string (major units)
const a = money("10.50", "USD"); // $10.50
const b = money("1,234.56", "USD"); // Note: May not work with commas

// Use Money.fromMajor for explicit parsing
const c = Money.fromMajor("10.50", "USD"); // $10.50

// Clean user input before parsing
function parseUserInput(input: string, currency: string): Money {
  // Remove currency symbols, spaces, and thousand separators
  const cleaned = input
    .replace(/[$€£¥]/g, "")
    .replace(/\s/g, "")
    .replace(/,/g, ""); // Assumes US format - be careful!

  return Money.fromMajor(cleaned, currency);
}

console.log(parseUserInput("$1,234.56", "USD").format()); // "$1,234.56"
console.log(parseUserInput("€ 999,99", "EUR").format()); // Note: Careful with EU format!
```

### Locale-Aware Parsing

```typescript
import { money, Money } from "@monetra/core";

/**
 * Parse money from a localized string
 */
function parseLocalized(
  input: string,
  currency: string,
  locale: string
): Money {
  // Get locale's number format
  const formatter = new Intl.NumberFormat(locale);
  const parts = formatter.formatToParts(1234.56);

  const groupSep = parts.find((p) => p.type === "group")?.value || ",";
  const decimalSep = parts.find((p) => p.type === "decimal")?.value || ".";

  // Clean input based on locale
  let cleaned = input
    .replace(/[^\d.,\-]/g, "") // Keep only digits, dots, commas, minus
    .replace(new RegExp(`\\${groupSep}`, "g"), ""); // Remove thousand separators

  // Normalize decimal separator
  if (decimalSep !== ".") {
    cleaned = cleaned.replace(decimalSep, ".");
  }

  return Money.fromMajor(cleaned, currency);
}

// US format
console.log(parseLocalized("$1,234.56", "USD", "en-US").format());
// "$1,234.56"

// German format
console.log(parseLocalized("1.234,56 €", "EUR", "de-DE").format());
// "€1,234.56"
```

### From Form Inputs

```typescript
import { Money, money } from "@monetra/core";

interface MoneyInputResult {
  money: Money | null;
  error: string | null;
}

function validateMoneyInput(
  value: string,
  currency: string,
  options?: {
    min?: Money;
    max?: Money;
    required?: boolean;
  }
): MoneyInputResult {
  // Handle empty
  if (!value || value.trim() === "") {
    if (options?.required) {
      return { money: null, error: "Amount is required" };
    }
    return { money: Money.zero(currency), error: null };
  }

  try {
    // Clean and parse
    const cleaned = value.replace(/[^\d.\-]/g, "");
    const parsed = Money.fromMajor(cleaned, currency);

    // Validate range
    if (options?.min && parsed.lessThan(options.min)) {
      return {
        money: null,
        error: `Minimum amount is ${options.min.format()}`,
      };
    }

    if (options?.max && parsed.greaterThan(options.max)) {
      return {
        money: null,
        error: `Maximum amount is ${options.max.format()}`,
      };
    }

    return { money: parsed, error: null };
  } catch (e) {
    return { money: null, error: "Invalid amount format" };
  }
}

// Usage
const result = validateMoneyInput("1234.56", "USD", {
  min: money("1.00", "USD"),
  max: money("10000.00", "USD"),
  required: true,
});

if (result.error) {
  console.log("Error:", result.error);
} else {
  console.log("Parsed:", result.money?.format());
}
```

---

## Serialization {#serialization}

### JSON Serialization

```typescript
import { money, Money } from "@monetra/core";

const amount = money("99.99", "USD");

// Serialize
const json = amount.toJSON();
console.log(json);
// { amount: "9999", currency: "USD", precision: 2 }

const jsonString = JSON.stringify(amount);
console.log(jsonString);
// '{"amount":"9999","currency":"USD","precision":2}'

// Deserialize
function deserializeMoney(json: string | object): Money {
  const data = typeof json === "string" ? JSON.parse(json) : json;
  return Money.fromMinor(BigInt(data.amount), data.currency);
}

const restored = deserializeMoney(jsonString);
console.log(restored.format()); // "$99.99"
```

### Database Storage

```typescript
import { Money, money } from "@monetra/core";

// Option 1: Store as minor units (integer) + currency code
interface MoneyRecord {
  amount_minor: bigint; // or string for databases without BigInt
  currency_code: string;
}

function toRecord(m: Money): MoneyRecord {
  return {
    amount_minor: m.minor,
    currency_code: m.currency.code,
  };
}

function fromRecord(record: MoneyRecord): Money {
  return Money.fromMinor(record.amount_minor, record.currency_code);
}

// Option 2: Store as decimal string + currency
interface MoneyStringRecord {
  amount: string; // "123.45"
  currency: string; // "USD"
}

function toStringRecord(m: Money): MoneyStringRecord {
  return {
    amount: m.format({ symbol: false }).replace(/,/g, ""),
    currency: m.currency.code,
  };
}

function fromStringRecord(record: MoneyStringRecord): Money {
  return Money.fromMajor(record.amount, record.currency);
}
```

---

## Custom Formatters

### Compact Notation

```typescript
import { Money, money } from "@monetra/core";

function formatCompact(m: Money): string {
  const value = Number(m.minor) / 10 ** m.currency.decimals;

  if (Math.abs(value) >= 1_000_000_000) {
    return `${m.currency.symbol}${(value / 1_000_000_000).toFixed(1)}B`;
  }
  if (Math.abs(value) >= 1_000_000) {
    return `${m.currency.symbol}${(value / 1_000_000).toFixed(1)}M`;
  }
  if (Math.abs(value) >= 1_000) {
    return `${m.currency.symbol}${(value / 1_000).toFixed(1)}K`;
  }

  return m.format();
}

console.log(formatCompact(money("1234567890", "USD"))); // "$1.2B"
console.log(formatCompact(money("5432100", "USD"))); // "$5.4M"
console.log(formatCompact(money("42500", "USD"))); // "$42.5K"
console.log(formatCompact(money("999", "USD"))); // "$999.00"
```

### Accounting Format

```typescript
import { Money, money } from "@monetra/core";

function formatAccounting(m: Money): string {
  if (m.isNegative()) {
    return `(${m.abs().format()})`;
  }
  return m.format();
}

console.log(formatAccounting(money("1234.56", "USD"))); // "$1,234.56"
console.log(formatAccounting(money("-1234.56", "USD"))); // "($1,234.56)"
```

---

## Best Practices

1. **Store amounts as minor units (integers)** - More precise than decimals
2. **Store currency code with every amount** - Never assume currency
3. **Use locale for display, not storage** - Formatting is a view concern
4. **Validate on input** - Reject invalid formats early
5. **Use `toJSON()` for serialization** - Consistent format across your app

---

## Next Steps

- **[Allocation Guide](./allocation.md)** - Splitting money
- **[Custom Tokens](./custom-tokens.md)** - Define custom currencies
- **[API Reference](../api/money.md)** - Full Money API
