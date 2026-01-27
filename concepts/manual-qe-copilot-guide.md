# GitHub Copilot Guide for Manual & Exploratory Quality Engineers

This guide is designed for **Manual and Exploratory Quality Engineers** who are starting to use GitHub Copilot in VS Code. It focuses on using Copilot as a **"Pair Tester"** and **"Knowledge Assistant"** to enhance your daily workflow without requiring immediate coding skills.

The goal is to help you familiarize yourself with VS Code and AI assistance, gradually building the technical understanding you will need for your future transition into automation.

---

## Prerequisite: The Setup

Before starting your day, ensure you have the following in VS Code:
1.  **GitHub Copilot Chat Extension**: This is your primary interface. It allows you to chat with Copilot just like you would with a colleague.
2.  **Workspace**: Open the repository or folder you are working on. Even if you aren't coding, having the files open allows Copilot to understand the context of your project.

---

## Daily Workflow: A Day in the Life

Here is how you can integrate GitHub Copilot into your activities throughout the day.

### 🌅 09:00 AM: Catch-up & Requirement Analysis

**Goal:** Understand what needs to be tested and identify ambiguity early.

Instead of reading through dense tickets or documentation in isolation, use Copilot to summarize and clarify.

**Tasks:**
1.  **Summarize User Stories:** Paste a complex User Story or Requirement into the Chat.
    *   **Prompt:** *"Summarize this user story into a bulleted list of key functionalities I need to verify. Highlight any missing acceptance criteria."*
2.  **Clarify Technical Terms:** If a ticket mentions "API rate limiting" or "JWT tokens", ask Copilot to explain it in simple terms.
    *   **Prompt:** *"Explain 'idempotency' in the context of API testing as if I were a manual tester. Why is it important?"*
3.  **Risk Assessment:**
    *   **Prompt:** *"I am testing a new 'Password Reset' feature. What are the top security risks and edge cases I should focus on?"*

### ☀️ 11:00 AM: Test Planning & Chartering

**Goal:** Brainstorm test ideas and structure your exploratory sessions.

**Tasks:**
1.  **Generate Test Charters:** Create a focus for your exploratory session.
    *   **Prompt:** *"Create a Test Charter for a 60-minute exploratory session on the 'Checkout Page'. Focus on 'User Experience' and 'Error Handling'."*
2.  **Brainstorm Scenarios:** Don't start from a blank page.
    *   **Prompt:** *"List 10 test scenarios for a 'Search Bar'. Include 3 happy paths, 4 edge cases (like special characters), and 3 negative scenarios."*
3.  **Heuristic Suggestions:**
    *   **Prompt:** *"Suggest testing heuristics (like Goldilocks or SFDIPOT) that would be useful for testing a 'File Upload' feature."*

### 🥪 01:00 PM: Test Data Generation

**Goal:** Quickly create diverse and realistic data for testing.

**Tasks:**
1.  **Generate Form Data:**
    *   **Prompt:** *"Generate a table of 5 test users. Include fields: First Name, Last Name, Email (mix of valid and invalid formats), and Phone Number."*
2.  **Create Specific Formats:**
    *   **Prompt:** *"Give me 5 examples of valid JSON for a 'Product' object, and 3 examples of broken JSON to test error handling."*
3.  **Boundary Values:**
    *   **Prompt:** *"What are the boundary values I should test for a 'Age' input field that accepts 18-65?"* (Copilot will suggest 17, 18, 19... 64, 65, 66).

### 🕑 03:00 PM: Execution & Bug Reporting

**Goal:** Execute tests and report issues clearly and professionally.

**Tasks:**
1.  **Analyze Errors:** If you see an error message or a log entry in the console (even if you don't fully understand it), paste it into Copilot.
    *   **Prompt:** *"I got this error in the browser console: `Uncaught TypeError: Cannot read property 'map' of undefined`. What does this mean and what might have caused it?"*
2.  **Format Bug Reports:**
    *   **Prompt:** *"I found a bug where clicking 'Submit' with an empty field crashes the page. Draft a professional bug report with sections: Title, Steps to Reproduce, Expected Result, and Actual Result."*
3.  **Refine Bug Titles:**
    *   **Prompt:** *"Rewrite this bug title to be more descriptive: 'Login doesn't work'."*

### 🌇 04:30 PM: Learning & Preparation (The "Future You")

**Goal:** Gradually familiarize yourself with the code and technical concepts.

**Tasks:**
1.  **Code Walkthrough:** Open a file in the repository (e.g., a `.ts` or `.java` file) that relates to what you tested.
    *   **Prompt:** *"Explain what this file does in high-level English. I don't know code yet, so keep it simple."*
2.  **Understand Logic:** Highlight a small block of code (like an `if/else` statement).
    *   **Prompt:** *"What logic is this specific block checking? Under what conditions does it go to the 'else' part?"*
3.  **Syntax Familiarization:**
    *   **Prompt:** *"What does `===` mean in this file? How is it different from `==`?"*

---

## 5 Tips for Success

1.  **Context is King:** Copilot knows about the file you have open. If you are testing the "Login" feature, open the `login.js` or `login.html` file (even if you don't edit it) to give Copilot context.
2.  **Iterate:** If the first answer isn't what you wanted, rephrase. * "That's too technical. Explain it simply."* or *"Give me more negative test cases."*
3.  **Use Shortcuts:** Type `/` in the Chat input to see commands like `/explain` which is great for learning.
4.  **Be Curious:** Use Copilot to "look under the hood". It removes the fear of asking "stupid questions".
5.  **Don't Paste Secrets:** Never paste real customer data, passwords, or API keys into the Chat. Use placeholders like `[API_KEY]`.
