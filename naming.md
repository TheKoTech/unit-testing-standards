# Naming

Good names state a behavior rule a non-programmer understands. Behavior does not change after refactoring.
`Calculates price and adds fee fields from the order object` → `Applies standard shipping rate for orders of medium package size`
`Handles touch event and assigns event data to field` → `Remembers last touch screen coordinates`

When writing names gather behavior context. Reference source code, find consumers if it's unclear from source. If not enough context explicitly warn user or refuse to invent behavior.

Code entities, literals, keywords and magic values break on refactor: function/variable names, return, null, true/false. Watch for subtle cases.
`stopTimer returns null if timer has not started` → `Stopping a timer that hasn't started is ignored`
`Calculate shipping fee uses order weight to determine the correct rate` → `Heavier orders pay a higher shipping fee`
`Refund amount returns to the customer's balance after order cancellation` → `Order cancellation refunds to customer's balance`
`Adds a delivery fee for orders under $50` → `Adds a delivery fee for orders below free shipping threshold`
`Password reset token expires after 1 hour` → `Password reset token expires after a delay`

Naming templates like `should...` or `[method] - [scenario] - [result]` make names less informative and longer.
`Should display a loading spinner on component init` → `Displays a loading spinner on component init`
`sendCodeType - a valid code is sent - returns true` → `Sends the computed auth response code`

Keep names concise but informative. Names must fully explain the behavior but not be redundant or vague.
`A valid user is renamed successfully when given a valid name` → `Renames user with a valid name` - simple rule, short name
`Updates balance` → `Purchasing for an order deducts funds from the user's balance` - complex rule, long name

Test name states one unit of behavior for maintainability and focused failures. Names that join behavior with "and" signals the test checks too much and should split.
`Cart is emptied and total resets after checkout` → `Cart is empty after checkout completes`, `Total resets after checkout completes`

Do not use non-standard abbreviations, they reduce clarity.
`Opening a non-existing page redirects to 404 page` → `Opening a non-existing page redirects to not found page`
`Maps addr DTO` → `Maps address DTO`

Use parameterized tests' parameters in names. Test runner will show what parameters failed.
`Validates email by the RFC 5322 standard` → `${email} follows RFC 5322 standard`

Variables in the middle of parameterized test names interrupt reading.
`Maps UTM parameter ${utmParam} to ${onelinkParam} for a given config` → `Maps UTM parameters to Onelink: ${utmParam} -> ${onelinkParam}`
`Settings of fetched ${currentOrder} are saved to persistent storage` → `${currentOrder} settings are saved to persistent storage`

Variables in names must be descriptive. Improves clarity.
`Formats raw dates to the user locale: ${input} -> ${output}` → `Formats raw dates to the user locale: ${rawDate} - ${localizedDate}`

Root `describe()` block must include tested entity. It breaks no-entity rule, but useful to find the suite.
`BillingService`, `FormComponent`, `highlightSearchMatches`

Use nested `describe()` blocks to group tests by behavior.
`Balance updates`, `Form validation`, `Protects against XSS attacks`
