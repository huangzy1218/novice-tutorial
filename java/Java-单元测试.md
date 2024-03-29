# 单元测试

## 编写JUnit测试

单元测试就是针对最小的功能单元编写测试代码。对Java程序进行单元测试就是针对单个Java方法的测试。

测试驱动开发，是指先编写接口，紧接着编写测试。编写完测试后，才开始真正编写实现代码。

```ascii
   编写接口
     │
     ▼
    编写测试
     │
     ▼
┌─> 编写实现
│    │
│ N  ▼
└── 运行测试
     │ Y
     ▼
    任务完成
```

### JUnit

JUnit 是一个 Java 语言的回归测试框架（regression testing framework），由 Kent Beck 和 Erich Gamma 建立。

JUnit就会给出成功的测试和失败的测试，还可以生成测试报告，不仅包含测试的成功率，而且可以统计测试的代码覆盖率。对于高质量的代码来说，测试覆盖率应该在**80%**以上。基本所有的IDE均集成了JUnit，可以在IDE中编写并运行JUnit测试。

```java
public class FactorialTest extends TestCase {
    @Test
    public void testFact() {
        assertEquals(1, Factorial.fact(1));
        assertEquals(2, Factorial.fact(2));
        assertEquals(6, Factorial.fact(3));
        assertEquals(3628800, Factorial.fact(10));
    }
}
```

`assertEquals(expected, actual)`是最常用的测试方法，它在`Assertion`类中定义。`Assertion`还定义了其他断言方法，例如：

- `assertTrue()`: 期待结果为`true`
- `assertFalse()`: 期待结果为`false`
- `assertNotNull()`: 期待结果为非`null`
- `assertArrayEquals()`: 期待结果为数组并与期望数组每个元素的值均相等

### 单元测试的优点

单元测试可以确保单个方法按照正确预期运行，如果修改了某个方法的代码，只需确保其对应的单元测试通过，即可认为改动正确。此外，测试代码本身就可以作为示例代码，用来演示如何调用该方法。

在编写单元测试的时候，我们要遵循一定的规范：

一是单元测试代码本身必须非常简单，能一下看明白，决不能再为测试代码编写测试；

二是每个单元测试应当互相独立，不依赖运行的顺序。

三是测试时不但要覆盖常用测试用例，还要特别注意测试边界条件，例如输入为`0`，`null`，空字符串`""`等情况。

## 使用`Fixture`

```java
public class Calculator {
    private long n = 0;

    public long add(long x) {
        n = n + x;
        return n;
    }

    public long sub(long x) {
        n = n - x;
        return n;
    }
}
```

```java
public class CalculatorTest extends TestCase {
    Calculator calculator;

    @BeforeEach
    public void setUp() {
        this.calculator = new Calculator();
    }

    @AfterEach
    public void tearDown() {
        this.calculator = null;
    }

    @Test
    public void testAdd() {
        assertEquals(100, this.calculator.add(100));
        assertEquals(150, this.calculator.add(50));
        assertEquals(130, this.calculator.add(-20));
    }

    @Test
    public void testSub() {
        assertEquals(-100, this.calculator.sub(100));
        assertEquals(-150, this.calculator.sub(50));
        assertEquals(-130, this.calculator.sub(-20));
    }
}
```

`@BeforeEach`和`@AfterEach`的方法，它们会在运行每个`@Test`方法前后自动运行。

测试代码在JUnit中运行顺序如下：

```java
for (Method testMethod : findTestMethods(CalculatorTest.class)) {
    var test = new CalculatorTest(); // 创建Test实例
    invokeBeforeEach(test);
        invokeTestMethod(test, testMethod);
    invokeAfterEach(test);
}
```

## 异常测试

在编写JUnit测试的时候，除了正常的输入输出，我们还要特别针对可能导致异常的情况进行测试。

```java
@Test
public void testNegative() {
    assertThrows(IllegalArgumentException.class, new Executable() {
        @Override
        public void execute() throws Throwable {
            Factorial.fact(-1);
        }
    });
}

// 简写如下
@Test
public void testNegative() {
    assertThrows(IllegalArgumentException.class, () -> {
        Factorial.fact(-1);
    });
}
```

JUnit提供`assertThrows()`来期望捕获一个指定的异常。第二个参数`Executable`封装了我们要执行的会产生异常的代码。当我们执行`Factorial.fact(-1)`时，必定抛出`IllegalArgumentException`。

