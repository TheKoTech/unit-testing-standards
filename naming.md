# Test naming

Names describe behavior, not code. A non-programmer must understand the name.

## Describe blocks

Only the root `describe()` may name a code entity. Nested blocks group by behavior and obey the rules below.

root: `BillingService`, `FormComponent`, `mapUtmParamsToOnelink` nested: `Balance updates`, `Form validation`, `Maps parameters`

## Reject a name for any of these

- `template` — any fixed opening formula: `should ...`, `Test that ...`, `[method] - [scenario] - [result]`.
- `code` — a method, type, variable, or argument name. Refactoring does not update name strings.
- `literal` — a keyword or raw value: `return`, `throw`, `true`, `false`, `null`, `undefined`. Inflections count.
- `magic` — a number the reader must convert or verify elsewhere: `3600 seconds`, `$50`, `over 100`. Keep a number the reader grasps at once: `404`, `weekend`, `third attempt`.
- `compiled` — a `${param}` in the middle of the name. Put parameters first or last, never inside the sentence.
- `vague` — too short to identify the behavior or to find by search. Parameter names count: `${rawDate}`, not `${input}`.
- `verbose` — empty words such as `correctly` or `successfully`. Also a repeated condition.
- `duplicate` — a name that repeats another name in the same file. Cases of one parameterized test are exempt.

Exception: utility code without business logic. Use technical terms. Rule `code` still applies.

Read the source under test before you name or rename it. A `vague` name lacks the facts for a rewrite. Never invent a domain fact.

## Examples

✓ `User is renamed with a valid name`
✗ `renameUser should return true when name is valid` — template, code, literal
✗ `A valid user is renamed successfully when given a valid name` — verbose

✓ `Price is unavailable for a trip without a route`
✗ `should return false when tariff has no routeMeta` — template, literal, code

✓ `Maps UTM parameters to Onelink: ${utmParam} -> ${onelinkParam}` — utility code, technical terms allowed
✗ `mapUtmParams() correctly maps utmParams arg based on UtmMapperConfig` — code, verbose
✗ `Maps UTM parameter ${a} to ${b} for a given config` — compiled, vague
✗ `Maps UTM parameters` — vague

✓ `Applies standard shipping rate below the free-shipping threshold: ${amount}`
✗ `Applies standard shipping rate for orders under $50: ${amount}` — magic

✓ `${address} is a valid email` — a leading parameter keeps the sentence readable
✗ `isValidEmail returns true for ${address}` — code, literal

✓ `Destination address is visible for multi-destination orders` — domain logic earns a longer name
