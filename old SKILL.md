---
name: unit-testing-standards
description: Unit Testing Standards
disable-model-invocation: true
# tokens: 5595, chars: 21625
---

# Unit Testing Standards

## Core Philosophy

The goal of testing is to prevent regressions and project entropy. Tests must ensure stable growth by protecting business logic first. High-quality tests prioritize refactoring resistance — they should not break when the internal implementation changes, only when the business rule is violated.

## AAA

FOLLOW the Arrange-Act-Assert pattern.

- Arrange: Use factories for mocks and data. Never wrap the entire test logic (the lifecycle) in a factory; keep the setup visible to maintain readability.
- Act: Exactly one action per unit test. If a test requires multiple actions to achieve a result, the Public API is likely poorly designed (leaking implementation details).
  - EXCEPTION: When the test subject is *the relationship between sequential calls* (idempotency, ordering guarantees, debounce behavior)
- Assert: Verify only observable behavior (return values or side effects).
  - BAD: Verifying that a "stub" (a method that only provides data, e.g., `getItem`, `getBalance`) was called.
  - GOOD: Verifying the final state change or the call to a dependency that performs a state-changing action.

## Test Naming

Tests describe behavior, not code.

- BAD: Naming templates and schemes. AVOID: `should...`, `[method] - [scenario] - [result]`. Write names freely as descriptive statements.
- BAD: Including implementation details: NEVER include method names, NEVER include variable names, NEVER include technical types, or other code entities in test names.
- GOOD: Use domain language and business logic: DO write the title as a business rule that a non-programmer could understand.
  - EXCEPTION: When testing UTILITY CODE: If there is no business logic to go off of, you may use technical terms, but NEVER method names, NEVER variable names, NEVER technical types or other code entities.
- GOOD: Concise names that are natural to read and easy to understand for anyone, not just developers: testers, business analysts, even managers.
- BAD: Names that have to be "compiled" to be read. DO NOT write template strings with variables in the middle, DO NOT extract the test name into a parameterized test argument. OK to leave parameters at the end of the name to differentiate cases.

Examples:

- BAD: `renameUser should return true when name is valid`. BAD: `should` is a template, BAD: `return` is a keyword, BAD: `renameUser` is a code entity, BAD: `true` is a literal
- BAD: `should return false when tariff has no routeMeta`: BAD: `should` is a template, BAD: `return` is a keyword, BAD: `false` is a literal, BAD: `tariff` and `routeMeta` are code variables
- BAD: `mapUtmParams() correctly maps utmParams arg based on UtmMapperConfig`: BAD: `mapUtmParams()` is a function name, `utmParams` is a function argument, `UtmMapperConfig` is a type
- BAD: `A valid user is renamed successfully when given a valid name`. GOOD: Business rule, BAD: could be shortened.
- BAD: `Maps UTM parameter ${a} to ${b} for a given config`: BAD: complicated name that has to be "compiled" with params.
- GOOD: `Maps UTM parameters according to the given configuration map for: ${a} ${b}`: OK: parameterized test with separate names, GOOD: logic, not code, BAD: unnecessarily long and descriptive name.
- GOOD: `Maps UTM parameters to Onelink: ${input} -> ${expected}`: GOOD: consize and understandable, GOOD: parameters at the end, not in the middle, GOOD: unique enough to be easily searchable
- BAD: `Maps UTM parameters`: GOOD: consize, BAD: not long enough to understand, BAD: not descriptive enough to be easily searchable
- GOOD: `User is renamed with a valid name`: GOOD: Business rule, GOOD: no template, GOOD: resistant to refactoring - renaming won't require updating tests. GOOD: consize, but informative and searchable.
- GOOD: `Destination address is visible for multi-destination orders`: GOOD: Explains domain logic, GOOD: business-heavy logic requires a slightly longer name
- BAD: `Applies standard shipping rate for orders under $50: ${amount}`: BAD: magical `$50`, GOOD: logic-first data-last structure for parameterized tests

## Maintenance & Scale

