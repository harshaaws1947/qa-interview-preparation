# Test Framework Design Interview Questions

## Framework Fundamentals

### 1. What is a Test Automation Framework?
A **test automation framework** is a set of guidelines, coding standards, tools, and practices that provide structure for creating and maintaining automated tests.

**Benefits:**
- Code reusability
- Easy maintenance
- Consistent test execution
- Better reporting
- Faster test development

### 2. What are different types of test automation frameworks?

| Framework Type | Description | Best For |
|----------------|-------------|----------|
| Linear/Record-Playback | Sequential test scripts | Simple, one-time tests |
| Modular | Tests divided into modules | Medium projects |
| Data-Driven | Test data separated from logic | Same tests, different data |
| Keyword-Driven | Actions as keywords | Non-technical testers |
| Hybrid | Combination of above | Large enterprise projects |
| BDD | Behavior-driven with Gherkin | Business collaboration |

### 3. What is Page Object Model (POM)?
**POM** is a design pattern where each web page is represented as a class containing elements and methods.

**Benefits:**
- Separation of test logic and page elements
- Code reusability
- Easy maintenance
- Reduced code duplication

```java
public class LoginPage {
    private WebDriver driver;

    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "login-btn")
    private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public void enterUsername(String username) {
        usernameField.sendKeys(username);
    }

    public void enterPassword(String password) {
        passwordField.sendKeys(password);
    }

    public HomePage clickLogin() {
        loginButton.click();
        return new HomePage(driver);
    }

    // Fluent method chaining
    public LoginPage login(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        clickLogin();
        return this;
    }
}
```

### 4. Explain the framework architecture you have built

**Typical Framework Structure:**
```
test-automation-framework/
├── src/main/java/
│   ├── pages/              # Page Objects
│   ├── utils/              # Utilities (Wait, Screenshot, Config)
│   ├── config/             # Configuration files
│   └── constants/          # Constants and Enums
├── src/test/java/
│   ├── base/               # Base Test class
│   ├── tests/              # Test classes
│   └── listeners/          # TestNG Listeners
├── test-data/              # Test data (JSON, Excel, CSV)
├── reports/                # Test reports
├── screenshots/            # Failure screenshots
├── pom.xml                 # Maven dependencies
├── testng.xml              # TestNG configuration
└── README.md
```

### 5. What design patterns do you use in test automation?

**1. Page Object Model**
- Encapsulates page elements and actions

**2. Singleton Pattern**
- Single instance for WebDriver, ConfigReader
```java
public class DriverManager {
    private static WebDriver driver;

    private DriverManager() {}

    public static WebDriver getDriver() {
        if (driver == null) {
            driver = new ChromeDriver();
        }
        return driver;
    }
}
```

**3. Factory Pattern**
- Create objects without exposing creation logic
```java
public class DriverFactory {
    public static WebDriver createDriver(String browser) {
        switch(browser.toLowerCase()) {
            case "chrome": return new ChromeDriver();
            case "firefox": return new FirefoxDriver();
            case "edge": return new EdgeDriver();
            default: throw new IllegalArgumentException("Invalid browser");
        }
    }
}
```

**4. Builder Pattern**
- Build complex objects step by step
```java
User user = new User.Builder()
    .setName("John")
    .setEmail("john@test.com")
    .setAge(30)
    .build();
```

## Data-Driven Testing

### 6. How do you implement Data-Driven Testing?

**Using TestNG DataProvider:**
```java
@DataProvider(name = "loginData")
public Object[][] getLoginData() {
    return new Object[][] {
        {"validuser", "validpass", true},
        {"invaliduser", "invalidpass", false},
        {"", "password", false}
    };
}

@Test(dataProvider = "loginData")
public void testLogin(String username, String password, boolean expected) {
    loginPage.login(username, password);
    assertEquals(homePage.isLoggedIn(), expected);
}
```

