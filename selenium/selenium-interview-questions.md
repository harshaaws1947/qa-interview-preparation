# Selenium Interview Questions for QA/SDET

## Selenium Basics

### 1. What is Selenium and its components?
**Selenium** is an open-source test automation framework for web applications.

**Components:**
- **Selenium WebDriver**: Browser automation API
- **Selenium IDE**: Record and playback tool (browser extension)
- **Selenium Grid**: Distributed test execution

### 2. What are the advantages of Selenium?
- Open source and free
- Supports multiple browsers (Chrome, Firefox, Edge, Safari)
- Supports multiple programming languages (Java, Python, C#, JavaScript)
- Supports multiple OS (Windows, Mac, Linux)
- Large community support
- Can be integrated with testing frameworks (TestNG, JUnit)

### 3. What are the limitations of Selenium?
- Cannot automate desktop applications
- Cannot automate mobile apps (need Appium)
- No built-in reporting (need ExtentReports, Allure)
- Cannot handle captcha/OTP
- No official support (community-driven)

### 4. What are the different locator strategies in Selenium?

| Locator | Syntax | When to Use |
|---------|--------|-------------|
| ID | `By.id("id")` | Most preferred (fastest) |
| Name | `By.name("name")` | Good for forms |
| Class Name | `By.className("class")` | Single class names |
| Tag Name | `By.tagName("tag")` | For specific HTML elements |
| Link Text | `By.linkText("text")` | For anchor tags |
| Partial Link Text | `By.partialLinkText("part")` | Partial link matching |
| CSS Selector | `By.cssSelector("css")` | Complex selections |
| XPath | `By.xpath("xpath")` | Most flexible |

### 5. What is the difference between findElement and findElements?

| findElement() | findElements() |
|---------------|----------------|
| Returns single WebElement | Returns List<WebElement> |
| Throws NoSuchElementException if not found | Returns empty list if not found |
| Finds first matching element | Finds all matching elements |

## Locators Deep Dive

### 6. Explain XPath with examples

**Absolute XPath:** Full path from root
```
/html/body/div/form/input[1]
```

**Relative XPath:** Starts from anywhere
```
//input[@id='username']
//div[@class='login']//button
```

**XPath Functions:**
```xpath
//input[contains(@id,'user')]
//input[starts-with(@id,'user')]
//input[text()='Submit']
//div[normalize-space()='Login']
```

**XPath Axes:**
```xpath
//div[@id='parent']/child::span
//input[@id='field']/following-sibling::button
//button[@id='submit']/preceding-sibling::input
//input[@id='child']/parent::div
//input[@id='field']/ancestor::form
```

### 7. Explain CSS Selectors with examples

```css
#id                     /* ID selector */
.classname              /* Class selector */
tagname                 /* Tag selector */
[attribute='value']     /* Attribute selector */
tag#id.class            /* Combined */
parent > child          /* Direct child */
parent child            /* Any descendant */
element:first-child     /* Pseudo-class */
element:nth-child(2)    /* Nth child */
input[type='text']      /* Attribute */
input[id*='user']       /* Contains */
input[id^='user']       /* Starts with */
input[id$='name']       /* Ends with */
```

### 8. Which is better: XPath or CSS Selector?

| Aspect | XPath | CSS Selector |
|--------|-------|--------------|
| Speed | Slower | Faster |
| Backward traversal | Yes | No |
| Text matching | Yes | No |
| Browser support | All | All (natively) |
| Readability | Less readable | More readable |

**Recommendation:** Use CSS for simple locators, XPath for complex scenarios

## Waits

### 9. What are the different types of waits in Selenium?

**1. Implicit Wait**
```java
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
```
- Applied globally to all elements
- Polls DOM until element found or timeout

**2. Explicit Wait**
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("element")));
```
- Applied to specific elements/conditions
- More flexible with ExpectedConditions

**3. Fluent Wait**
```java
FluentWait<WebDriver> wait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(500))
    .ignoring(NoSuchElementException.class);
```
- Custom polling intervals
- Can ignore specific exceptions

### 10. Why not use Thread.sleep()?
- Hard-coded wait (wastes time if element appears earlier)
- Not reliable (element may not be ready even after sleep)
- Slows down test execution
- **Always prefer dynamic waits**

## WebDriver Commands

### 11. Common WebDriver methods

```java
// Browser commands
driver.get("https://example.com");
driver.getCurrentUrl();
driver.getTitle();
driver.getPageSource();
driver.navigate().back();
driver.navigate().forward();
driver.navigate().refresh();

// Window commands
driver.manage().window().maximize();
driver.manage().window().minimize();
driver.manage().window().fullscreen();
driver.manage().window().getSize();
driver.manage().window().setSize(new Dimension(1920, 1080));

// Element commands
element.click();
element.sendKeys("text");
element.clear();
element.getText();
element.getAttribute("value");
element.isDisplayed();
element.isEnabled();
element.isSelected();
```

### 12. How to handle dropdowns?

```java
// Using Select class
Select dropdown = new Select(driver.findElement(By.id("dropdown")));

