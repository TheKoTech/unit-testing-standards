# What to Test

Not everything needs to be tested. Value comes from protected behavior rules, not from coverage percent.

Cover code that decides: branches, calculations and rules with more than one outcome. Each distinct outcome needs a case.
`applyDiscount(order)` with tier rates and an expiry check → cover each rate, each boundary and the expired case.

Cover business rules first. They cost most when broken, protecting them is most valuable. Rest judge by:
- References. Bugs in heavily referenced code deal more damage.
- Complexity. Branches and computation can fail in more places.
- Failure cost. Money, data loss, access rules and other irreversible effects cost most.
- Detection cost. Rare paths, silent failures and wrong values escape manual checks.
- Test cost. A pure function costs one call to cover. Heavy mock setup lowers net value.

Cover a rule through widest public entry point that contains it. A wider test catches more regressions. A test that exercises one line inside a larger function misses how that line affects the rest.

Cover a fixed bug with a test - rule had no protection.

Skip trivial code. Trivial code returns a value or forwards a call, with no branch and no rule. A test for it catches no regression and adds maintenance cost.
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

Skip external dependencies. Framework, library and language authors test their own code.
`Injected service is available in the component` → skip, the framework guarantees it

## Which cases to cover

Cover the boundaries of a rule, not the middle of a range that always produces the same result. Test the values where behavior changes.
`Blocks orders over 100` → cover 100 (allowed) and 101 (blocked), not 50 and 200 alone.

Cover empty, absent and failed input when the code branches on it: null, empty list, rejected request, thrown error. Skip impossible branches.

Cover a sequence of calls when the rule is about the sequence: idempotency, ordering, debounce, retry. One call proves nothing here.
`Duplicate purchase requests for the same item are ignored` → call the purchase twice, expect one request

One unit of behavior - one test. A test of two rules fails ambiguously and costs more to maintain. Duplicate tests of one rule are redundant, so merge or delete them.

## What to assert

Assert observable behavior only: a return value, or a side effect that changes state, like a write, a dispatch, an action on a dependency or UI updates. A call to a getter or a read-only stub is an implementation detail, not behavior. Do not assert it.
x `expect(balanceStore.getBalance).toHaveBeenCalled()`
+ `expect(shopStore.executePurchase).toHaveBeenCalledWith(100)`

Assert the value, not the fact of a call. A loose assertion still passes after a regression.
`expect(store.dispatch).toHaveBeenCalledOnce()` → `expect(store.dispatch).toHaveBeenCalledOnceWith(new UserLoaded(user))`

Treat the tested system as a black box. Never read, call or mock a private or internal member from a test. Verify private behavior only through a public return value or a call to a collaborator.