**Reading from Excel (Apache POI):**
```java
public List<Map<String, String>> readExcelData(String filePath) {
    List<Map<String, String>> data = new ArrayList<>();
    FileInputStream fis = new FileInputStream(filePath);
    Workbook workbook = new XSSFWorkbook(fis);
    Sheet sheet = workbook.getSheetAt(0);

    Row headerRow = sheet.getRow(0);
    for (int i = 1; i <= sheet.getLastRowNum(); i++) {
        Row row = sheet.getRow(i);
        Map<String, String> rowData = new HashMap<>();
        for (int j = 0; j < row.getLastCellNum(); j++) {
            rowData.put(
                headerRow.getCell(j).getStringCellValue(),
                row.getCell(j).getStringCellValue()
            );
        }
        data.add(rowData);
    }
    return data;
}
```

**Reading from JSON:**
```java
public User getUserFromJson(String filePath) {
    ObjectMapper mapper = new ObjectMapper();
    return mapper.readValue(new File(filePath), User.class);
}
```

### 7. How do you manage test configuration?

**Using Properties File:**
```java
// config.properties
browser=chrome
base.url=https://app.example.com
implicit.wait=10
explicit.wait=20

// ConfigReader.java
public class ConfigReader {
    private static Properties properties;

    static {
        try {
            properties = new Properties();
            properties.load(new FileInputStream("config/config.properties"));
        } catch (IOException e) {
            throw new RuntimeException("Failed to load config");
        }
    }

    public static String getProperty(String key) {
        return properties.getProperty(key);
    }
}
```

**Using Environment Variables:**
```java
public class Config {
    public static String getBaseUrl() {
        String env = System.getProperty("env", "qa");
        return switch(env) {
            case "qa" -> "https://qa.example.com";
            case "staging" -> "https://staging.example.com";
            case "prod" -> "https://example.com";
            default -> throw new IllegalArgumentException("Invalid env");
        };
    }
}
```

## Reporting and Logging

### 8. How do you implement reporting in your framework?

**Extent Reports:**
```java
public class ExtentManager {
    private static ExtentReports extent;
    private static ThreadLocal<ExtentTest> test = new ThreadLocal<>();

    public static ExtentReports getInstance() {
        if (extent == null) {
            extent = new ExtentReports();
            ExtentSparkReporter reporter = new ExtentSparkReporter("reports/report.html");
            extent.attachReporter(reporter);
        }
        return extent;
    }

    public static ExtentTest createTest(String name) {
        ExtentTest extentTest = getInstance().createTest(name);
        test.set(extentTest);
        return extentTest;
    }

    public static ExtentTest getTest() {
        return test.get();
    }
}
```

**TestNG Listener for Reporting:**
```java
public class TestListener implements ITestListener {
    @Override
    public void onTestStart(ITestResult result) {
        ExtentManager.createTest(result.getMethod().getMethodName());
    }

    @Override
    public void onTestSuccess(ITestResult result) {
        ExtentManager.getTest().pass("Test passed");
    }

    @Override
    public void onTestFailure(ITestResult result) {
        ExtentManager.getTest().fail(result.getThrowable());
        // Capture screenshot
        String screenshot = captureScreenshot();
        ExtentManager.getTest().addScreenCaptureFromPath(screenshot);
    }
}
```

### 9. How do you implement logging?

```java
// Using Log4j2
public class LogUtils {
    private static final Logger logger = LogManager.getLogger(LogUtils.class);

    public static void info(String message) {
        logger.info(message);
    }

    public static void error(String message, Throwable t) {
        logger.error(message, t);
    }

    public static void debug(String message) {
        logger.debug(message);
    }
}

// log4j2.xml
<Configuration>
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
        <File name="File" fileName="logs/test.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %-5level - %msg%n"/>
        </File>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="File"/>
        </Root>
    </Loggers>
</Configuration>
```

## CI/CD Integration

### 10. How do you integrate tests with CI/CD?

**Jenkins Pipeline:**
```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test -Denv=qa -Dbrowser=chrome'
            }
        }

        stage('Report') {
            steps {
                publishHTML([
                    reportDir: 'reports',
                    reportFiles: 'report.html',
                    reportName: 'Test Report'
                ])
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'reports/**', fingerprint: true
        }
        failure {
            emailext subject: 'Test Failed',
                     body: 'Check Jenkins for details',
                     recipientProviders: [developers()]
        }
    }
}
```

