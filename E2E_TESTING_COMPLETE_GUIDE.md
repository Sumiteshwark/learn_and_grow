# E2E Testing Complete Guide — Web (React) & Mobile (React Native)

A comprehensive guide covering E2E testing concepts, workflows, team responsibilities, CI/CD pipelines, and the complete production release lifecycle.

---

## Table of Contents

1. [Types of Tests — Unit vs Integration vs E2E](#1-types-of-tests-unit-vs-integration-vs-e2e)
2. [Unit Tests — Deep Dive](#2-unit-tests-deep-dive)
3. [Integration Tests — Deep Dive](#3-integration-tests-deep-dive)
4. [Contract Tests — Deep Dive](#4-contract-tests-deep-dive)
5. [E2E Tests — Deep Dive](#5-e2e-tests-deep-dive)
6. [Web E2E vs Mobile E2E](#6-web-e2e-vs-mobile-e2e)
7. [Smoke Tests — What, Why, and How](#7-smoke-tests-what-why-and-how)
8. [Team Structure & Responsibilities](#8-team-structure-and-responsibilities)
9. [Tech Stack for React Web E2E](#9-tech-stack-for-react-web-e2e)
10. [Project Structure](#10-project-structure)
11. [Implementation Patterns](#11-implementation-patterns)
12. [Complete Production Pipeline — Detailed Flow](#12-complete-production-pipeline-detailed-flow)
13. [Gate Reference Summary](#13-gate-reference-summary)
14. [Hotfix — Emergency Production Fix](#14-hotfix-emergency-production-fix)
15. [Testing Pyramid](#15-testing-pyramid)

---

## 1. Types of Tests—Unit vs Integration vs E2E

| Aspect | Unit Test | Integration Test | E2E Test |
|---|---|---|---|
| **What it tests** | Single function/component in isolation | Multiple units working together | Full user journey through the app |
| **Dependencies** | All mocked | Internal = real, External = mocked | Everything real (real browser, real server) |
| **Speed** | Very fast (ms) | Medium (seconds) | Slow (seconds to minutes) |
| **Failure diagnosis** | Easy — exact function identified | Medium — one of several connected pieces | Hard — could be anything in the stack |
| **Confidence** | Logic is correct | Pieces connect correctly | Whole app works for users |
| **Who writes** | Developers | Developers / SDETs | SDETs (QA Engineers) |
| **Framework (React)** | Jest + React Testing Library | Jest + React Testing Library | Playwright / Cypress |
| **Bug catch rate** | ~70% of bugs | ~20% of bugs | ~10% of bugs |

---

## 2. Unit Tests—Deep Dive

### What they do
Test a **single function or component in isolation**. All external dependencies (APIs, databases, other modules) are mocked.

### Example — Pure function test
```ts
// formatPrice.test.ts
import { formatPrice } from './formatPrice';

describe('formatPrice', () => {
  test('formats whole numbers', () => {
    expect(formatPrice(10)).toBe('$10.00');
  });

  test('formats decimals', () => {
    expect(formatPrice(10.5)).toBe('$10.50');
  });

  test('handles zero', () => {
    expect(formatPrice(0)).toBe('$0.00');
  });

  test('handles negative', () => {
    expect(formatPrice(-5)).toBe('-$5.00');
  });
});
```

### Example — Component test with mocks
```tsx
// Button.test.tsx
import { render, fireEvent } from '@testing-library/react-native';
import Button from './Button';

const onPressMock = jest.fn();

test('calls onPress when pressed', () => {
  const { getByTestId } = render(
    <Button title="Submit" onPress={onPressMock} testID="btn" />
  );
  fireEvent.press(getByTestId('btn'));
  expect(onPressMock).toHaveBeenCalledTimes(1);
});
```

### Benefits of unit tests
1. **Catch logic bugs fast** — verify edge cases, boundary conditions, error handling
2. **Speed** — run thousands in seconds, instant feedback during development
3. **Pinpoint failures** — know exactly which function broke and why
4. **Force better design** — hard-to-test code = code doing too much
5. **Safe refactoring** — immediately know if you broke the contract
6. **Living documentation** — test names describe what the function does

### What unit tests miss
- Integration bugs (pieces work alone but fail together)
- Real network/database behavior
- UI rendering issues in real browsers/devices
- Race conditions and timing issues

---

## 3. Integration Tests—Deep Dive

### What they do
Test **multiple units working together**. Internal interactions between modules are **real** (not mocked). Only truly external services may be mocked.

### Key principle
The whole point is to catch bugs at the **seams** where modules connect. Mocking those seams defeats the purpose.

### What gets mocked vs what stays real

| Scenario | What's real | What's mocked |
|---|---|---|
| API integration test | Your code + real API server (test server) | Nothing, or just auth |
| Database integration test | Your code + real database (test DB) | External APIs |
| Service integration test | Multiple services talking to each other | Nothing ideally |
| Component integration test | Multiple components + real state management | External APIs, native modules |

### Example — Unit test vs Integration test

```ts
// UNIT TEST: mock the database
jest.mock('./database');
test('getUser formats name correctly', () => {
  database.findById.mockReturnValue({ first: 'John', last: 'Doe' });
  expect(getUser(1)).toEqual({ fullName: 'John Doe' });
});

// INTEGRATION TEST: use a real test database
test('getUser fetches from DB and formats correctly', async () => {
  // No mock — real DB query + real formatting logic
  await seedDatabase({ id: 1, first: 'John', last: 'Doe' });
  const user = await getUser(1);
  expect(user).toEqual({ fullName: 'John Doe' });
});
```

### Summary
- **Unit test**: "Does this one piece work correctly in isolation?"
- **Integration test**: "Do these pieces work correctly **together**?"
- Integration tests mock **as little as possible** — only external services you can't control

---

## 4. Contract Tests—Deep Dive

### What is a contract test?
A contract test verifies that **two services agree on the API format** between them — the request shape and the response shape. Think of it as a **written agreement** between a frontend and backend (or between two microservices):

> "I will send you `{ email: string, password: string }` and you will return `{ token: string, userId: string }`"

That agreement is the **contract**.

### Why do we need it?

Without contract tests, this happens:

```
Frontend expects:    { userId: "abc", token: "xyz" }
Backend changes to:  { user_id: "abc", accessToken: "xyz" }
                          ^                ^
                     renamed            renamed

Unit tests:   All pass (both sides tested in isolation with their own mocks)
Integration:  May not catch it (if using mocks)
Production:   APP BREAKS
```

Contract tests catch this **before** deployment.

### How it works

```
Frontend (consumer)                    Backend (provider)
        |                                      |
        |  Writes a contract:                  |
        |  "I expect POST /login to            |
        |   return { token, userId }"          |
        |                                      |
        +---------> contract.json <-----------+
                         |                     |
                         |    Backend runs     |
                         |    contract against |
                         |    its real API     |
                         |                     |
                    If mismatch:               |
                    TEST FAILS                 |
                    "You broke the contract!"  |
```

### Example contract file

```json
// contract: login-api.contract.json
{
  "consumer": "web-frontend",
  "provider": "auth-service",
  "interactions": [
    {
      "description": "valid login",
      "request": {
        "method": "POST",
        "path": "/api/login",
        "body": {
          "email": "string",
          "password": "string"
        }
      },
      "response": {
        "status": 200,
        "body": {
          "token": "string",
          "userId": "string"
        }
      }
    }
  ]
}
```

### How contract testing works step by step

```
Step 1: Frontend team (consumer) writes the contract
        "Here's what I expect from your API"
                    |
                    v
Step 2: Contract is stored in a shared location
        (Pact Broker, git repo, or artifact storage)
                    |
                    v
Step 3: Backend team (provider) pulls the contract
        Runs it against their actual API
                    |
                    v
Step 4: If the API matches the contract -> PASS
        If the API doesn't match       -> FAIL
        Backend team is notified: "You broke the contract!"
                    |
                    v
Step 5: Backend fixes the issue OR
        Both teams agree to update the contract
```

### Consumer-Driven Contract Testing (most common approach)

```ts
// Frontend writes this test (using Pact)
import { PactV3 } from '@pact-foundation/pact';

const provider = new PactV3({
  consumer: 'WebFrontend',
  provider: 'AuthService',
});

describe('Auth API Contract', () => {
  it('returns token on valid login', async () => {
    // Define what we expect
    provider
      .given('a user exists')
      .uponReceiving('a valid login request')
      .withRequest({
        method: 'POST',
        path: '/api/login',
        body: { email: 'user@test.com', password: 'pass123' },
      })
      .willRespondWith({
        status: 200,
        body: {
          token: like('some-jwt-token'),    // any string
          userId: like('user-123'),          // any string
        },
      });

    // Run the actual consumer code against the mock
    await provider.executeTest(async (mockServer) => {
      const result = await loginUser(mockServer.url, 'user@test.com', 'pass123');
      expect(result.token).toBeDefined();
      expect(result.userId).toBeDefined();
    });
  });
});
```

```ts
// Backend verifies the contract (using Pact)
import { Verifier } from '@pact-foundation/pact';

describe('Auth Service Provider Verification', () => {
  it('validates the contract with WebFrontend', async () => {
    const verifier = new Verifier({
      providerBaseUrl: 'http://localhost:3000',
      pactUrls: ['./pacts/WebFrontend-AuthService.json'],
      stateHandlers: {
        'a user exists': async () => {
          // Seed test data
          await createTestUser({ email: 'user@test.com', password: 'pass123' });
        },
      },
    });
    await verifier.verifyProvider();
  });
});
```

### Contract testing tools

| Tool | Best for | Language |
|---|---|---|
| **Pact** | Most popular, consumer-driven | JavaScript, Java, Python, Go, Ruby, .NET |
| **Spring Cloud Contract** | Java/Spring microservices | Java |
| **Swagger/OpenAPI validation** | Lightweight, schema-based | Any |
| **Specmatic** | OpenAPI spec as contract | Any |

### When to use contract tests

| Situation | Use contract tests? |
|---|---|
| Microservices architecture (many services talking) | Yes, essential |
| Separate frontend and backend teams | Yes, very useful |
| Monolith app (everything in one codebase) | Usually not needed |
| Third-party APIs you don't control | Not applicable (you can't enforce the contract) |
| Backend changes frequently | Yes, prevents surprise breaks |
| Single full-stack developer | Overkill |

### Contract tests vs other test types

| Aspect | Contract Test | Integration Test | E2E Test |
|---|---|---|---|
| **What it verifies** | API shape agreement | Multiple units work together | Full user journey |
| **Needs real server?** | Provider verification does | Sometimes | Yes |
| **Speed** | Fast | Medium | Slow |
| **Catches** | API breaking changes | Wiring bugs | User-facing bugs |
| **Runs in pipeline** | Gate 3 (regression) | Gate 3 (regression) | Gate 1 (smoke) + Gate 3 (full) |

---

## 5. E2E Tests—Deep Dive

### What they do
Simulate a **real user** interacting with the application from start to finish. Real browser, real server, real database — no mocks.

### What makes them different
- Run against a **deployed application** (staging or test environment)
- Use a **real browser** (Chrome, Firefox, Safari)
- Interact with UI elements the way a user would (click, type, navigate)
- Verify complete user journeys, not individual functions

### Example — Playwright E2E test
```ts
import { test, expect } from '@playwright/test';

test('user can login and send a message', async ({ page }) => {
  // Navigate to app
  await page.goto('https://staging.myapp.com');

  // Login
  await page.fill('[data-testid="email"]', 'user@test.com');
  await page.fill('[data-testid="password"]', 'pass123');
  await page.click('[data-testid="submit"]');

  // Verify dashboard loaded
  await expect(page).toHaveURL('/dashboard');

  // Send a message
  await page.click('[data-testid="new-message"]');
  await page.fill('[data-testid="message-input"]', 'Hello world');
  await page.click('[data-testid="send"]');

  // Verify message appears
  await expect(page.locator('text=Hello world')).toBeVisible();
});
```

---

## 6. Web E2E vs Mobile E2E

### Core concept is the same
Both test the full user journey end-to-end by simulating real user interactions.

### But execution is fundamentally different

| Aspect | Web E2E (React) | Mobile E2E (React Native) |
|---|---|---|
| **Runs on** | Browser (Chrome, Firefox, Safari) | Emulator/Simulator or real device (iOS/Android) |
| **Interacts with** | DOM elements (HTML/CSS) | Native UI elements (UIKit/Android Views) |
| **Selectors** | CSS selectors, XPath, `data-testid` | Accessibility IDs, `testID`, text labels |
| **Tools** | Playwright, Cypress, Selenium | Maestro, Detox, Appium |
| **Actions** | `click()`, `fill()`, `hover()` | `tapOn`, `swipe`, `scroll`, `inputText` |
| **Navigation** | URL-based (`goto('/login')`) | Screen-based (tap buttons to navigate) |
| **Setup** | Just needs a browser | Needs device/emulator + app build |
| **Speed** | Fast (seconds) | Slow (needs app compile + device boot) |
| **Platform testing** | One codebase, multiple browsers | Must test iOS AND Android separately |

### Same test — different worlds

**Web (Playwright):**
```js
await page.goto('/login');
await page.fill('[data-testid="email"]', 'user@test.com');
await page.fill('[data-testid="password"]', 'pass123');
await page.click('[data-testid="submit"]');
await expect(page.locator('[data-testid="rooms-list"]')).toBeVisible();
```

**Mobile (Maestro):**
```yaml
- tapOn:
    id: 'login-view-email'
- inputText: user@test.com
- pressKey: enter
- tapOn:
    id: 'login-view-password'
- inputText: pass123
- tapOn:
    id: login-view-submit
- extendedWaitUntil:
    visible:
      id: rooms-list-view
    timeout: 60000
```

### Key differences in practice

**1. No URL navigation in mobile**
- Web: jump to any page via URL
- Mobile: must tap through the UI to reach every screen

**2. Platform-specific behavior**
- Web: mostly consistent across browsers
- Mobile: iOS and Android behave differently (keyboard, gestures, permissions, notifications), often need separate test runs

**3. Native features (mobile only)**
- Push notifications
- Camera/photo permissions
- Biometric auth (Face ID, fingerprint)
- Deep links
- App backgrounding/foregrounding
- Swipe gestures, long press

**4. Environment complexity**
- Web: start a server, open a browser — done
- Mobile: compile native app -> boot emulator/simulator -> install app -> run tests

**5. Flakiness**
Mobile E2E tests are generally more flaky due to animation timing, emulator performance, and network variability

### Mobile E2E framework options
- **Maestro** — simplest, YAML-based (used by Rocket.Chat React Native)
- **Detox** (by Wix) — most popular for React Native, JavaScript-based
- **Appium** — cross-platform, Selenium-like API for mobile

---

## 7. Smoke Tests—What, Why, and How

### What is a smoke test?
A small subset of your most critical test cases that verifies the **core functionality** is working. A quick health check — "Is the app fundamentally broken or not?"

The name comes from hardware testing — power on a circuit board, if it smokes, something is fundamentally wrong.

### What qualifies as a smoke test?

**Decision framework — ask 3 questions:**

| Question | Yes | No |
|---|---|---|
| If this breaks, can users use the app at all? | Smoke test | Regular test |
| Does this affect revenue / core business? | Smoke test | Regular test |
| Does 80%+ of users hit this flow? | Smoke test candidate | Regular test |

### Example — E-commerce app

| Flow | Smoke? | Why |
|---|---|---|
| Homepage loads | Yes | 100% of users see this |
| User can log in | Yes | Can't do anything without it |
| Search works | Yes | Core feature, 90%+ users use it |
| Can add to cart | Yes | Core business flow |
| Can checkout / pay | Yes | Revenue depends on this |
| Can view order history | No | Important but not critical |
| Can write a review | No | Nice-to-have, not core |
| Can change address | No | Used by small % of users |
| Dark mode works | No | Cosmetic, doesn't block usage |

### Example — Chat app (like Rocket.Chat)

| Flow | Smoke? | Why |
|---|---|---|
| Login | Yes | Blocks everything |
| See rooms list | Yes | Main screen after login |
| Open a room | Yes | Core functionality |
| Send a message | Yes | The whole point of the app |
| Search rooms | Maybe | Important but app works without it |
| Change avatar | No | Not critical |
| E2E encryption | No | Subset of users |
| i18n / language | No | Doesn't block core usage |

### Who decides?
Collaborative decision:
- **QA Lead / SDET Lead** — proposes the smoke test list based on risk & usage
- **Product Manager** — validates from business perspective
- **Engineering Lead** — validates from technical perspective
- All agree -> smoke test suite is defined

### How to implement in code

**Option 1: Tags (most common)**
```ts
// Playwright
test('user can login', { tag: '@smoke' }, async ({ page }) => {
  // ...
});

test('user can send message', { tag: '@smoke' }, async ({ page }) => {
  // ...
});

test('user can change avatar', async ({ page }) => {
  // no tag = not smoke
});
```

Run commands:
```bash
npx playwright test --grep @smoke          # smoke only ~5 min
npx playwright test                         # full suite ~30 min
```

**Option 2: Separate folders**
```
e2e/
  smoke/              # Only critical paths
    login.spec.ts
    dashboard.spec.ts
    send-message.spec.ts
  regression/         # Everything else
    settings.spec.ts
    avatar.spec.ts
    i18n.spec.ts
```

**Option 3: Config-based**
```ts
// playwright.config.ts
export default defineConfig({
  projects: [
    {
      name: 'smoke',
      testMatch: /.*\.smoke\.spec\.ts/,
      timeout: 30000,
    },
    {
      name: 'regression',
      testMatch: /.*\.spec\.ts/,
      timeout: 60000,
    }
  ]
});
```

### Typical numbers

| App size | Smoke tests | Full suite |
|---|---|---|
| Small app | 5-10 tests | 50-100 tests |
| Medium app | 15-30 tests | 200-500 tests |
| Large app (enterprise) | 30-50 tests | 1000+ tests |

**Rule of thumb**: Smoke tests = ~5-10% of full E2E suite, run in under 5 minutes.

---

## 8. Team Structure and Responsibilities

### Typical large company engineering team

```
Engineering Team
  |-- Frontend Devs (React)        -> write unit + component tests
  |-- Backend Devs                  -> write API unit + integration tests
  |-- SDETs (QA Engineers)          -> write E2E tests        <-- main E2E owners
  |-- Manual QA (if any)            -> exploratory testing
```

### SDET (Software Development Engineer in Test)
- **Primary owner** of E2E tests
- Dedicated engineers who only write test automation
- Typically 1 SDET per 4-6 developers

### Responsibility matrix

| Role | Responsibility |
|---|---|
| **Product Manager** | Define what to test (requirements/user stories) |
| **QA Lead** | Test strategy, prioritize what gets E2E coverage |
| **SDET** | Write, maintain, debug E2E tests |
| **Frontend Dev** | Add `data-testid` to components, fix bugs found by E2E |
| **DevOps / SRE** | CI/CD pipeline, test environments, monitoring |
| **Engineering Manager** | Ensure test coverage goals are met |

### The requirements flow

```
PM writes a ticket:
  "As a user, I can log in with email/password and see my dashboard"

QA Lead breaks it into test scenarios:
  - Valid login -> redirects to dashboard
  - Invalid password -> shows error message
  - Empty fields -> shows validation errors
  - Locked account -> shows account locked message
  - Session expired -> redirects to login
```

### Frontend dev's contract with SDETs

React developers must add `data-testid` attributes so SDETs can reliably select elements:

```tsx
<input data-testid="email-input" type="email" />
<button data-testid="login-submit">Log in</button>
<span data-testid="login-error">{error}</span>
```

Without these, E2E tests become fragile (relying on CSS classes or text content that changes).

---

## 9. Tech Stack for React Web E2E

| Component | Tool |
|---|---|
| **E2E Framework** | Playwright (most popular now) or Cypress |
| **Language** | TypeScript |
| **CI/CD** | GitHub Actions / Jenkins / CircleCI |
| **Reporting** | Allure / Playwright HTML Report |
| **Environment** | Staging server or Docker-based local env |
| **Test Data** | API seeding or database fixtures |

---

## 10. Project Structure

```
project/
  src/                              # React app code
  e2e/                              # E2E tests (separate folder)
    fixtures/                       # Test data, auth states
      users.json
      auth.setup.ts                 # Login once, reuse session
    pages/                          # Page Object Model (POM)
      LoginPage.ts
      DashboardPage.ts
      ProfilePage.ts
    tests/                          # Actual test files
      auth/
        login.spec.ts
        logout.spec.ts
      dashboard/
        dashboard.spec.ts
      settings/
        profile.spec.ts
    utils/                          # Helpers, API calls
      api.ts
      helpers.ts
  playwright.config.ts
  package.json
```

---

## 11. Implementation Patterns

### Pattern 1: Page Object Model (POM)

Reusable page objects that abstract UI interactions:

```ts
// e2e/pages/LoginPage.ts
export class LoginPage {
  constructor(private page: Page) {}

  // Selectors
  private emailInput = '[data-testid="email-input"]';
  private passwordInput = '[data-testid="password-input"]';
  private submitButton = '[data-testid="login-submit"]';
  private errorMessage = '[data-testid="login-error"]';

  // Actions
  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.page.fill(this.emailInput, email);
    await this.page.fill(this.passwordInput, password);
    await this.page.click(this.submitButton);
  }

  async getError() {
    return this.page.textContent(this.errorMessage);
  }
}
```

### Pattern 2: Write test specs

```ts
// e2e/tests/auth/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../../pages/LoginPage';

test.describe('Login', () => {
  let loginPage: LoginPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.goto();
  });

  test('successful login redirects to dashboard', { tag: '@smoke' }, async ({ page }) => {
    await loginPage.login('user@test.com', 'password123');
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="welcome-message"]')).toBeVisible();
  });

  test('invalid password shows error', async () => {
    await loginPage.login('user@test.com', 'wrongpassword');
    const error = await loginPage.getError();
    expect(error).toContain('Invalid credentials');
  });

  test('empty fields show validation', async ({ page }) => {
    await page.click('[data-testid="login-submit"]');
    await expect(page.locator('text=Email is required')).toBeVisible();
    await expect(page.locator('text=Password is required')).toBeVisible();
  });
});
```

### Pattern 3: Test data setup via API (not UI)

```ts
// e2e/utils/api.ts
// Create test data via API, not through UI clicks — much faster
export async function createTestUser(request: APIRequestContext) {
  const response = await request.post('/api/users', {
    data: {
      email: `test-${Date.now()}@test.com`,
      password: 'Test123!',
      name: 'Test User'
    }
  });
  return response.json();
}

export async function deleteTestUser(request: APIRequestContext, userId: string) {
  await request.delete(`/api/users/${userId}`);
}
```

### Pattern 4: Auth setup (login once, reuse across tests)

```ts
// e2e/fixtures/auth.setup.ts
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[data-testid="email-input"]', 'admin@test.com');
  await page.fill('[data-testid="password-input"]', 'password');
  await page.click('[data-testid="login-submit"]');
  await page.waitForURL('/dashboard');

  // Save signed-in state for other tests to reuse
  await page.context().storageState({ path: './e2e/.auth/user.json' });
});
```

---

## 12. Complete Production Pipeline—Detailed Flow

This is the complete lifecycle from code push to production deployment with every gate, pass/fail path, and responsible party.

```
                        Developer pushes code
                                |
                                v
                         PR created on GitHub
                                |
                                v
                +-------------------------------+
                |  GATE 1: Automated CI checks  |
                |  * Unit tests (Jest)          |
                |  * Lint + type check (ESLint) |
                |  * Smoke E2E tests            |
                |  (~5-10 minutes)              |
                +---------------+---------------+
                        +-------+-------+
                      Pass             Fail
                        |                |
                        |         PR is blocked
                        |         CI posts failure details
                        |         Dev fixes code & pushes again
                        |                |
                        |         +------+
                        |         | (re-runs Gate 1)
                        |         v
                        | <-------+
                        |
                        v
                +-------------------------------+
                |  GATE 2: Code Review          |
                |  * Peer developer reviews     |
                |  * SDET reviews test coverage |
                |  * Tech lead approves (if big)|
                +---------------+---------------+
                        +-------+-------+
                    Approved         Changes requested
                        |                |
                        |         Dev addresses feedback
                        |         Pushes new commits
                        |                |
                        |         Gate 1 re-runs automatically
                        |         Re-review requested
                        |                |
                        | <--------------+
                        |
                        v
                +-------------------------------+
                |  Merge PR -> develop/staging  |
                |  (automatic or manual merge)  |
                +---------------+---------------+
                                |
                                v
                +-------------------------------+
                |  GATE 3: Full Regression      |
                |  * All E2E tests              |
                |  * All integration tests      |
                |  * API contract tests         |
                |  * Performance baseline check |
                |  (~30-60 minutes)             |
                +---------------+---------------+
                        +-------+-------+
                      Pass             Fail
                        |                |
                        |         Investigate failure
                        |           +----+----+
                        |        Real bug   Flaky test
                        |           |           |
                        |      Dev creates   SDET fixes
                        |      hotfix PR     the test
                        |           |           |
                        |      Goes back to  Re-run
                        |      Gate 1        regression
                        |           |           |
                        | <---------+-----------+
                        |
                        v
                +-------------------------------+
                |  Promote develop -> main      |
                |  (merge or fast-forward)      |
                |  main is now stable + tested  |
                +---------------+---------------+
                                |
                                v
                +-------------------------------+
                |  GATE 4: Nightly Run          |
                |  (runs every night on main)   |
                |  * Full E2E suite             |
                |  * Cross-browser (Chrome,     |
                |    Firefox, Safari, Edge)     |
                |  * Multiple viewports         |
                |    (desktop, tablet, mobile)  |
                |  (~1-2 hours)                 |
                +---------------+---------------+
                        +-------+-------+
                      Pass             Fail
                        |                |
                        |         QA team investigates
                        |         next morning
                        |           +----+----+
                        |      Real bug    Env issue
                        |           |           |
                        |      Bug ticket   DevOps
                        |      created,     fixes infra,
                        |      prioritized  re-runs
                        |           |           |
                        | <---------+-----------+
                        |
                        v
                +-------------------------------+
                |  Release candidate tagged     |
                |  (e.g., v2.5.0-rc.1)          |
                |  Deployed to pre-prod / QA    |
                +---------------+---------------+
                                |
                                v
                +-------------------------------+
                |  GATE 5: Release Validation   |
                |  * Smoke tests on pre-prod    |
                |  * Full regression on pre-prod|
                |  * Manual QA / exploratory    |
                |  * Security scan              |
                |  * Performance / load testing |
                |  * Accessibility audit        |
                |  (~1-2 days)                  |
                +---------------+---------------+
                        +-------+-------+
                      Pass             Fail
                        |                |
                        |         +------+------+
                        |     Critical       Minor
                        |     blocker        issue
                        |         |              |
                        |     Fix required,  Logged as
                        |     new RC tagged  known issue,
                        |     (v2.5.0-rc.2)  proceed
                        |         |              |
                        |     Gate 5 re-runs     |
                        |         |              |
                        | <-------+              |
                        | <----------------------+
                        |
                        v
                +-------------------------------+
                |  GATE 6: Go / No-Go Meeting   |
                |  Attendees:                   |
                |  * Engineering Lead           |
                |  * QA Lead                    |
                |  * Product Manager            |
                |  * DevOps / SRE               |
                |                               |
                |  Review:                      |
                |  * Test results               |
                |  * Known issues list          |
                |  * Rollback plan              |
                +---------------+---------------+
                        +-------+-------+
                       Go            No-Go
                        |                |
                        |         Fix blockers
                        |         New RC -> Gate 5
                        |
                        v
                +-------------------------------+
                |  DEPLOY TO PRODUCTION         |
                |  * Canary release (5% traffic)|
                |  * Monitor error rates        |
                |  * Monitor performance        |
                |  (~15-30 minutes)             |
                +---------------+---------------+
                        +-------+-------+
                    Healthy          Errors spike
                        |                |
                        |         Rollback immediately
                        |         Investigate root cause
                        |         Fix -> new RC cycle
                        |
                        v
                +-------------------------------+
                |  Full rollout (100% traffic)  |
                +---------------+---------------+
                                |
                                v
                +-------------------------------+
                |  POST-DEPLOY                  |
                |  * Smoke tests on production  |
                |  * Monitor dashboards (24-48h)|
                |  * Tag release (v2.5.0)       |
                |  * Close release ticket       |
                |  * Retro if issues found      |
                +-------------------------------+
```

### CI/CD Configuration Example (GitHub Actions)

```yaml
# .github/workflows/e2e.yml
name: E2E Tests
on: [pull_request]

jobs:
  gate-1:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm test                              # unit tests
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npm start & npx wait-on http://localhost:3000
      - run: npx playwright test --grep @smoke     # smoke E2E only
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: test-report
          path: playwright-report/
```

### Parallelization for large suites

```bash
# Split full regression across 4 CI machines
npx playwright test --shard=1/4
npx playwright test --shard=2/4
npx playwright test --shard=3/4
npx playwright test --shard=4/4
```

---

## 13. Gate Reference Summary

| Gate | What runs | Time | Trigger | If fails |
|---|---|---|---|---|
| **Gate 1** | Unit + lint + smoke E2E | ~5-10 min | Every PR | PR blocked, dev fixes |
| **Gate 2** | Code review (human) | Hours to 1 day | After Gate 1 passes | Changes requested, dev updates |
| **Gate 3** | Full regression + integration | ~30-60 min | After merge to develop/staging | Real bug -> hotfix PR, Flaky -> SDET fixes |
| **Gate 4** | Nightly full suite + cross-browser | ~1-2 hours | Cron (every night on main) | Bug ticket filed or infra fix |
| **Gate 5** | Release validation (auto + manual + security + perf) | ~1-2 days | Release candidate tagged | Critical -> new RC, Minor -> proceed with known issue |
| **Gate 6** | Go/No-Go meeting (human decision) | ~30 min | After Gate 5 passes | No-Go -> fix and re-validate |
| **Deploy** | Canary rollout + monitoring | ~15-30 min | After Go decision | Errors spike -> instant rollback |
| **Post-deploy** | Production smoke + monitoring | ~24-48 hours | After full rollout | Issues -> hotfix cycle |

---

## 14. Hotfix—Emergency Production Fix

### What is a hotfix?
A hotfix is an **urgent, small fix** that goes directly to production (or main) **bypassing the normal flow** because something critical is broken in production right now.

### Normal flow vs Hotfix flow

**Normal flow:**
```
feature branch -> PR -> Gate 1 -> code review -> staging -> regression -> main -> release
(takes hours to days)
```

**Hotfix flow:**
```
Production is broken RIGHT NOW (e.g., users can't login)
        |
        v
Developer creates hotfix branch FROM main (not develop)
        |
        v
Writes the minimal fix (smallest possible change)
        |
        v
PR created -> fast code review (1 reviewer, urgent)
        |
        v
Smoke tests run (skip full regression — too slow for emergency)
        |
        v
Merge to main -> deploy to production immediately
        |
        v
Then backport (merge) the fix into develop/staging too
```

### Key characteristics

| Aspect | Normal fix | Hotfix |
|---|---|---|
| **Urgency** | Can wait | Production is broken NOW |
| **Branch from** | `develop` | `main` (or production tag) |
| **Scope** | Can be any size | Smallest possible change |
| **Review** | Full review + all gates | Fast review, minimal gates |
| **Testing** | Full regression | Smoke tests only |
| **Deploy** | Normal release cycle | Immediately to production |
| **After deploy** | Done | Must backport fix to develop |

### Git branching for hotfix

```
main        ----*----*----*----[BUG]----*----(hotfix merged)----
                                 \              /
hotfix/fix-auth  -               fix-commit---+
                                                \
develop     ----*----*----*----*----*-----------(backport merged)---
```

### Detailed hotfix process

```
        Production bug reported
        (users can't login, payments failing, app crashing)
                |
                v
        +-------------------------------+
        |  TRIAGE (5-15 minutes)        |
        |  * Confirm the bug is real    |
        |  * Assess severity            |
        |  * Assign to developer        |
        +---------------+---------------+
                +-------+-------+
            Critical         Not critical
                |                |
                v                v
          Hotfix process    Normal fix process
                |            (goes through regular pipeline)
                v
        +-------------------------------+
        |  CREATE HOTFIX BRANCH         |
        |  git checkout main            |
        |  git checkout -b hotfix/      |
        |    fix-auth-config            |
        +---------------+---------------+
                        |
                        v
        +-------------------------------+
        |  WRITE MINIMAL FIX            |
        |  * Change as few lines as     |
        |    possible                   |
        |  * No refactoring             |
        |  * No "while I'm here" fixes  |
        |  * Only fix the bug           |
        +---------------+---------------+
                        |
                        v
        +-------------------------------+
        |  FAST CODE REVIEW             |
        |  * 1 senior reviewer (not 2+) |
        |  * Review in minutes, not hrs |
        |  * Focus: does it fix the bug |
        |    without breaking anything? |
        +---------------+---------------+
                +-------+-------+
            Approved         Needs changes
                |                |
                |         Fix immediately
                |         (minutes, not hours)
                | <--------------+
                |
                v
        +-------------------------------+
        |  SMOKE TESTS ONLY             |
        |  * Skip full regression       |
        |  * Run critical path tests    |
        |  * Verify the fix works       |
        |  (~5 minutes)                 |
        +---------------+---------------+
                +-------+-------+
              Pass             Fail
                |                |
                |         Fix the fix
                |         Re-run smoke
                | <--------------+
                |
                v
        +-------------------------------+
        |  MERGE TO MAIN & DEPLOY       |
        |  * Merge hotfix -> main       |
        |  * Deploy to production       |
        |  * Monitor error rates        |
        +---------------+---------------+
                +-------+-------+
            Fixed           Still broken
                |                |
                |         Rollback deploy
                |         Investigate more
                |         Another hotfix attempt
                |
                v
        +-------------------------------+
        |  BACKPORT TO DEVELOP          |
        |  * Merge hotfix -> develop    |
        |  * Ensures develop has the    |
        |    fix too                    |
        |  * Prevents regression when   |
        |    develop is next promoted   |
        +---------------+---------------+
                        |
                        v
        +-------------------------------+
        |  POST-MORTEM                  |
        |  * Why did this happen?       |
        |  * Why wasn't it caught by    |
        |    existing tests?            |
        |  * Add test to prevent        |
        |    recurrence                 |
        |  * Update monitoring/alerts   |
        +-------------------------------+
```

### Example scenario

```
Friday 5 PM:
  - Users report: "Login is broken, getting 500 error"
  - Cause: A config change in the last release broke the auth endpoint

Hotfix process:
  1. Developer creates branch: hotfix/fix-auth-config from main
  2. Changes 1 line: fixes the config value
  3. PR created, senior dev reviews in 10 minutes
  4. Smoke tests pass (login works again)
  5. Merged to main, deployed to production
  6. Users can login again within 1 hour
  7. Monday: developer merges the fix into develop branch too
  8. Tuesday: team holds post-mortem, adds a test for this config
```

### When to use hotfix vs normal fix

| Situation | Use |
|---|---|
| Users can't login | Hotfix |
| Payment processing broken | Hotfix |
| App crashes on launch | Hotfix |
| Security vulnerability discovered | Hotfix |
| Data being corrupted | Hotfix |
| Button color is wrong | Normal fix |
| Minor UI glitch | Normal fix |
| Feature not working for 2% of users | Depends on severity |
| Typo in a page | Normal fix |
| Performance slightly degraded | Normal fix (unless severe) |

### Rule of thumb
**Hotfix = production is broken and users/revenue are being impacted right now.** Everything else goes through the normal pipeline.

### Common mistakes with hotfixes

| Mistake | Why it's bad |
|---|---|
| Branching from `develop` instead of `main` | Hotfix includes unfinished features from develop |
| Making the fix too large | More risk, more review time, defeats the purpose |
| Skipping the backport to develop | Next release from develop will reintroduce the bug |
| No post-mortem | Same bug class will happen again |
| Using hotfix for non-critical issues | Erodes the process, team stops taking it seriously |

---

## 15. Testing Pyramid

```
             /\
            /  \
           /    \
          / E2E  \           <-- Few tests, slow, high confidence
         /  tests \             (~10% of tests, catch ~10% of bugs)
        /----------\
       /            \
      / Integration  \      <-- Medium tests, medium speed
     /    tests       \         (~20% of tests, catch ~20% of bugs)
    /------------------\
   /                    \
  /    Unit tests        \  <-- Many tests, fast, low confidence per test
 /                        \     (~70% of tests, catch ~70% of bugs)
/==========================\
```

### The principle
- **Many** unit tests at the base (fast, cheap, run often)
- **Some** integration tests in the middle
- **Few** E2E tests at the top (slow, expensive, run selectively)

Each layer catches a **different class** of bugs. You need all three for comprehensive coverage.

### Anti-patterns to avoid

| Anti-pattern | Problem |
|---|---|
| **Ice cream cone** (many E2E, few unit) | Slow CI, flaky tests, hard to debug failures |
| **No E2E at all** | Unit tests pass but app is broken for users |
| **E2E testing everything** | Takes hours, too flaky, expensive to maintain |
| **No smoke test subset** | Every PR waits 30+ min for full suite |

---

## Pipeline Approach Comparison

Different companies choose different strategies based on their needs:

### Approach 1: Full regression before merge (safest)
```
PR -> smoke -> full regression -> code review -> merge to main
```
- **Used by**: Companies where main must always be deployable
- **Downside**: Slower PR cycle, devs wait 30+ min

### Approach 2: Smoke on PR, regression after merge (fastest)
```
PR -> smoke -> code review -> merge to main -> full regression
```
- **Used by**: Companies prioritizing developer velocity
- **Downside**: Broken code can land on main temporarily

### Approach 3: Staging branch buffer (most common, recommended)
```
PR -> smoke -> code review -> merge to staging -> full regression -> promote to main
```
- **Used by**: Most large companies
- **Benefit**: Fast PR cycle + main stays stable

### What decides the approach?

| Factor | Recommended approach |
|---|---|
| Deploy from main directly to prod? | Must run full suite BEFORE merge |
| Have a staging/develop branch? | Run regression after merge to staging, before main |
| Small team, fast iteration? | Smoke on PR + nightly regression |
| Large team, many PRs daily? | Staging branch buffer |
| Regulated industry (banking, healthcare)? | Everything before merge, no exceptions |

---

## When Tests Fail — Investigation Flow

```
Test fails in CI
      |
      v
SDET investigates
      |
      v
  +---+-----------+
  |               |
Real bug       Flaky test
  |               |
  v               v
File bug       Diagnose root cause:
ticket to      - Animation timing?
dev team       - Network race condition?
               - Test environment issue?
               - Bad selector?
               |
               v
            Fix the test:
            - Add proper waits
            - Use better selectors
            - Add retry logic
            - Fix test data setup
```

### Reporting tools
- **Playwright HTML Report** — screenshots, videos, traces of failures
- **Allure Dashboard** — trends, history, pass rate over time
- **Slack/Teams notifications** — alert team on failures

---

*This guide covers the complete lifecycle of E2E testing in production environments for React web applications, from requirements gathering through deployment and monitoring.*
