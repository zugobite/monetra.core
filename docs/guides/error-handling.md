# Error Handling Guide

`@zugobite/monetra-core` provides typed, descriptive errors to help you handle edge cases gracefully. All errors extend `MonetraError` and include error codes for programmatic handling.

---

## Table of Contents

- [Error Types](#error-types)
- [Error Codes](#error-codes)
- [Handling Errors](#handling-errors)
- [Validation Strategies](#validation-strategies)
- [Best Practices](#best-practices)

---

## Error Types {#error-types}

### MonetraError (Base)

All @zugobite/monetra-core errors extend this base class:

```typescript
import { MonetraError, MonetraErrorCode } from "@zugobite/monetra-core";

try {
  // Some operation
} catch (error) {
  if (error instanceof MonetraError) {
    console.log(error.name); // Error class name
    console.log(error.message); // Descriptive message
    console.log(error.code); // MonetraErrorCode for programmatic handling
  }
}
```

### CurrencyMismatchError

Thrown when performing operations on Money objects with different currencies:

```typescript
import { money, CurrencyMismatchError } from "@zugobite/monetra-core";

const usd = money("100.00", "USD");
const eur = money("50.00", "EUR");

try {
  usd.add(eur); // Error: Cannot add different currencies
} catch (error) {
  if (error instanceof CurrencyMismatchError) {
    console.log(error.expected); // "USD"
    console.log(error.received); // "EUR"
    console.log(error.code); // "MONETRA_CURRENCY_MISMATCH"

    // Error message includes helpful tip:
    // "Currency mismatch: expected USD, received EUR.
    //  Tip: Use a Converter to convert between currencies..."
  }
}
```

**When it occurs:**

- Adding/subtracting Money with different currencies
- Comparing Money with different currencies
- Ledger operations with mismatched currencies

**Solution:**

```typescript
import { money, Converter } from "@zugobite/monetra-core";

const usd = money("100.00", "USD");
const eur = money("50.00", "EUR");

// Convert to same currency first
const converter = new Converter("USD", { EUR: 0.92 });
const eurInUsd = converter.convert(eur, "USD");

const total = usd.add(eurInUsd); // Now works
```

---

### RoundingRequiredError

Thrown when an operation produces a non-integer result but no rounding mode is specified:

```typescript
import { money, RoundingRequiredError, RoundingMode } from "@zugobite/monetra-core";

const price = money("100.00", "USD");

try {
  price.divide(3); // Error: 100 / 3 = 33.333... requires rounding
} catch (error) {
  if (error instanceof RoundingRequiredError) {
    console.log(error.code); // "MONETRA_ROUNDING_REQUIRED"

    // Error message includes helpful tip:
    // "Rounding required for divide: result 33.33333... is not an integer.
    //  Tip: Provide a rounding mode:
    //     money.divide(value, { rounding: RoundingMode.HALF_UP })"
  }
}
```

**When it occurs:**

- Division that doesn't result in exact integers
- Percentage calculations
- Currency conversions

**Solution:**

```typescript
import { money, RoundingMode } from "@zugobite/monetra-core";

const price = money("100.00", "USD");

// Provide a rounding mode
const third = price.divide(3, { rounding: RoundingMode.HALF_UP });
console.log(third.format()); // "$33.33"

// Or use allocate() for lossless division
const parts = price.allocate([1, 1, 1]);
console.log(parts.map((p) => p.format()));
// ["$33.34", "$33.33", "$33.33"]  ← No cents lost!
```

---

### InvalidPrecisionError

Thrown when a value exceeds the currency's allowed decimal places:

```typescript
import { money, InvalidPrecisionError } from "@zugobite/monetra-core";

try {
  // USD has 2 decimal places
  money("100.001", "USD"); // Error: Too many decimals
} catch (error) {
  if (error instanceof InvalidPrecisionError) {
    console.log(error.code); // "MONETRA_INVALID_PRECISION"
    console.log(error.message);
  }
}
```

**When it occurs:**

- Creating Money with more decimals than the currency allows
- Parsing user input with invalid precision

**Solution:**

```typescript
import { money, RoundingMode } from "@zugobite/monetra-core";

// Option 1: Round the input
const amount = money("100.001", "USD", { rounding: RoundingMode.HALF_UP });
console.log(amount.format()); // "$100.00"

// Option 2: Validate input before creating
function validateDecimalPlaces(value: string, maxDecimals: number): boolean {
  const parts = value.split(".");
  return parts.length === 1 || parts[1].length <= maxDecimals;
}

if (validateDecimalPlaces("100.001", 2)) {
  // Safe to create
}
```

---

### InsufficientFundsError

Thrown when an operation would result in a negative balance:

```typescript
import { money, InsufficientFundsError } from "@zugobite/monetra-core";

try {
  // Application-level balance check
  const balance = money("50.00", "USD");
  const withdrawal = money("100.00", "USD");

  if (balance.lessThan(withdrawal)) {
    throw new InsufficientFundsError("Not enough funds for withdrawal");
  }
} catch (error) {
  if (error instanceof InsufficientFundsError) {
    console.log(error.code); // "MONETRA_INSUFFICIENT_FUNDS"
    console.log(error.message);
  }
}
```

**When it occurs:**

- Withdrawal exceeds available balance
- Transfer amount exceeds source balance

**Solution:**

```typescript
import { money, Money } from "@zugobite/monetra-core";

const balance = money("50.00", "USD");
const withdrawal = money("100.00", "USD");

// Check before attempting
if (balance.greaterThanOrEqual(withdrawal)) {
  const remaining = balance.subtract(withdrawal);
} else {
  console.log("Not enough funds");
}
```

---

### OverflowError

Thrown when arithmetic exceeds safe integer bounds:

```typescript
import { money, OverflowError } from "@zugobite/monetra-core";

try {
  const huge = money("999999999999999999999999", "USD");
  huge.multiply(999999999999); // Error: Result too large
} catch (error) {
  if (error instanceof OverflowError) {
    console.log(error.code); // "MONETRA_OVERFLOW"
    console.log(error.message);
  }
}
```

**When it occurs:**

- Multiplication of very large amounts
- Exponentiation with large values
- Repeated operations accumulating large results

**Solution:**

```typescript
// Break operations into smaller steps
// or use reasonable application limits
const MAX_AMOUNT = 1_000_000_000_00n; // $1 billion in cents

function safeMultiply(m: Money, factor: number): Money | null {
  const result = m.multiply(factor);
  if (result.minor > MAX_AMOUNT) {
    return null; // Handle overflow case
  }
  return result;
}
```

---

## Error Codes {#error-codes}

Use error codes for programmatic error handling:

```typescript
import { MonetraError, MonetraErrorCode } from "@zugobite/monetra-core";

try {
  // Some operation
} catch (error) {
  if (error instanceof MonetraError) {
    switch (error.code) {
      case MonetraErrorCode.CURRENCY_MISMATCH:
        // Handle currency mismatch
        break;
      case MonetraErrorCode.ROUNDING_REQUIRED:
        // Handle rounding requirement
        break;
      case MonetraErrorCode.INVALID_PRECISION:
        // Handle precision error
        break;
      case MonetraErrorCode.INSUFFICIENT_FUNDS:
        // Handle insufficient funds
        break;
      case MonetraErrorCode.OVERFLOW:
        // Handle overflow
        break;
    }
  }
}
```

### Error Code Reference

| Code                         | Constant                              | Description                        |
| ---------------------------- | ------------------------------------- | ---------------------------------- |
| `MONETRA_CURRENCY_MISMATCH`  | `MonetraErrorCode.CURRENCY_MISMATCH`  | Operations on different currencies |
| `MONETRA_ROUNDING_REQUIRED`  | `MonetraErrorCode.ROUNDING_REQUIRED`  | Division requires rounding mode    |
| `MONETRA_INVALID_PRECISION`  | `MonetraErrorCode.INVALID_PRECISION`  | Too many decimal places            |
| `MONETRA_INSUFFICIENT_FUNDS` | `MonetraErrorCode.INSUFFICIENT_FUNDS` | Not enough balance                 |
| `MONETRA_OVERFLOW`           | `MonetraErrorCode.OVERFLOW`           | Arithmetic overflow                |

---

## Handling Errors {#handling-errors}

### Type-Safe Error Handling

```typescript
import {
  money,
  MonetraError,
  CurrencyMismatchError,
  RoundingRequiredError,
  InvalidPrecisionError,
  InsufficientFundsError,
  OverflowError,
  RoundingMode,
} from "@zugobite/monetra-core";

function performCalculation(amount: string, currency: string) {
  try {
    const m = money(amount, currency);
    return m.divide(3, { rounding: RoundingMode.HALF_UP });
  } catch (error) {
    if (error instanceof InvalidPrecisionError) {
      return { error: "Invalid amount format", type: "validation" };
    }
    if (error instanceof RoundingRequiredError) {
      return { error: "Calculation error", type: "internal" };
    }
    if (error instanceof MonetraError) {
      return { error: error.message, type: "money" };
    }
    throw error; // Re-throw unexpected errors
  }
}
```

### Result Pattern

```typescript
import { money, Money, MonetraError, RoundingMode } from "@zugobite/monetra-core";

type Result<T> =
  | { success: true; value: T }
  | { success: false; error: string; code: string };

function safeMoney(amount: string, currency: string): Result<Money> {
  try {
    return {
      success: true,
      value: money(amount, currency),
    };
  } catch (error) {
    if (error instanceof MonetraError) {
      return {
        success: false,
        error: error.message,
        code: error.code,
      };
    }
    return {
      success: false,
      error: "Unknown error",
      code: "UNKNOWN",
    };
  }
}

// Usage
const result = safeMoney("100.50", "USD");
if (result.success) {
  console.log(result.value.format());
} else {
  console.error(result.error);
}
```

---

## Validation Strategies {#validation-strategies}

### Input Validation

```typescript
import { getCurrency, money, MonetraError, RoundingMode } from "@zugobite/monetra-core";

interface ValidationResult {
  valid: boolean;
  error?: string;
  money?: Money;
}

function validateMoneyInput(
  amount: string,
  currencyCode: string
): ValidationResult {
  // Check currency exists
  let currency;
  try {
    currency = getCurrency(currencyCode);
  } catch {
    return { valid: false, error: `Unknown currency: ${currencyCode}` };
  }

  // Check amount format
  if (!/^-?\d+(\.\d+)?$/.test(amount)) {
    return { valid: false, error: "Invalid number format" };
  }

  // Check decimal places
  const parts = amount.split(".");
  if (parts[1] && parts[1].length > currency.decimals) {
    return {
      valid: false,
      error: `${currencyCode} only allows ${currency.decimals} decimal places`,
    };
  }

  // Check for negative (if not allowed)
  if (parseFloat(amount) < 0) {
    return { valid: false, error: "Amount cannot be negative" };
  }

  // Create Money object
  try {
    const m = money(amount, currency);
    return { valid: true, money: m };
  } catch (error) {
    if (error instanceof MonetraError) {
      return { valid: false, error: error.message };
    }
    throw error;
  }
}
```

## Best Practices {#best-practices}

### 1. Always Catch Specific Errors

```typescript
import {
  money,
  CurrencyMismatchError,
  RoundingRequiredError,
  InvalidPrecisionError,
  MonetraError,
} from "@zugobite/monetra-core";

try {
  // Operation
} catch (error) {
  // Handle specific errors first
  if (error instanceof CurrencyMismatchError) {
    // Handle currency mismatch
  } else if (error instanceof RoundingRequiredError) {
    // Handle rounding requirement
  } else if (error instanceof InvalidPrecisionError) {
    // Handle precision error
  } else if (error instanceof MonetraError) {
    // Handle any other @zugobite/monetra-core error
  } else {
    throw error; // Re-throw unexpected errors
  }
}
```

### 2. Validate Before Operating

```typescript
import { money, getCurrency, Money } from "@zugobite/monetra-core";

// Validate currency exists
function isValidCurrency(code: string): boolean {
  try {
    getCurrency(code);
    return true;
  } catch {
    return false;
  }
}

// Validate balance before operation
function canAfford(balance: Money, cost: Money): boolean {
  return balance.greaterThanOrEqual(cost);
}
```

### 3. Use Error Codes in Logs

```typescript
import { MonetraError } from "@zugobite/monetra-core";

function logMoneyError(error: MonetraError, context: object) {
  console.error({
    errorCode: error.code,
    errorType: error.name,
    message: error.message,
    timestamp: new Date().toISOString(),
    ...context,
  });
}
```

### 4. Provide User-Friendly Messages

```typescript
import { MonetraError, MonetraErrorCode } from "@zugobite/monetra-core";

const userMessages: Record<MonetraErrorCode, string> = {
  [MonetraErrorCode.CURRENCY_MISMATCH]: "Please select matching currencies",
  [MonetraErrorCode.ROUNDING_REQUIRED]: "Unable to calculate exact amount",
  [MonetraErrorCode.INVALID_PRECISION]: "Please enter a valid amount",
  [MonetraErrorCode.INSUFFICIENT_FUNDS]:
    "Not enough balance for this transaction",
  [MonetraErrorCode.OVERFLOW]: "Amount is too large to process",
};

function getUserMessage(error: MonetraError): string {
  return userMessages[error.code as MonetraErrorCode] ?? "An error occurred";
}
```

---

## Next Steps

- **[Best Practices](../best-practices.md)** — Patterns and anti-patterns
- **[Money API](../core/money.md)** — Full Money API reference
- **[Allocation Guide](./allocation.md)** — Avoid rounding errors