**GitHub Actions:**
```yaml
name: Test Automation

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Set up JDK
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'

    - name: Run Tests
      run: mvn test -Dbrowser=chrome -Dheadless=true

    - name: Upload Reports
      uses: actions/upload-artifact@v3
      with:
        name: test-reports
        path: reports/
```

## Parallel Execution

### 11. How do you run tests in parallel?

**TestNG Parallel Execution:**
```xml
<!-- testng.xml -->
<suite name="Parallel Suite" parallel="methods" thread-count="4">
    <test name="Chrome Tests">
        <parameter name="browser" value="chrome"/>
        <classes>
            <class name="tests.LoginTest"/>
            <class name="tests.HomeTest"/>
        </classes>
    </test>
</suite>
```

**Thread-Safe WebDriver:**
```java
public class DriverManager {
    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();

    public static WebDriver getDriver() {
        return driver.get();
    }

    public static void setDriver(WebDriver webDriver) {
        driver.set(webDriver);
    }

    public static void quitDriver() {
        if (driver.get() != null) {
            driver.get().quit();
            driver.remove();
        }
    }
}
```

### 12. How do you handle test dependencies?

```java
// Using TestNG dependencies
@Test
public void login() {
    // Login test
}

@Test(dependsOnMethods = "login")
public void createOrder() {
    // Depends on successful login
}

// Using priority
@Test(priority = 1)
public void test1() {}

@Test(priority = 2)
public void test2() {}
```

## Advanced Topics

### 13. How do you handle dynamic elements?

```java
// Custom wait for dynamic elements
public WebElement waitForElement(By locator, int timeout) {
    return new FluentWait<>(driver)
        .withTimeout(Duration.ofSeconds(timeout))
        .pollingEvery(Duration.ofMillis(500))
        .ignoring(NoSuchElementException.class)
        .ignoring(StaleElementReferenceException.class)
        .until(ExpectedConditions.visibilityOfElementLocated(locator));
}

// Retry mechanism
public void clickWithRetry(WebElement element, int maxRetries) {
    int attempt = 0;
    while (attempt < maxRetries) {
        try {
            element.click();
            return;
        } catch (StaleElementReferenceException e) {
            attempt++;
        }
    }
    throw new RuntimeException("Failed after " + maxRetries + " attempts");
}
```

### 14. How do you maintain test independence?

**Principles:**
1. Each test should set up its own data
2. Tests should not depend on other tests
3. Clean up after test execution
4. Use unique test data

```java
@BeforeMethod
public void setUp() {
    driver = DriverFactory.createDriver("chrome");
    testUser = DataGenerator.createUniqueUser();
}

@AfterMethod
public void tearDown() {
    // Clean up test data
    apiClient.deleteUser(testUser.getId());
    DriverManager.quitDriver();
}
```

### 15. How do you handle flaky tests?

**Strategies:**
1. **Retry Mechanism:**
```java
@Test(retryAnalyzer = RetryAnalyzer.class)
public void flakyTest() {}

public class RetryAnalyzer implements IRetryAnalyzer {
    private int count = 0;
    private static final int MAX_RETRY = 2;

    @Override
    public boolean retry(ITestResult result) {
        if (count < MAX_RETRY) {
            count++;
            return true;
        }
        return false;
    }
}
```

2. **Better Waits:**
```java
// Instead of Thread.sleep
new WebDriverWait(driver, Duration.ofSeconds(10))
    .until(ExpectedConditions.elementToBeClickable(element));
```

3. **Isolate Tests:** Don't share state between tests
4. **Stable Test Data:** Use dedicated test environments

---

## Quick Tips for Interview

1. **Know POM thoroughly** - Explain with real examples
2. **Understand design patterns** - Singleton, Factory, Builder
3. **CI/CD experience** - Jenkins, GitHub Actions
4. **Parallel execution** - ThreadLocal, TestNG configuration
5. **Reporting** - Extent Reports, Allure
6. **Handling challenges** - Dynamic elements, flaky tests