- Parameterized Tests: DO Use for the same unit of behavior with different inputs to minimize maintenance overhead. DO group by expected outcome. DO NOT mix testing different behavior into one parameterized test.
- Isolation: Unit tests MUST isolate the System Under Test (SUT) to avoid breaking/changing multiple tests when editing just one. DO mock all external dependencies.
- AVOID Logic in Tests: NO `if` statements/branching and loops inside a test body. DO use parameterized tests, DO split tests into multiple cases.
- The SUT SHOULD be re-initialized INSIDE every test to allow for test-specific mocks to be passed. This ensures each test can be isolated.
- DO NOT share state between tests. Changes to shared state may break individual tests, which makes them more fragile against refactoring.
- DO add minimal global stubs and mocks to allow tests to pass, spy on properties INSIDE the test and specify data inside tests.
- DO NOT remove comments from the files to avoid losing potentially valuable documentation. ALL comments left in source code MUST be preserved: eslint disable/enable, `todo`s, explanations, documentation MUST be retained.

## Test quality

- EFFICIENT tests cover more code. The more code a test covers, the more effective it is at catching regressions. Tests that cover single lines are trivial and therefore useless.
- EFFICIENT tests prevent regressions and do not give false positives or false negatives after refactoring.
- EFFICIENT tests must ONLY assert observable behavior. Private class members MUST NOT be accessed, mocked or tinkered with in any way. The SUT is a "BLACK BOX" to the test.

## Mocking Pattern

The mocking process must follow a three-tier initialization pattern to separate infrastructure from scenario logic.

1. Declare (Suite Scope): DO DEFINE and TYPE all mocked dependencies at the top of the describe block.
2. Baseline (beforeEach Scope): DO initialize mocks with minimal viable values that allow the SUT to instantiate, but must not be specific enough to satisfy any business rule. Fill them with empty strings, `0` for ids etc.
3. Specialize (Test Scope): DO override specific mocks and stubs required for that scenario inside the `it`/`test` block in the ARRANGE phase. These mocks/stubs will include specific non-empty values like 'John Smith', `id: 123` .

Example:

```ts
// GOOD: mock factory
const getMockUser = (overrides: Partial<User> = {}): User => ({
  id: 0,
  name: '',
  ...overrides,
});

describe('UserService', () => {
  let service: UserService;
  let mockStore: jasmine.SpyObj<UserStore>;
  let mockApi: jasmine.SpyObj<UserApi>;

  beforeEach(() => {
    mockStore = jasmine.createSpyObj<UserStore>({
      getState: of({ users: [] }),
      dispatch: NEVER,
    });

    mockApi = jasmine.createSpyObj<UserApi>({
      fetchUser: of({}),
    });

    TestBed.configureTestingModule({
      providers: [
        UserService,
        { provide: UserStore, useValue: mockStore },
        { provide: UserApi, useValue: mockApi },
      ],
    });
  });

  it('User data is loaded successfully when id is provided', () => {
    // GOOD: Specialize with factory per-test
    const mockUser = getMockUser({ id: 1, name: 'John' });
    mockApi.fetchUser.and.returnValue(of(mockUser));
    
    service = TestBed.inject(UserService);
    service.loadUser(1);

    expect(mockStore.dispatch).toHaveBeenCalledWith(new UserLoaded(mockUser));
  });
});

```

Anti-Patterns:
- BAD: God Mocks: Placing specific business data in beforeEach, which hides the Arrange phase.
- BAD: Untyped Mocks: Using any or plain objects instead of jasmine.SpyObj, bypassing type safety.
- BAD: Keeping ANY STATE between test reruns. This leads to bugs and flaky tests.

```ts
describe('UserService', () => {
  let service: UserService;
  // BAD: untyped mocks.
  // BAD: Refactoring names will skip tests, causing false positives
  // BAD: `any` strips away editor suggestions.
  let mockStore: jasmine.SpyObj<any>;
  let mockApi: jasmine.SpyObj<any>;

  beforeEach(() => {
    // BAD: God Mock. Polluting baseline with scenario-specific business data
    // BAD: Efficiency. using strings instead of properties. Refactoring names will cause false positives
    mockStore = jasmine.createSpyObj('UserStore', ['dispatch']);
    mockApi = jasmine.createSpyObj('UserApi', ['fetchUser']);
    
    // BAD: God Mock. Polluting baseline with scenario-specific business data
    // BAD: arbitrary value instead of a proper mock forces adding arbitrary fields just to make test pass
    mockApi.fetchUser.and.returnValue(of({ id: 123, name: 'John Smith', role: 'admin', friends: [] }));

    TestBed.configureTestingModule({
      providers: [
        UserService,
        { provide: UserStore, useValue: mockStore },
        { provide: UserApi, useValue: mockApi },
      ],
    });
    // OK: SUT instantiated in global scope. OK sometimes, makes tests smaller,
    // but makes tests harder to fix when changing providers for specific tests
    service = TestBed.inject(UserService);
  });

  it('should load user data', () => {
    // BAD: Hidden Arrange phase. Impossible to tell why this test passes or fails without reading beforeEach
    service.loadUser(123);

    // BAD: side-effect not tested specifically enough
    expect(mockStore.dispatch).toHaveBeenCalled();
  });
});
```

