# Effective Code Generation & Prompting for Quality Engineering

This guide is designed for Principal Quality Engineers and SDETs seeking to leverage Large Language Models (LLMs) to generate production-ready, maintainable, and high-performance test automation code. It moves beyond basic code snippets to focus on architectural patterns, advanced prompting strategies, and long-term codebase health.

**Note:** This guide utilises British English spelling conventions (e.g., *optimise*, *behaviour*, *analyse*).

---

## 1. Automation Strategy: Modular & Maintainable Scripts

Generating a standalone script is easy; generating a scalable framework requires specific architectural instructions. When working with modern frameworks like Playwright or Cypress, always enforce the **Page Object Model (POM)** to separate selectors from test logic.

### The Page Object Model (POM) Approach

Avoid prompting for "tests" directly. Instead, prompt for the building blocks of your framework: Page Objects, Fixtures, and Utility classes.

#### ❌ Before: The "Quick Fix" Prompt
> "Write a Playwright test to log in to a website with a username and password."

*Result:* A flat, brittle script with hardcoded selectors and no reusability.

#### ✅ After: The Architectural Prompt
> "Act as a Principal SDET. Create a **Playwright Page Object Model (POM)** class for a `LoginPage` using **TypeScript**.
>
> **Requirements:**
> 1. Define locators as private readonly properties.
> 2. Implement a strictly typed method `login(credentials: UserCredentials): Promise<void>`.
> 3. Use user-facing locators (e.g., `getByRole`, `getByLabel`) over CSS/XPath where possible to improve accessibility testing.
> 4. Do not include assertions inside the Page Object; return values or elements for the test to assert against.
> 5. Adhere to strict ESLint standards."

*Result:* A modular, strictly typed class ready for enterprise integration.

---

## 2. Advanced Prompting Methods

To derive complex test logic from requirements, use advanced prompting techniques that guide the LLM's reasoning process.

### Role-Based Prompting
Assigning a specific persona primes the model to access relevant domain knowledge and coding standards.

* **Example:** "You are a **Security Quality Engineer**. Review the following API test plan and suggest three additional test cases focusing on OWASP Top 10 vulnerabilities, specifically Broken Object Level Authorization (BOLA)."

### Chain-of-Thought (CoT) Prompting
Encourage the model to "think out loud" before generating code. This is crucial for complex logical flows, such as e-commerce checkouts or multi-step form validations.

#### Prompt Example:
> "We need to test a guest checkout flow for an e-commerce site. The steps are:
> 1. Add item to cart.
> 2. Proceed to checkout.
> 3. Select 'Guest Checkout'.
> 4. Enter shipping details.
> 5. Verify shipping cost calculation based on postcode.
>
> **Task:** strict step-by-step reasoning plan for how to verify the shipping cost *before* you write the Playwright code. Consider how we should handle the asynchronous nature of the API call that calculates shipping."

### Few-Shot Prompting
Provide examples (shots) of your team's coding style or custom libraries to ensure the output matches your existing codebase.

#### Prompt Example:
> "We use a custom wrapper for API calls.
>
> **Example 1:**
> `await api.get('/users').expectStatus(200);`
>
> **Example 2:**
> `await api.post('/login', payload).expectBodyContains('token');`
>
> **Task:**
> Using the pattern above, write a test step to `PUT` an update to a user profile and verify that the response time is under 500ms."

---

## 3. Test Data & Edge Cases

LLMs excel at generating high-fidelity synthetic data. Use them to uncover edge cases that humans often overlook.

### AI-Driven Boundary Analysis

Instead of manually crafting JSON payloads, ask the LLM to generate them based on boundary heuristics.

#### ❌ Before:
> "Give me a JSON for a user."

#### ✅ After:
> "Generate a JSON dataset for a 'User Registration' API endpoint.
>
> **Requirements:**
> 1. **Happy Path:** One valid user.
> 2. **Edge Cases:**
>    - `email`: A string of 255 characters (max length).
>    - `age`: 0, 18 (boundary), and -1 (invalid).
>    - `name`: String containing SQL injection characters (e.g., `' OR '1'='1`).
>    - `bio`: String containing 4-byte unicode characters (emojis).
>
> Output as a generic JSON array."

---

## 4. Maintenance & Optimisation

Legacy suites often suffer from flakiness (unreliable tests) and poor performance. Use LLMs as a static analysis tool to refactor and stabilise code.

### Detecting Flaky Tests
Flakiness often stems from hard-coded waits (`waitForTimeout`) or reliance on unstable selectors.

#### Prompt Example for Refactoring:
> "Analyse the following Playwright test code. Identify any usage of `page.waitForTimeout()` or non-resilient selectors (like long CSS chains).
>
> **Task:** Refactor the code to use **Auto-waiting locators** (e.g., `expect(locator).toBeVisible()`) and Playwright's API assertions. Explain why the changes will reduce flakiness."

### Performance Optimisation
Ask the LLM to identify bottlenecks, such as serial execution of independent tests or unnecessary UI interactions.

#### Prompt Example:
> "Review this test suite. We are currently logging in via the UI `beforeEach` test.
>
> **Task:** Rewrite the setup to perform authentication via an **API Request context** and inject the storage state into the browser context. This should bypass the UI login form to speed up execution."

---

## Quick Reference Summary

| Technique | Best Used For | Key Command |
| :--- | :--- | :--- |
| **POM Strategy** | New feature development | "Create a Page Object class with strictly typed methods..." |
| **Role-Based** | specialised testing (Security, A11y) | "Act as a [Role]..." |
| **Chain-of-Thought** | Complex logic/workflows | "Reason step-by-step before coding..." |
| **Few-Shot** | Matching team style | "Here are three examples of our pattern..." |
| **Refactoring** | Legacy code cleanup | "Replace hard waits with auto-waiting assertions..." |