`assertThrows()`在捕获到指定异常时表示通过测试，未捕获到异常，或者捕获到的异常类型不对，均表示测试失败。

## 条件测试

在运行测试的时候，有些时候，我们需要排出某些`@Test`方法，标记`@Disable`。

```java
@Disabled
@Test
void testBug101() {
    // 这个测试不会运行-
}
```

标记`@Disabled`，JUnit仍然识别出这是个测试方法，只是暂时不运行。它会在测试结果中显示：

```
Tests run: 68, Failures: 2, Errors: 0, Skipped: 5
```

类似`@Disabled`这种注解就称为条件测试，JUnit根据不同的条件注解，决定是否运行当前的`@Test`方法。

```java
public class Config {
    public String getConfigFile(String fileName) {
        String os = System.getProperty("os.name").toLowerCase();
        if (os.contains("win")) {
            return "C:\\" + fileName;
        }
        if (os.contains("mac") || os.contains("linux") ||
            os.contains("unix")) {
            return "/usr/local" + fileName;
        }
        throw new UnsupportedOperationException();
    }
}
```

```java
public class ConfigTest extends TestCase {
    Config config = new Config();

    @Test
    @EnabledOnOs(OS.WINDOWS)
    public void testWindow() {
        assertEquals("C:\\test.ini", config.getConfigFile("test.ini"));
    }

    @Test
    @EnabledOnOs({OS.LINUX, OS.MAC})
    public void testLinuxAndMac() {
        assertEquals("usr/local/test.config", config.getConfigFile("test.cfg"));
    }
}
```

不在Windows平台执行的测试，可以加上`@DisabledOnOs(OS.WINDOWS)`。

只能在Java 9或更高版本执行的测试，可以加上`@DisabledOnJre(JRE.JAVA_8)`：

```java
@Test
@DisabledOnJre(JRE.JAVA_8)
void testOnJava9OrAbove() {
    // TODO: this test is disabled on java 8
}
```

只能在64位操作系统上执行的测试，可以用`@EnabledIfSystemProperty`判断：

```java
@Test
@EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
void testOnlyOn64bitSystem() {
    // TODO: this test is only run on 64 bit system
}
```

需要传入环境变量`DEBUG=true`才能执行的测试，可以用`@EnabledIfEnvironmentVariable`：

```java
@Test
@EnabledIfEnvironmentVariable(named = "DEBUG", matches = "true")
void testOnlyOnDebugMode() {
    // TODO: this test is only run on DEBUG=true
}
```

## 参数化测试

如果待测试的输入和输出是一组数据： 可以把测试数据组织起来 用不同的测试数据调用相同的测试方法。

JUnit提供了一个`@ParameterizedTest`注解，用来进行参数化测试。

```java
@ParameterizedTest
@ValueSource(ints = {0, 1, 5, 100})
public void testAbs(int x) {
    assertEquals(x, Math.abs(x));
}

@ParameterizedTest
@ValueSource(ints = {-1, -5, -100})
public void testAbsNegative(int x) {
    assertEquals(-x, Math.abs(x));
}
```

```java
public static String capitalize(String s) {
    if (s.length() == 0) {
        return s;
    }
    return Character.toUpperCase(s.charAt(0)) + s.substring(1).toLowerCase();
}
```

可通过`@MethodSource`传入参数，它允许我们编写一个同名的静态方法来提供测试参数：

```java
@ParameterizedTest
@MethodSource
void testCapitalize(String input, String result) {
    assertEquals(result, StringUtils.capitalize(input));
}

static List<Arguments> testCapitize() {
    return List.of(
        Arguments.of("abc", "Abc"),
        Arguments.of("APPLE", "Apple"),
        Arguments.of("gooD", "Good"));
}
```

另一种传入测试参数的方法是使用`@CsvSource`，它的每一个字符串表示一行，一行包含的若干参数用`,`分隔：

```java
@ParameterizedTest
@CsvSource({"abc, Abc", "APPLE, Apple", "gooD, Good"})
void testCapitalize(String input, String result) {
    assertEquals(result, StringUtils.capitalize(input));
}
```

我们可以把测试数据提到一个独立的CSV文件中，然后标注`@CsvFileSource`：

```java
@ParameterizedTest
@CsvFileSource(resources = {"src/test-capitialize.csv"})
void testCapitalize(String input, String result) {
    assertEquals(result, StringUtils.capitalize(input));
}
```