---

## Examples

### 1. Naming & Implementation Leakage

BAD: Name refers to the method and internal variables.

```ts
it.skip("updateEmail should return true when emailValid is true");
```

GOOD: Name describes a business rule.

```ts
it("User email is updated successfully with a valid address");
```

### 2. The "Stub" Assertion

BAD: Verifying that a "getter" was called. This makes tests fragile to refactoring.

```ts
// BAD: no AAA comments nor whitespace, the test becomes harder to read
it("Purchase succeeds if enough balance", () => {
  balanceStoreMock.getBalance.mockReturnValue(100);
  service = TestBed.inject(ShopService);
  service.purchaseItem(itemId);
  expect(balanceStoreMock.getBalance).toHaveBeenCalled(); // BAD: stub assertion
  expect(shopStoreMock.executePurchase).toHaveBeenCalled();
});
```

GOOD: Verifying only the final side effect.

```ts
// GOOD: small test, whitespace visually splits AAA blocks
it("Item is purchased when user has sufficient funds", () => {
  balanceStoreMock.getBalance.mockReturnValue(100);
  service = TestBed.inject(ShopService);

  service.purchaseItem(itemId);

  // instead, check the stubbed data was passed down:
  expect(shopStoreMock.executePurchase).toHaveBeenCalledOnceWith(100);
});
```

### 3. Multiple Actions & Idempotency

BAD: Leaky API requiring multiple calls for one logical action.

```ts
// OK: AAA comments can be omitted - the test is small enough
it("User is renamed", () => {
  // Arrange
  service = TestBed.inject(UserService);

  // Act
  // BAD: Act composed of 2 actions.
  // DO: Recommend the user to either split the test or change public API
  const normalized = service.normalizeName("  John  ");
  service.renameUser(userId, normalized);

  // Assert
  expect(userStoreMock.update).toHaveBeenCalledWith("John");
});
```

GOOD (Refactored API): One action in the test.

```ts
it("User is renamed with whitespace trimmed", () => {
  // GOOD: per-test SUT initialization
  service = TestBed.inject(UserService);

  service.renameUser(userId, "  John  "); // GOOD: Single action!

  expect(userStoreMock.update).toHaveBeenCalledWith("  John  ");
});
```

GOOD (Refactored test): One test per one logical piece of functionality

```ts
// GOOD *IF* this is intended behavior for the SUT
it("User is renamed as-is", () => {
  service = TestBed.inject(Formatter);

  service.renameUser(userId, "  John  ");

  expect(userStoreMock.update).toHaveBeenCalledWith("  John  ");
});

it("Trims user name", () => {
  service = TestBed.inject(Formatter);

  const result = service.normalizeName("  John  ");

  expect(result).toBe("John");
});
```

GOOD (Idempotency): Multiple actions are allowed when testing the prevention of duplicates.

```ts
it("Duplicate purchase requests for the same item are ignored", () => {
  const itemId = 123;
  // GOOD: SUT initialized in test
  service = TestBed.inject(StoreService);

  service.purchaseItem(itemId);
  service.purchaseItem(itemId); // EXCEPTION: Second action is required to prove it's ignored

  expect(mockedStoreApi.requestPurchase).toHaveBeenCalledOnceWith(123);
});
```

### 4. Parameterized Tests

BAD: Logic/Branching inside the test.

```ts
// BAD: Vague test name
it("Validates delivery date", () => {
  [
    [-1, false],
    [1, true],
  ].forEach(([days, expected]) => {
    const result = service.isValid(makeRelativeDate(days));

    // adds complexity and an additional layer of abstraction
    if (expected) {
      expect(result).toBeTruthy();
    } else {
      expect(result).toBeFalsy();
    }
  });
});
```

