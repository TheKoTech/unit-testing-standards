# What to Test

Not everything needs to be tested. Value comes from protected behavior rules. Coverage percent is irrelevant.

## Approach

Treat tested system as a black box. Test it strictly against its public interface. Testing and mocking private fields makes tests fragile to refactoring and cover less code. Therefore, cover a rule through widest public entry point that contains it to maximize test value.

## Choosing code

Cover code that decides: branches, calculations and rules with more than one outcome. Each distinct outcome needs a case.
`applyDiscount(order)` with tier rates and an expiry check - cover each rate, each boundary and the expired case.

- Business rules cost most when broken, protecting them is top priority.
- Bugs in heavily referenced code cost a lot.
- Money or data loss, wrong access rules - irreversible effects cost a lot.
- Complex computations, branches and chains of operations are easier to break.
- Rare paths, silent failures and wrong values escape manual checks.
- Test cost. A pure function costs less to cover. Heavy mock setup lowers net value.

Cover a fixed bug with a test - it means the rule had no protection.

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

## Choosing cases

Cover empty, absent and failed input when the code branches on it: null, empty list, rejected request, thrown error. Skip impossible branches, rely on compiler.

Cover boundaries of a rule, not the middle of a range that always produces the same result. Test the values where behavior changes.
`Blocks orders over 100` → cover 100 (allowed) and 101 (blocked), not 50 and 200 alone.

One unit of behavior - one test. A test of two rules fails ambiguously and costs more to maintain. Duplicate tests of one rule bring no value, so merge or delete them.

Cover a sequence of calls when the rule is about the sequence: idempotency, ordering, debounce, retry. One call proves nothing here.
`Duplicate purchase requests for the same item are ignored` → call the purchase twice, expect one request
