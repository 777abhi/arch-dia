# Playwright Guide for Manual & Exploratory Quality Engineers

This guide is designed for **Manual and Exploratory Quality Engineers** who want to add Playwright to their toolkit. Even if you don't write code every day, Playwright offers powerful tools to speed up your testing, improve bug reporting, and handle repetitive tasks.

---

## Why Playwright for Manual Testing?

You might think Playwright is just for automation engineers, but it has features that are incredibly useful for manual testing:

*   **⚡ Speed:** Automate the "boring" parts (like logging in or filling out long forms) so you can focus on the tricky exploratory testing.
*   **🎯 Precision:** Use tools to identify the *exact* element that is broken (e.g., "The button with `data-testid=submit` is not working" is better than "The submit button is broken").
*   **📹 Evidence:** Generate code snippets, screenshots, or videos to attach to bug reports, making it easier for developers to reproduce issues.
*   **🌐 Browser Check:** Easily run your steps on Chrome, Firefox, and Safari to check for cross-browser issues without manually repeating the work.

---

## Part 1: Setting up your Environment

Before you can use Playwright, you need a few tools installed.

### 1. Install VS Code
Visual Studio Code (VS Code) is the industry-standard code editor. It has a fantastic extension for Playwright.
*   **Download:** [code.visualstudio.com](https://code.visualstudio.com/)

### 2. Install Node.js
Playwright runs on Node.js. You just need to install it; you don't need to know how to program in it.
*   **Download:** [nodejs.org](https://nodejs.org/en/download/) (Choose the "LTS" version).
*   **Verify:** Open your terminal (Command Prompt or Terminal app) and type `node -v`. You should see a version number (e.g., `v18.x.x`).

### 3. Install the Playwright VS Code Extension
This extension makes using Playwright much easier.
1.  Open VS Code.
2.  Click the **Extensions** icon on the left sidebar (blocks icon).
3.  Search for **"Playwright Test for VSCode"** (by Microsoft).
4.  Click **Install**.

---

## Part 2: Initializing Playwright

Now, let's set up a "playground" folder for your Playwright work.

1.  Create a new folder on your computer (e.g., `my-testing-playground`).
2.  Open this folder in VS Code (`File > Open Folder...`).
3.  Open the **Command Palette** (Cmd+Shift+P on Mac, Ctrl+Shift+P on Windows).
4.  Type `Playwright: Install Playwright` and select it.
5.  Check the browsers you want (Chromium, Firefox, WebKit) and click **OK**.
    *   *Note: This might take a few minutes as it downloads the browser binaries.*

---

## Part 3: The Toolkit (Features & Commands)

Here are the specific tools you will use as a Manual QE.

### 🎥 The Code Generator (Codegen)

This is the most powerful tool for you. It records your interactions and generates code. Even if you don't save the code, it helps you replay actions.

**How to use it:**
1.  Open the VS Code terminal (`Terminal > New Terminal`).
2.  Run the command:
    ```bash
    npx playwright codegen <url>
    ```
    *Example:* `npx playwright codegen https://www.saucedemo.com`

**What happens?**
*   Two windows will open:
    1.  **Browser Window:** Where you interact with the website.
    2.  **Playwright Inspector:** Where the code is being written in real-time.

**Try this:**
*   Click inputs, type text, and click buttons in the browser. Watch the Inspector generate the steps!
*   **Hit Record:** You can pause and resume recording in the Inspector.

### 🔍 Pick Locator

When writing a bug report, pointing to the exact element is helpful.

**How to use it:**
1.  In the **Playwright Inspector** (during codegen), click the **"Explore"** (cursor) icon.
2.  Hover over any element on the webpage.
3.  The Inspector will show you the "Locator" (e.g., `getByRole('button', { name: 'Login' })` or `.submit-btn`).
4.  Copy this locator and paste it into your bug ticket!

---

## Part 4: Practical Use Cases

### Use Case 1: "Skip the Boring Stuff"
**Scenario:** You need to test the "Checkout" page, but you have to login and add items to the cart first—every single time.

**Solution:**
1.  Run `npx playwright codegen https://mysite.com`.
2.  Record yourself logging in and adding items to the cart.
3.  Copy the generated code.
4.  Save it in a file like `setup.spec.ts`.
5.  Now, you can run this test to quickly get the browser to the state you need, then take over manually! (You can use `page.pause()` in the code to stop it where you want).

### Use Case 2: Perfect Reproduction Steps
**Scenario:** A developer says "I can't reproduce that bug."

**Solution:**
1.  Turn on Codegen.
2.  Perform the *exact* steps to trigger the bug.
3.  Copy the code generated.
4.  Paste it into the Jira ticket/bug report under "Reproduction Steps (Playwright Script)".
5.  The developer can run that script and see exactly what you did.

### Use Case 3: Cross-Browser Sanity Check
**Scenario:** You manually tested on Chrome. Does it work on Safari (WebKit)?

**Solution:**
1.  Save your recorded steps as a test file.
2.  Run the test against all browsers using the VS Code extension sidebar.
3.  Green checkmarks? You just verified 3 browsers in the time it took to test one!

---

## Part 5: Cheat Sheet

Here are the commands you will use most often. Run these in the VS Code Terminal.

| Command | Description |
| :--- | :--- |
| `npx playwright codegen` | Opens a blank browser to start recording. |
| `npx playwright codegen <url>` | Opens the specific URL to start recording. |
| `npx playwright test` | Runs all tests (headless - no browser UI seen). |
| `npx playwright test --ui` | Opens UI Mode (great for watching tests run visually). |
| `npx playwright show-report` | Shows a HTML report of the last run. |

---

## 5 Tips for Success

1.  **Don't Fear the Code:** You don't need to understand every line. Focus on the *actions* (click, fill, press).
2.  **Use Locators:** Learning to read locators (like `text='Submit'`) will make you a better tester because you'll understand how the page is built.
3.  **Short & Sweet:** Keep your recordings short. One specific workflow per recording is easier to manage.
4.  **Ask for Help:** If the recorder gets stuck, ask a developer or automation engineer. It's a great way to collaborate!
5.  **Experiment:** Try breaking things! Use Playwright to type super fast or click multiple times to see how the app handles it.