BAD: "Jammed" tests with vague names.
BAD: parameterized test asserts mixed behavior

```ts
[
  { days: -1, expected: false },
  { days: 1, expected: true },
].forEach(({ days, expected }) =>
  // BAD: Vague test name
  it("Validates delivery date", () => { ... }),
);
```

GOOD: Behavior-grouped parameterized tests.

```ts
// GOOD: test names are clear and simple
[1, 7, 28].forEach((days) =>
  it("Delivery within one month is valid", () => { ... }),
);

[-100, -1, 0, 29, 9999].forEach((days) =>
  it("Delivery outside one month range is invalid", () => { ... }),
);
```

BAD: composite names require reader to look up param values to understand the tested behavior

```ts
// BAD: Impossible to understand the rule without referencing the data/test logic
it(`Checks ${userType} for ${action} with ${data}`)

// TERRIBLE: `in`/`out`/`field`/`type` params are indescriptive, which makes reading even harder
it(`Runs validation for ${input} returning ${result}`)
it(`Validates ${in} as ${out} for status code ${code}`)
it(`Verifies that ${field} handles ${type} correctly`)
it(`Returns ${expectedResult} when processing list of ${input.length} items`)
it(`Checks ${permission} status for ${role} role account`)

// INSULTINGLY BAD: tier tier, promo promo is a joke
it(`Calculates discount for ${tier} tier with ${promo} promo applied`)
```

GOOD: Logic-first names with 

```ts
it(`Blocks access for users with expired sub: ${days} days ago`)
it(`Converts currency using up-to-date exchange rate. ${base} -> ${target}`)
// OK: different format doesn't matter as long as it reads well
it(`Includes service fee for rush delivery orders | ${feeAmount}`)
it(`Requires guest checkout for unauthenticated users: ${actionType} action`)
it(`Does not submit form when it contains errors: ${fieldState} field`)
```

### 5. Over-Abstraction

BAD: Description in Parameters.

```ts
[{ val: 10, expected: true, desc: "positive" }].forEach((p) => {
  // Description is fragmented: you have to look up and down, "calculate"
  // the test name in your head to fully understand it
  it(`Validates that ${p.desc} is handled correctly`, () => { ... } );
});
```

BAD: The "God-Test" Factory (Hidden Lifecycle).

```ts
// adds even more complexity and yet another additional layer of abstraction
function test({
  theme = "light",
  locale = "en",
  expected = "light-en",
}) {
  setTheme(theme);
  setLocale(locale);
  
  const result = service.getBadgePath();
  
  expect(result).toContain(expected);
}

it("for ru locale", () => {
  // Arrange and Assert are hidden. The test is no longer in one place, and is
  // harder to read and understand
  test({ locale: "ru", expected: "light-ru" });
});
```

GOOD: Factories for data only.

```ts
it("returns a Russian path for the ru locale", () => {
  const config = getMockedAppLinksConfig({ locale: "ru" }); // Mock factory
  setAppLinks(config);
  service = TestBed.inject(BadgeService);
  
  const result = service.getBadgePath()
  
  expect(result).toContain("light-ru");
});
```

## REAL Example