// Select options
dropdown.selectByVisibleText("Option 1");
dropdown.selectByValue("value1");
dropdown.selectByIndex(0);

// Get selected option
dropdown.getFirstSelectedOption().getText();

// Get all options
List<WebElement> options = dropdown.getOptions();
```

### 13. How to handle alerts?

```java
// Switch to alert
Alert alert = driver.switchTo().alert();

// Alert actions
alert.accept();           // Click OK
alert.dismiss();          // Click Cancel
alert.getText();          // Get alert text
alert.sendKeys("input");  // Input text (prompt)
```

### 14. How to handle multiple windows/tabs?

```java
// Get current window handle
String mainWindow = driver.getWindowHandle();

// Get all window handles
Set<String> allWindows = driver.getWindowHandles();

// Switch to new window
for (String window : allWindows) {
    if (!window.equals(mainWindow)) {
        driver.switchTo().window(window);
        break;
    }
}

// Switch back to main window
driver.switchTo().window(mainWindow);
```

### 15. How to handle frames/iframes?

```java
// Switch to frame
driver.switchTo().frame("frameName");       // By name
driver.switchTo().frame(0);                 // By index
driver.switchTo().frame(frameElement);      // By element

// Switch to parent frame
driver.switchTo().parentFrame();

// Switch to default content
driver.switchTo().defaultContent();
```

## Actions Class

### 16. How to perform mouse and keyboard actions?

```java
Actions actions = new Actions(driver);

// Mouse actions
actions.moveToElement(element).perform();           // Hover
actions.click(element).perform();
actions.doubleClick(element).perform();
actions.contextClick(element).perform();            // Right-click
actions.dragAndDrop(source, target).perform();

// Keyboard actions
actions.sendKeys(Keys.ENTER).perform();
actions.keyDown(Keys.CONTROL).sendKeys("a").keyUp(Keys.CONTROL).perform();  // Ctrl+A

// Chain actions
actions.moveToElement(element)
       .click()
       .sendKeys("text")
       .perform();
```

## JavaScript Executor

### 17. When to use JavaScriptExecutor?

```java
JavascriptExecutor js = (JavascriptExecutor) driver;

// Click element (when regular click fails)
js.executeScript("arguments[0].click();", element);

// Scroll to element
js.executeScript("arguments[0].scrollIntoView(true);", element);

// Scroll page
js.executeScript("window.scrollBy(0, 500)");
js.executeScript("window.scrollTo(0, document.body.scrollHeight)");

// Get element properties
String value = (String) js.executeScript("return arguments[0].value;", element);

// Enter text
js.executeScript("arguments[0].value='text';", element);

// Wait for page load
js.executeScript("return document.readyState").equals("complete");
```

## Framework Design

### 18. What is Page Object Model (POM)?
- Design pattern for test automation
- Separates page elements from test logic
- Each web page has corresponding Page Class
- Contains web elements and methods to interact with them

**Benefits:**
- Code reusability
- Easy maintenance
- Readability
- Reduced code duplication

### 19. What is Page Factory?
- Optimized way to create page objects
- Uses `@FindBy` annotations
- Lazy initialization of elements

```java
public class LoginPage {
    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    public LoginPage(WebDriver driver) {
        PageFactory.initElements(driver, this);
    }
}
```

### 20. What is Data-Driven Testing?
- Test data is separated from test logic
- Same test runs with different data sets
- Data can be in Excel, JSON, CSV, database

**Implementation:**
- TestNG DataProvider
- Apache POI for Excel
- JSON/CSV parsing libraries

## Advanced Topics

### 21. How to take screenshots in Selenium?

```java
TakesScreenshot ts = (TakesScreenshot) driver;
File source = ts.getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(source, new File("screenshot.png"));
```

### 22. What is Selenium Grid?
- Enables distributed test execution
- Hub-Node architecture
- Run tests on multiple machines/browsers in parallel

### 23. How to handle file upload/download?

**Upload:**
```java
// Using sendKeys
driver.findElement(By.id("upload")).sendKeys("/path/to/file");

// Using AutoIT/Robot class for native dialogs
```

**Download:**
- Configure browser preferences
- Disable download prompts
- Verify downloaded file

### 24. What are common Selenium exceptions?

| Exception | Cause |
|-----------|-------|
| NoSuchElementException | Element not found |
| ElementNotInteractableException | Element not clickable/editable |
| StaleElementReferenceException | Element no longer in DOM |
| TimeoutException | Wait timed out |
| NoAlertPresentException | No alert to switch to |
| InvalidSelectorException | Invalid locator syntax |

### 25. How to handle StaleElementReferenceException?
- Re-locate the element
- Use try-catch and retry
- Use ExpectedConditions.refreshed()
- Implement Page Object pattern correctly

---

## Quick Tips for Interview

1. **Know all locator strategies** - especially XPath and CSS
2. **Understand waits** - implicit vs explicit vs fluent
3. **Practice Actions class** - hover, drag-drop, keyboard
4. **Know exception handling** - common exceptions and solutions
5. **Understand POM** - be ready to explain/design
