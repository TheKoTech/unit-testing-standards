# What to Test

Not everything needs to be tested. 

Trivial code returns a value or forwards a call, with no branching and no rule. Testing trivial code does not catch regressions but adds maintenance cost.
```ts
// skip: no rule to protect
getFullName() { 
  return this.first + ' ' + this.last
}
// skip: delegation
getDiscount(order: Order) {
  return this.pricingService.getDiscount(order)
}
```

Tests are valuable when they covers decisions, calculations, rules with more than one possible outcome.
`applyDiscount(order)` with tier rates and an expiry check → test each rate, each boundary, and the expired case.

One test verifies one unit of behavior. Test that exercises a single line inside a larger function misses how that line affects the rest of it.

Cover the boundaries of a rule, not the middle of a range that always produces the same result. Test the values where behavior changes.
`Blocks orders over 100` → test order 100 (allowed) and order 101 (blocked), not order 50 and order 200 alone.

One unit of behavior - one test. Duplicate tests of one rule are redundant. Tests that check multiple units of behavior are harder to read and maintain.

Assert observable behavior only. Observable behavior is a return value or a side effect that changes state, like a write, a dispatch, or an action on a dependency. Do not assert that a getter or a read-only stub was called. That call is an implementation detail, not a behavior.
`expect(balanceStore.getBalance).toHaveBeenCalled()` → `expect(shopStore.executePurchase).toHaveBeenCalledWith(100)`

Treat the unit under test as a black box. Never read, call, or mock a private or internal member from a test. Verify private behavior only through a public return value or a call to a collaborator.
