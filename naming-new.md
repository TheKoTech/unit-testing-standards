# Naming

Bad name explains code. Good name explains a behavior rule, a non-programmer understands it. Behavior does not change after refactoring.
`Calculates price and adds fee fields from the order object` -> `Applies standard shipping rate for orders of medium package size`
`Handles touch event and assigns event data to field` -> `Remembers last touch screen coordinates`

When writing names, gather behavior context. Reference source code, find consumers if it's unclear from source.

Code entities and literals will break on refactor. Pay attention: can be subtle.
`isLoading input renders loading spinner component` -> `Renders loading spinner`
`Calculate shipping fee uses order weight to determine the correct rate` -> `Heavier orders pay a higher shipping fee`
`Refund amount returns to the customer's balance after order cancellation` -> `Order cancellation refunds to customer's balance`

Naming templates like `should...` or `[method] - [scenario] - [result]` make names less informative and longer. Good names are free in expression.
`Should display a loading spinner on component init` -> `Displays a loading spinner on component init`
`sendCodeType - a valid code is sent - returns true` -> `Sends the computed auth response code`

Keep names concise, but informative. Names must fully explain the behavior, but not be redundant or vague. Look for balance.
`A valid user is renamed successfully when given a valid name` -> `User is renamed with a valid name` - simple rule, short name
`Updates balance` -> `Purchasing for an order deducts funds from the user's balance` - complex rule, long name

No magic values. Business logic changes will update values, but not test name strings.
`Adds a delivery fee for orders under $50` -> `Adds a delivery fee for orders below free shipping threshold`
`Password reset token expires after 1 hour` -> `Password reset token expires after a delay`

Prefer clarity over conciseness. Do not use non-standard abbreviations and shortenings.
`Opening a non-existing page redirects to 404 page` -> `Opening a non-existing page redirects to not found page`
`Maps addr DTO` -> `Maps address DTO`

Unique names within one suite are searchable and easy to find. Use parameterized tests' params in names. Test runner will show what params failed.
`Validates email by the RFC 5322 standard` -> `${email} follows RFC 5322 standard`

Variables in the middle of parameterized test names interrupt reading.
`Maps UTM parameter ${utmParam} to ${onelinkParam} for a given config` -> `Maps UTM parameters to Onelink: ${utmParam} -> ${onelinkParam}`
`Settings of fetched ${currentOrder} are saved to persistent storage` -> `${currentOrder} settings are saved to persistent storage`

Variables in names must be descriptive just like the name.
`Formats raw dates to the user locale: ${input} -> ${output}` -> `Formats raw dates to the user locale: ${rawDate} - ${localizedDate}`

Root `describe()` block must include tested entity. Technically contains a code entity, but useful to find the suite.
`BillingService`, `FormComponent`, `highlightSearchMatches`

Use nested `describe()` blocks to group tests by behavior.
`Balance updates`, `Form validation`, `Protects against XSS attacks`
