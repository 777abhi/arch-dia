# QE Exploratory Scenarios with GitHub Copilot Agent

This guide provides practical, scenario-based examples of how Quality Engineers (QEs) can leverage the **GitHub Copilot Agent** (specifically the `@workspace` command) in VS Code for non-coding activities.

While standard Copilot offers code completion, the **Agent** acts as a conversational AI that understands your entire project structure. This makes it an invaluable tool for exploratory testing, analysis, and planning.

---

## Scenario 1: Rapid System Understanding (Onboarding)

**Context:** You are new to the team or picking up a legacy feature you didn't write. You need to understand how the system works without reading thousands of lines of code.

**Traditional Approach:** Manually clicking through folders, guessing entry points, and reading disjointed documentation.

**Copilot Agent Approach:**
Use `@workspace` to ask for a high-level summary of the architecture or specific feature flows.

**Prompt:**
> `@workspace Explain how the 'User Authentication' flow works in this project. Identify the key files involved and how data flows from the UI to the database.`

**Outcome:**
Copilot will analyze the repository and provide a summary, listing the frontend components (e.g., `Login.tsx`), the backend controllers (e.g., `AuthController.ts`), and the database models involved.

---

## Scenario 2: Requirement vs. Implementation Gap Analysis

**Context:** You have a user story that says "Users must be at least 18 years old." You want to check if this validation exists in the code before you even start testing.

**Traditional Approach:** Searching for keywords like "age" or "18" and trying to decipher the logic.

**Copilot Agent Approach:**
Ask `@workspace` to verify specific logic against your understanding.

**Prompt:**
> `@workspace Check the registration logic. Is there any server-side validation that prevents a user under 18 from signing up? Point me to the file and line number.`

**Outcome:**
Copilot might reply: *"Yes, in `UserService.js`, there is a check `if (age < 18) throw new Error(...)`."* Or it might say: *"I see frontend validation in `SignupForm.js`, but I cannot find corresponding server-side validation,"* which immediately identifies a potential bug/gap.

---

## Scenario 3: Smart Test Data Generation

**Context:** You are performing exploratory testing on a "Product Upload" form. You need to know the specific constraints (max length, allowed characters) to create effective negative test cases.

**Traditional Approach:** Trial and error (black-box testing) or asking a developer.

**Copilot Agent Approach:**
Use `@workspace` to extract validation rules directly from the source of truth.

**Prompt:**
> `@workspace Look at the 'Product' model or schema. What are the validation rules for the 'SKU' field? Are there regex patterns or length limits I should test?`

**Outcome:**
Copilot finds the schema definition and tells you: *"The SKU field requires a pattern matching `^[A-Z]{3}-\d{4}$` (e.g., ABC-1234) and is limited to 8 characters."*
*   **Action:** You now know to test "abc-1234" (lowercase), "ABCD-1234" (too long), and "AB-1234" (too short).

---

## Scenario 4: Impact Analysis for Regression

**Context:** A developer modified the `DiscountCalculator` service. You need to know what other parts of the application might be broken by this change.

**Traditional Approach:** Guessing based on feature names or running the entire regression suite (time-consuming).

**Copilot Agent Approach:**
Ask `@workspace` to identify dependencies.

**Prompt:**
> `@workspace Who calls or imports the DiscountCalculator? List all the features or pages that rely on this service so I can target my regression testing.`

**Outcome:**
Copilot lists the usage references: *"The `DiscountCalculator` is used in: 1. `CheckoutPage`, 2. `CartSummary`, and 3. The `PromotionalEmail` job."*
*   **Action:** You focus your exploratory charter specifically on these three areas.

---

## Scenario 5: Decrypting Technical Jargon & Logs

**Context:** During testing, you encounter a cryptic error message in the browser console or server logs.

**Traditional Approach:** Googling the error or pasting it into a generic chatbot without context.

**Copilot Agent Approach:**
Paste the error and ask `@workspace` to explain it *in the context of your specific code*.

**Prompt:**
> `@workspace I'm seeing this error: "Foreign key constraint violation on table 'orders'". Based on the database schema in this repo, what does this actually mean? What data was I likely missing?`

**Outcome:**
Copilot explains: *"This error usually happens when you try to create an order for a `customerId` that doesn't exist in the `Customers` table. In your test, ensure the user is fully created before submitting the order."*

---

## Scenario 6: Test Charter Generation

**Context:** You have 30 minutes to explore a new "Search" feature. You need a plan.

**Traditional Approach:** Writing a plan from scratch based on UI intuition.

**Copilot Agent Approach:**
Ask `@workspace` to generate a charter based on the implementation details.

**Prompt:**
> `@workspace I want to perform exploratory testing on the Search feature implemented in SearchService.ts. What specific edge cases should I look for based on how the search query is constructed? Generate a 30-minute test charter.`

**Outcome:**
Copilot suggests:
*   **Mission:** Explore search resiliency.
*   **Scenarios:**
    *   Test SQL injection attempts (because it sees raw SQL concatenation).
    *   Test empty strings and wildcard characters (`%`, `*`).
    *   Test extremely long strings (buffer overflow risks).
    *   Verify how it handles special characters like emojis.

---

## Summary of Commands

| Intent | Command / Participant | Example |
| :--- | :--- | :--- |
| **Broad Context** | `@workspace` | `@workspace How does the payment flow work?` |
| **Specific File** | Open file + `@workspace` | `@workspace What edge cases are handled in this file?` |
| **Explanation** | `/explain` | Highlight code -> `/explain what this logic does in plain English` |
| **Terminal Errors**| `@terminal` | `@terminal Explain the last error message` |

By using these agent capabilities, you move from "Black Box" testing to "Grey Box" testing—leveraging the code to inform your testing strategy without needing to write the code yourself.