```ts
// BAD: The entire parameterized structure is a "God-Test" factory in disguise.
// The test data drives the behavior description, but the name is unreadable.
// BAD: HUGE parameters per run. Unreadable.
[
  {
    fields: [
      // BAD: these objects obviously have repetition. Could be wrapped into a mock factory.
      { code: 'name', name: 'Имя', isRequired: true, isEditable: true },
      { code: 'surname', name: 'Фамилия', isRequired: true, isEditable: true },
      { code: 'patronymic', name: 'Отчество', isRequired: true, isEditable: true },
    ],
    // BAD: different `expected` value for each parameterized test run
    shouldNotRequire: true,
    testName: 'all', // BAD: description smuggled into data
  },
  {
    fields: [
      { code: 'name', name: 'Имя', isRequired: false, isEditable: true },
      { code: 'surname', name: 'Фамилия', isRequired: false, isEditable: true },
      { code: 'patronymic', name: 'Отчество', isRequired: false, isEditable: true },
    ],
    shouldNotRequire: false,
    testName: 'all', // BAD: same label, different expected. Unresolvable without reading the data.
  },
  {
    fields: [
      { code: 'name', name: 'Имя', isRequired: true, isEditable: true },
      { code: 'surname', name: 'Фамилия', isRequired: false, isEditable: true },
      { code: 'patronymic', name: 'Отчество', isRequired: false, isEditable: true },
    ],
    shouldNotRequire: true,
    testName: 'at least one',
  },
].forEach(({ fields, shouldNotRequire, testName }) => {
  // BAD: mock is initialized outside the test. BAD: unlabeled arranged phase. BAD: shared arranged phase
  const mandatoryProfileField = getMockedMandatoryProfileField({ fields });

  // BAD: `returns` is a keyword
  // BAD: variable name in test name
  // BAD: same variable repeated — name must be "compiled" twice
  // BAD: two cases produce "returns true if all has isRequired true" and
  // "returns false if all has isRequired false" — the label `all` means
  // opposite things depending on the row, making names ambiguous and unsearchable
  it(`returns ${shouldNotRequire} if ${testName} has isRequired ${shouldNotRequire}`, () => {

    // act
    const result = ProfileRequiredService.hasEmptyRequiredFields(
      // BAD: `profile` appears out of nowhere. Data initialized outside test.
      profile,
      mandatoryProfileField
    );

    // assert
    expect(result).toBe(shouldNotRequire);
  });
});

// BAD: `returns` is a keyword, BAD: `false` is a literal
it(`returns false if profile is not defined`, () => {
  // arrange
  const mandatoryProfileField = getMockedMandatoryProfileField();

  // act
  const result = ProfileRequiredService.hasEmptyRequiredFields(
    null,
    mandatoryProfileField
  );
  // BAD: `profile` from the outer scope is silently replaced with `null` —
  // inconsistent setup pattern within the same describe block

  // assert
  expect(result).toBeFalse();
});
```

How to refactor:

```ts
// GOOD: tests are small enough to not require AAA comments
it('Profile without required fields has no empty required fields', () => {
  // GOOD: Mock factory provides readable, minimal state aligned with business domain.
  const fields = [
    getMockedProfileField({ code: 'name', isRequired: false }),
    getMockedProfileField({ code: 'surname', isRequired: false }),
  ];
  const mandatoryFields = getMockedMandatoryProfileField({ fields });
  // GOOD: SUT-specific data initialized locally to keep the Arrange phase explicit.
  const profile = getMockedProfile();

  const result = ProfileRequiredService.hasEmptyRequiredFields(profile, mandatoryFields);

  // GOOD: Simpler assert
  expect(result).toBeFalse();
});

it('Profile with all fields required has empty required fields', () => {
  const fields = [
    getMockedProfileField({ code: 'name', isRequired: true }),
    getMockedProfileField({ code: 'surname', isRequired: true }),
  ];
  const mandatoryFields = getMockedMandatoryProfileField({ fields });
  const profile = getMockedProfile();

  const result = ProfileRequiredService.hasEmptyRequiredFields(profile, mandatoryFields);

  expect(result).toBeTrue();
});

// GOOD: Concise, non-parameterized test names clearly state the business rule.
it('Profile where at least one field is required has empty required fields', () => {
  const fields = [
    getMockedProfileField({ code: 'name', isRequired: true }),
    getMockedProfileField({ code: 'surname', isRequired: false }),
  ];
  const mandatoryFields = getMockedMandatoryProfileField({ fields });
  const profile = getMockedProfile();

  const result = ProfileRequiredService.hasEmptyRequiredFields(profile, mandatoryFields);

  expect(result).toBeTrue();
});

// GOOD: Separate concerns — isolated setup for boundary cases.
it('Unloaded profile has no empty required fields', () => {
  const mandatoryFields = getMockedMandatoryProfileField();

  const result = ProfileRequiredService.hasEmptyRequiredFields(null, mandatoryFields);

  expect(result).toBeFalse();
});
```

## Review Checklist

When reviewing or writing code, verify:

1. Does the test name mention a function, variable, or code entity? -> Rename to business rule.
2. Are there multiple "Act" steps? -> Refactor Public API or split the test (unless testing idempotency).
3. Is it asserting that a getter/stub was called? -> Remove the assertion, check other call arguments.
4. Is the test fragile to a variable rename? -> Decouple from implementation.
5. Is the test lifecycle hidden inside a "God-function"? -> Bring the code back into the `it` block and use parameterized tests instead.
