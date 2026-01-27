# Playwright Guide for Manual & Exploratory Quality Engineers

This guide is designed for **Manual and Exploratory Quality Engineers** who want to add Playwright to their toolkit. Even if you don't write code every day, Playwright offers powerful tools to speed up your testing, improve bug reporting, and handle repetitive tasks.

---

## 💰 The ROI: Why You Should Care

Before we dive in, let's look at the time you can save. Imagine running these common tasks just **5 times a day**:

| Task Category | Manual Time | Playwright Time | Time Saved (Per Run) | **Annual Savings** (approx.) |
| :--- | :--- | :--- | :--- | :--- |
| **Login & Navigate** | 45 sec | 5 sec | **40 sec** | ⏲️ **16 hours/year** |
| **Fill Long Form** | 3 min | 15 sec | **~2.5 min** | ⏲️ **50 hours/year** |
| **Create 10 Users** | 15 min | 1 min | **14 min** | ⏲️ **280 hours/year** |
| **Cross-Browser Check** | 12 min | 45 sec | **11+ min** | ⏲️ **220 hours/year** |
| **Regression Smoke** | 60 min | 5 min | **55 min** | ⏲️ **1,100 hours/year** |

*By automating just the "boring stuff", you could free up weeks of time every year for high-value exploratory testing!*

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

## Part 5: 20 More Ways to Supercharge Your Testing

Beyond the basics, here are 20 practical scenarios where Playwright can be your superpower.

### 🏗️ Data & State Management
1.  **Bulk User Creation:** Record a script to sign up a new user. Run it in a loop to create 10 test users in seconds.
    ```typescript
    // Create 10 users in a loop
    for (let i = 0; i < 10; i++) {
      await page.goto('https://example.com/signup');
      await page.fill('#username', `testuser_${i}`);
      await page.fill('#password', 'password123');
      await page.click('#submit');
    }
    ```
2.  **Clean Slate Testing:** Create a "teardown" script that deletes all the items you created during your session, resetting the environment for the next round.
3.  **Role Switching:** Create separate setup scripts for `Admin`, `Editor`, and `Viewer`. Run one to instantly jump into that persona.
    ```typescript
    // setup-admin.ts
    await page.goto('https://example.com/login');
    await page.fill('#user', 'admin');
    await page.fill('#pass', 'adminPass');
    await page.click('#login-btn');
    await page.context().storageState({ path: 'admin-auth.json' });
    ```
4.  **Complex Form Filling:** For forms with 50+ fields (like insurance apps), record filling it once. Re-run it whenever you need to test the "Submit" button logic.
5.  **Cookie Injection:** Instead of logging in via UI, use a script to inject a valid authentication cookie to bypass the login screen entirely.
    ```typescript
    await context.addCookies([{
      name: 'session_id',
      value: '123456abcdef',
      domain: 'example.com',
      path: '/'
    }]);
    await page.reload();
    ```

### 🎨 UI & Visuals
6.  **Responsive Check:** Set the viewport size in your script to instantly see how the page looks on a mobile screen.
    ```typescript
    // Simulate iPhone 12
    await page.setViewportSize({ width: 390, height: 844 });
    ```
7.  **Dark Mode Verification:** Force the browser into dark mode to verify styles without changing your OS settings.
    ```typescript
    await page.emulateMedia({ colorScheme: 'dark' });
    ```
8.  **Full Page Screenshots:** Capture an entire scrolling page—perfect for design reviews.
    ```typescript
    await page.screenshot({ path: 'full-page.png', fullPage: true });
    ```
9.  **Text Localization:** Script a flow that switches languages and takes a screenshot of the header, helping you spot layout breaks in German or French.
10. **Element Visibility:** Use the Inspector to check if an element is technically "visible" or just hidden by CSS.
    ```typescript
    const isVisible = await page.isVisible('#hidden-button');
    console.log(`Is button visible? ${isVisible}`);
    ```

### ⚡ Workflow Accelerators
11. **Pagination Jumping:** Need to test page 50 of a list? Script clicking "Next" 49 times so you don't have to.
    ```typescript
    for (let i = 0; i < 49; i++) {
      await page.click('.next-page-btn');
      await page.waitForLoadState('networkidle');
    }
    ```
12. **Cart Loading:** Script adding 1 of every product type to the cart to test the "Checkout" summary calculation.
13. **Deep Linking:** If your app doesn't support direct URLs to modals, write a script to navigate and open the specific modal you need to test.
14. **Email Verification:** Automate the "Request Password Reset" action so you can focus solely on the email delivery and link behavior.
15. **Search Permutations:** Create a list of 20 search terms and loop through them, taking a screenshot of the results for each.
    ```typescript
    const terms = ['apple', 'banana', '<script>alert(1)</script>'];
    for (const term of terms) {
      await page.fill('#search', term);
      await page.press('#search', 'Enter');
      await page.screenshot({ path: `search-${term}.png` });
    }
    ```

### 🧪 Edge Cases & Stress
16. **Network Throttling:** Use Playwright to simulate "Slow 3G" network conditions while you manually interact with the page to test loading states.
    ```typescript
    // Connect to Chrome DevTools Protocol
    const client = await page.context().newCDPSession(page);
    await client.send('Network.emulateNetworkConditions', {
      offline: false,
      latency: 500, // ms
      downloadThroughput: 500 * 1024 / 8, // 500 kbps
      uploadThroughput: 500 * 1024 / 8,
    });
    ```
17. **Offline Mode:** Simulate going offline to see how the app handles a lost connection during a form submission.
    ```typescript
    await context.setOffline(true);
    // Try clicking submit now
    await page.click('#submit');
    ```
18. **Time Travel:** Override the browser's timezone or geolocation to test how the app behaves in Tokyo or London without using a VPN.
    ```typescript
    await context.grantPermissions(['geolocation']);
    await context.setGeolocation({ latitude: 35.6895, longitude: 139.6917 }); // Tokyo
    await page.emulateTimezone('Asia/Tokyo');
    ```
19. **Console Log Monitoring:** Run a test that listens for console errors and fails if any red text appears.
    ```typescript
    page.on('console', msg => {
      if (msg.type() === 'error')
        console.log(`Error text: "${msg.text()}"`);
    });
    ```
20. **Rapid Clicking:** Write a loop to click a "Submit" button 10 times in 1 second to test for race conditions or duplicate submissions.
    ```typescript
    for (let i = 0; i < 10; i++) {
      page.click('#submit-btn'); // No await means "fire and forget" (fast!)
    }
    ```

---

## Part 6: Cheat Sheet

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
