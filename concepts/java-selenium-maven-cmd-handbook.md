# Unified CMD Handbook: Java Automation (Maven, TestNG, Selenium)

This handbook serves as a technical reference for executing and managing a Java-based automation framework via the command line. It focuses on the initialisation of environments, dependency management, and advanced execution strategies.

---

## 1. Environment Setup

Ensure the Java Development Kit (JDK) 11+ and Apache Maven are correctly installed and configured.

### Java (JDK 11+)
Verify installation:
```bash
java -version
```

**Setting JAVA_HOME (Permanent)**

*   **Windows (PowerShell Admin):**
    ```powershell
    [System.Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Program Files\Java\jdk-11", "Machine")
    [System.Environment]::SetEnvironmentVariable("Path", $env:Path + ";%JAVA_HOME%\bin", "Machine")
    ```

*   **macOS / Linux (Bash/Zsh):**
    ```bash
    echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 11)' >> ~/.zshrc
    echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.zshrc
    source ~/.zshrc
    ```

### Apache Maven
Verify installation:
```bash
mvn -version
```

**Setting MAVEN_HOME (Permanent)**

*   **Windows (PowerShell Admin):**
    ```powershell
    [System.Environment]::SetEnvironmentVariable("MAVEN_HOME", "C:\Apache\maven", "Machine")
    [System.Environment]::SetEnvironmentVariable("Path", $env:Path + ";%MAVEN_HOME%\bin", "Machine")
    ```

*   **macOS / Linux:**
    ```bash
    echo 'export MAVEN_HOME=/opt/apache-maven' >> ~/.zshrc
    echo 'export PATH=$MAVEN_HOME/bin:$PATH' >> ~/.zshrc
    source ~/.zshrc
    ```

---

## 2. Dependency Management

While traditionally managed in the IDE, you can verify or add dependencies via CMD commands to optimise setup time.

**Add Dependencies (Directly to pom.xml):**
*Note: This requires the `versions-maven-plugin` or manual editing. A common approach to initialise a project with dependencies is using archetypes.*

**Generate a New Project with Archetype:**
```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=automation-framework -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

To manage dependencies manually, ensure your `pom.xml` includes:
```xml
<dependencies>
    <!-- Selenium -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.x.x</version>
    </dependency>
    <!-- TestNG -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.x.x</version>
    </dependency>
    <!-- WebDriverManager -->
    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>5.x.x</version>
    </dependency>
</dependencies>
```

---

## 3. Build Lifecycle & Housekeeping

Maintain a clean build environment to avoid caching issues.

**Standard Clean and Install:**
```bash
mvn clean install
```

**Force Update Snapshots/Dependencies:**
Use this if dependencies are not updating correctly.
```bash
mvn clean install -U
```

**Compile Only (No Test Execution):**
```bash
mvn clean compile -DskipTests
```

---

## 4. Selenium-Specific Execution

Pass system properties to your framework using the `-D` flag. Your code must handle `System.getProperty("key")`.

### Browser Parameters
Execute tests on a specific browser.
```bash
mvn test -Dbrowser=firefox
```

### Headless Mode
Toggle headless execution for CI/CD environments.
```bash
mvn test -Dheadless=true
```

### Driver Executables
Manually point to a specific driver binary if not using WebDriverManager.
```bash
mvn test -Dwebdriver.chrome.driver="/path/to/chromedriver"
```

---

## 5. TestNG Command Suite

Control which tests run directly from Maven.

### Running Specific Suites
Execute a specific `testng.xml` file.
```bash
mvn test -DsuiteXmlFile=testng.xml
```

### Filtering Execution
**By Groups:**
```bash
mvn test -Dgroups="regression,smoke"
```

**By Specific Class:**
```bash
mvn test -Dtest=LoginPageTest
```

**By Specific Method:**
```bash
mvn test -Dtest=LoginPageTest#testValidLogin
```

### Parallel Execution
Override parallelism settings defined in XML.
```bash
mvn test -Ddataproviderthreadcount=3 -DthreadCount=4
```
*Note: Ensure your `maven-surefire-plugin` is configured to accept these properties.*

---

## 6. Advanced Debugging & Grid

### Full Stack Traces
Enable full debug logging for Maven errors.
```bash
mvn test -X
```

### Selenium Grid / Hub
Execute tests against a remote grid.
```bash
mvn test -Dselenium.grid.url="http://localhost:4444/wd/hub"
```

### Custom Reports Directory
Redirect output to a temporary directory (useful for parallel CI jobs).
```bash
mvn test -Dreport.dir="./target/custom-reports"
```

---

## Quick Reference Table

| Goal | Command |
| :--- | :--- |
| **Run All Tests** | `mvn test` |
| **Clean & Update** | `mvn clean install -U` |
| **Specific Browser** | `mvn test -Dbrowser=edge` |
| **Headless Mode** | `mvn test -Dheadless=true` |
| **Specific Suite** | `mvn test -DsuiteXmlFile=sanity.xml` |
| **Specific Method** | `mvn test -Dtest=ClassName#MethodName` |
| **Debug Mode** | `mvn test -X` |
| **Skip Tests** | `mvn install -DskipTests` |
