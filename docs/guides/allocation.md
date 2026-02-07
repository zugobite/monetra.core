# Allocation & Splitting Guide

When you need to divide money among multiple parties, simple division often leads to rounding issues that cause pennies to disappear. `@monetra/core`'s allocation features ensure every cent is accounted for.

---

## Table of Contents

- [The Problem](#problem)
- [Equal Splitting](#equal-splitting)
- [Ratio-Based Allocation](#ratio-allocation)
- [Practical Examples](#examples)
- [Advanced Patterns](#advanced)

---

## The Problem {#problem}

Consider splitting $100 among 3 people:

```javascript
// Naive approach - loses money!
const total = 100.0;
const share = total / 3; // 33.333333...
const rounded = Math.floor(share * 100) / 100; // 33.33

const distributed = rounded * 3; // 99.99
const lost = total - distributed; // 0.01 - WHERE DID IT GO?!
```

Over thousands of transactions, these lost cents add up to real money—and accounting errors.

### The Solution

```typescript
import { money } from "@monetra/core";

const total = money("100.00", "USD");
const shares = total.split(3);
// [$33.34, $33.33, $33.33]

// Verify: nothing lost!
const sum = shares.reduce((a, b) => a.add(b));
console.log(sum.equals(total)); // true
console.log(sum.format()); // "$100.00"
```

---

## Equal Splitting {#equal-splitting}

### Basic Usage

```typescript
import { money } from "@monetra/core";

const bill = money("87.50", "USD");

// Split among 4 people
const shares = bill.split(4);
shares.forEach((share, i) => {
  console.log(`Person ${i + 1}: ${share.format()}`);
});
// Person 1: $21.88
// Person 2: $21.88
// Person 3: $21.87
// Person 4: $21.87

// The extra cents go to the first shares
```

### How Remainders Are Distributed

When the amount doesn't divide evenly, the remainder (in minor units) is distributed one unit at a time to the first shares:

```typescript
import { money } from "@monetra/core";

// 10000 cents / 3 = 3333 remainder 1
const amount = money("100.00", "USD");
const three = amount.split(3);
// [3334 cents, 3333 cents, 3333 cents]
// [$33.34, $33.33, $33.33]

// 10000 cents / 7 = 1428 remainder 4
const seven = amount.split(7);
// First 4 get 1429 cents, last 3 get 1428 cents
// [$14.29, $14.29, $14.29, $14.29, $14.28, $14.28, $14.28]
```

---

## Ratio-Based Allocation {#ratio-allocation}

When you need to split money by specific proportions (not equally), use `allocate()`:

### Basic Usage

```typescript
import { money } from "@monetra/core";

const revenue = money("10000.00", "USD");

// Split by percentage (50%, 30%, 20%)
const shares = revenue.allocate([50, 30, 20]);
console.log(shares.map((s) => s.format()));
// ["$5,000.00", "$3,000.00", "$2,000.00"]

// Split by parts (3:2:1 ratio)
const parts = revenue.allocate([3, 2, 1]);
console.log(parts.map((s) => s.format()));
// ["$5,000.00", "$3,333.33", "$1,666.67"]
```

### Remainder Distribution

Remainders are distributed to ratios with the largest relative "rounding error":

```typescript
import { money } from "@monetra/core";

// $100 split 70/30
const amount = money("100.00", "USD");
const split = amount.allocate([70, 30]);
console.log(split.map((s) => s.format()));
// ["$70.00", "$30.00"]

// $100 split 33/33/34
const thirds = amount.allocate([33, 33, 34]);
console.log(thirds.map((s) => s.format()));
// ["$33.00", "$33.00", "$34.00"]

// Handle odd divisions
const tricky = money("100.00", "USD").allocate([1, 1, 1]);
console.log(tricky.map((s) => s.format()));
// ["$33.34", "$33.33", "$33.33"]
```

### Practical Examples

<details open>
<summary><strong>TypeScript - Commission Splits</strong></summary>

```typescript
import { money, Money } from "@monetra/core";

interface SalesRep {
  id: string;
  name: string;
  commissionRate: number; // Percentage of deal
}

interface CommissionResult {
  rep: SalesRep;
  amount: Money;
}

function calculateCommissions(
  saleAmount: string,
  team: SalesRep[]
): CommissionResult[] {
  const sale = money(saleAmount, "USD");
  const rates = team.map((rep) => rep.commissionRate);

  // Allocate based on commission rates
  const allocations = sale.allocate(rates);

  return team.map((rep, i) => ({
    rep,
    amount: allocations[i],
  }));
}

// Example: $50,000 sale split between 3 reps
const team = [
  { id: "1", name: "Senior Rep", commissionRate: 50 }, // 50%
  { id: "2", name: "Junior Rep", commissionRate: 30 }, // 30%
  { id: "3", name: "Support", commissionRate: 20 }, // 20%
];

const commissions = calculateCommissions("50000.00", team);

commissions.forEach(({ rep, amount }) => {
  console.log(`${rep.name}: ${amount.format()}`);
});
// Senior Rep: $25,000.00
// Junior Rep: $15,000.00
// Support: $10,000.00
```

</details>

<details>
<summary><strong>React.js - Investment Portfolio</strong></summary>

```tsx
import React, { useState, useMemo } from "react";
import { money, Money } from "@monetra/core";

interface Allocation {
  name: string;
  percentage: number;
}

function PortfolioAllocator() {
  const [investmentAmount, setInvestmentAmount] = useState("10000");
  const [allocations, setAllocations] = useState<Allocation[]>([
    { name: "Stocks", percentage: 60 },
    { name: "Bonds", percentage: 30 },
    { name: "Cash", percentage: 10 },
  ]);

  const calculated = useMemo(() => {
    try {
      const total = money(investmentAmount, "USD");
      const percentages = allocations.map((a) => a.percentage);
      const amounts = total.allocate(percentages);

      return allocations.map((allocation, i) => ({
        ...allocation,
        amount: amounts[i],
      }));
    } catch {
      return null;
    }
  }, [investmentAmount, allocations]);

  const totalPercentage = allocations.reduce((sum, a) => sum + a.percentage, 0);

  const updatePercentage = (index: number, value: number) => {
    const updated = [...allocations];
    updated[index].percentage = value;
    setAllocations(updated);
  };

  return (
    <div className="portfolio-allocator">
      <h2>Portfolio Allocation</h2>

      <label>
        Investment: $
        <input
          type="number"
          value={investmentAmount}
          onChange={(e) => setInvestmentAmount(e.target.value)}
        />
      </label>

      <table>
        <thead>
          <tr>
            <th>Asset Class</th>
            <th>Allocation %</th>
            <th>Amount</th>
          </tr>
        </thead>
        <tbody>
          {calculated?.map((item, i) => (
            <tr key={item.name}>
              <td>{item.name}</td>
              <td>
                <input
                  type="number"
                  min="0"
                  max="100"
                  value={item.percentage}
                  onChange={(e) =>
                    updatePercentage(i, parseInt(e.target.value) || 0)
                  }
                />
                %
              </td>
              <td>{item.amount.format()}</td>
            </tr>
          ))}
        </tbody>
        <tfoot>
          <tr>
            <td>
              <strong>Total</strong>
            </td>
            <td className={totalPercentage !== 100 ? "error" : ""}>
              {totalPercentage}%
            </td>
            <td>{money(investmentAmount, "USD").format()}</td>
          </tr>
        </tfoot>
      </table>

      {totalPercentage !== 100 && (
        <p className="warning">Warning: Allocations should total 100%</p>
      )}
    </div>
  );
}
```

</details>

<details>
<summary><strong>Node.js - Revenue Sharing</strong></summary>

```javascript
import { money, Money } from "@monetra/core";

class RevenueSharing {
  constructor(stakeholders) {
    // stakeholders: [{ id, name, sharePercent }]
    this.stakeholders = stakeholders;

    // Validate shares total 100%
    const total = stakeholders.reduce((sum, s) => sum + s.sharePercent, 0);
    if (Math.abs(total - 100) > 0.01) {
      throw new Error(`Shares must total 100%, got ${total}%`);
    }
  }

  distributeRevenue(amount) {
    const revenue = money(amount, "USD");
    const shares = this.stakeholders.map((s) => s.sharePercent);
    const allocations = revenue.allocate(shares);

    return this.stakeholders.map((stakeholder, i) => ({
      stakeholderId: stakeholder.id,
      name: stakeholder.name,
      sharePercent: stakeholder.sharePercent,
      amount: allocations[i],
      formatted: allocations[i].format(),
    }));
  }

  async processMonthlyDistribution(monthlyRevenue) {
    const distributions = this.distributeRevenue(monthlyRevenue);

    // Log and return
    console.log(`Distributing ${money(monthlyRevenue, "USD").format()}:`);
    distributions.forEach((d) => {
      console.log(`  ${d.name} (${d.sharePercent}%): ${d.formatted}`);
    });

    // Verify
    const total = distributions.reduce(
      (sum, d) => sum.add(d.amount),
      Money.zero("USD")
    );
    console.log(`  Total distributed: ${total.format()}`);

    return distributions;
  }
}

// Usage
const partnership = new RevenueSharing([
  { id: "founder1", name: "Alice (Founder)", sharePercent: 40 },
  { id: "founder2", name: "Bob (Founder)", sharePercent: 35 },
  { id: "investor", name: "Seed Fund", sharePercent: 20 },
  { id: "esop", name: "Employee Pool", sharePercent: 5 },
]);

await partnership.processMonthlyDistribution("127500.00");
// Distributing $127,500.00:
//   Alice (Founder) (40%): $51,000.00
//   Bob (Founder) (35%): $44,625.00
//   Seed Fund (20%): $25,500.00
//   Employee Pool (5%): $6,375.00
//   Total distributed: $127,500.00
```

</details>

---

## Advanced Patterns {#advanced}

### Tiered Allocation

```typescript
import { money, Money } from "@monetra/core";

interface Tier {
  threshold: Money;
  rate: number; // Commission rate for this tier
}

function calculateTieredCommission(salesAmount: Money, tiers: Tier[]): Money {
  let remaining = salesAmount;
  let commission = Money.zero(salesAmount.currency);
  let previousThreshold = Money.zero(salesAmount.currency);

  for (const tier of tiers) {
    const tierAmount = Money.min(
      remaining,
      tier.threshold.subtract(previousThreshold)
    );

    if (tierAmount.isPositive()) {
      commission = commission.add(tierAmount.percentage(tier.rate * 100));
      remaining = remaining.subtract(tierAmount);
    }

    previousThreshold = tier.threshold;
  }

  return commission;
}

// Example: Sales commission with tiers
const tiers = [
  { threshold: money("10000", "USD"), rate: 0.05 }, // 5% on first $10k
  { threshold: money("50000", "USD"), rate: 0.08 }, // 8% on $10k-$50k
  { threshold: money("100000", "USD"), rate: 0.1 }, // 10% on $50k-$100k
  { threshold: money("999999999", "USD"), rate: 0.12 }, // 12% above
];

const sales = money("75000", "USD");
const commission = calculateTieredCommission(sales, tiers);
console.log(`Commission on ${sales.format()}: ${commission.format()}`);
// Commission on $75,000.00: $5,700.00
// ($10k × 5%) + ($40k × 8%) + ($25k × 10%) = $500 + $3,200 + $2,500
```

### Weighted Allocation with Minimums

```typescript
import { money, Money } from "@monetra/core";

interface Recipient {
  id: string;
  weight: number;
  minimum?: Money;
}

function allocateWithMinimums(
  total: Money,
  recipients: Recipient[]
): Map<string, Money> {
  const results = new Map<string, Money>();
  let remaining = total;

  // First pass: Allocate minimums
  for (const recipient of recipients) {
    if (recipient.minimum) {
      const allocation = Money.min(recipient.minimum, remaining);
      results.set(recipient.id, allocation);
      remaining = remaining.subtract(allocation);
    } else {
      results.set(recipient.id, Money.zero(total.currency));
    }
  }

  // Second pass: Allocate remainder by weight
  if (remaining.isPositive()) {
    const totalWeight = recipients.reduce((sum, r) => sum + r.weight, 0);
    const weights = recipients.map((r) => r.weight);
    const extraAllocations = remaining.allocate(weights);

    recipients.forEach((recipient, i) => {
      const current = results.get(recipient.id)!;
      results.set(recipient.id, current.add(extraAllocations[i]));
    });
  }

  return results;
}

// Example
const budget = money("10000", "USD");
const departments = [
  { id: "marketing", weight: 40, minimum: money("2000", "USD") },
  { id: "engineering", weight: 35, minimum: money("3000", "USD") },
  { id: "operations", weight: 25 },
];

const allocations = allocateWithMinimums(budget, departments);
allocations.forEach((amount, id) => {
  console.log(`${id}: ${amount.format()}`);
});
```

### Fractional Shares with Rounding Pools

```typescript
import { money, Money, RoundingMode } from "@monetra/core";

/**
 * Allocates with a "rounding pool" to handle accumulated errors
 */
function allocateWithRoundingPool(
  total: Money,
  shares: number[] // Can be fractional
): { allocations: Money[]; roundingAdjustment: Money } {
  const sum = shares.reduce((a, b) => a + b, 0);

  // Calculate exact allocations (may have fractional cents)
  const exactAllocations = shares.map((share) => {
    const ratio = share / sum;
    return total.multiply(ratio, { rounding: RoundingMode.FLOOR });
  });

  // Calculate the rounding pool (unallocated amount)
  const allocated = exactAllocations.reduce(
    (sum, a) => sum.add(a),
    Money.zero(total.currency)
  );
  const roundingPool = total.subtract(allocated);

  // Distribute rounding pool to the largest allocations
  const poolCents = Number(roundingPool.minor);
  const sortedIndices = shares
    .map((_, i) => i)
    .sort((a, b) => shares[b] - shares[a]);

  for (let i = 0; i < poolCents; i++) {
    const idx = sortedIndices[i % sortedIndices.length];
    exactAllocations[idx] = exactAllocations[idx].add(1); // Add 1 cent
  }

  return {
    allocations: exactAllocations,
    roundingAdjustment: roundingPool,
  };
}
```

---

## Summary

| Method            | Use Case              | Remainder Handling             |
| ----------------- | --------------------- | ------------------------------ |
| `split(n)`        | Equal division        | First shares get extra cents   |
| `allocate([...])` | Proportional division | Largest ratios get extra cents |

### Best Practices

1. **Always verify totals**: After splitting, ensure the sum equals the original
2. **Document your rounding policy**: Decide who gets remainders and stick to it
3. **Use allocate() for percentages**: Even if equal, `allocate([33.33, 33.33, 33.34])` is clearer
4. **Handle edge cases**: What if splitting among 0 people? Validate inputs!

---

## Next Steps

- **[Formatting Guide](./formatting.md)** - Display money values
- **[Error Handling](./error-handling.md)** - Handle allocation errors
- **[Best Practices](../best-practices.md)** - Production patterns
