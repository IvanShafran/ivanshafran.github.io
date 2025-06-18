Improve Android unit tests with shared preferences mock library
Shared preferences mock is the lightweight library let you increase coverage of unit tests and simplify code for them with one line of code

**TL;DR**: [Shared preferences mock](https://github.com/IvanShafran/shared-preferences-mock) is the lightweight library let you increase coverage of unit tests and simplify code for them with one line of code.

## The problem

Unit test on Android uses a framework mock where every method throws `UnsupportedOperationException `or does nothing depending on your settings in Gradle `testOptions`.

In unit tests, we mostly test business logic, which often depends on preferences. Therefore you should mock all classes which use `SharedPreferences `inside. It is a lot of boilerplate in tests. Moreover, because of this limitation, we never test preferences class code. However, it also may have bugs.

## The solution

The library implements `SharedPreferences`, `Context.getSharedPreferences` and `Context.deleteSharedPreferences` in memory using only Java API.

It allows you to use classes with shared preferences in unit tests as is and also to write tests for them.

## Usage

Add it in your root build.gradle at the end of repositories:

```groovy
allprojects {
    repositories {
        ...
        mavenCentral()
    }
}
```

Add the dependency to module build.gradle:

```groovy
dependencies {
    testImplementation 'io.github.ivanshafran:shared-preferences-mock:1.2.4'
}
```

In most cases, preference class or preference utils depends on `Context`. In unit test instead of real `Context` pass `Context` created by `new SPMockBuilder().createContext()`.

If you already have `Context` with some mocked methods, you can use `new SPMockBuilder().wrapContext(preconfiguredContext)`.

If you need raw SharedPreferences use new SPMockBuilder().createSharedPreferences().

If thread safety is necessary for your test, then configure `SPMockBuilder.setThreadSafe(true)`. It is false by default.

## Sample

The sample below describes common code where logic uses preference class to check condition. In line 29 `SPMockBuilder` creates a context for the preference unit tests. Simple, isn’t it?

```java
public class CounterPreferences {
    // ...
    private final SharedPreferences sharedPreferences;
    public CounterPreferences(@NonNull final Context context) {
        sharedPreferences = context.getSharedPreferences(FILENAME, Context.MODE_PRIVATE);
    }

    public int getCounter() {
        return sharedPreferences.getInt(KEY, 0);
    }
    // ...
}

public class ShowMessageLogic {
    // ...
    public boolean shouldShowMessage() {
        return counterPreferences.getCounter() >= 42;
    }
}

public class ShowMessageLogicTest {

    private final SPMockBuilder spMockBuilder = new SPMockBuilder();
    private CounterPreferences counterPreferences;
    private ShowMessageLogic showMessageLogic;

    @Before
    public void setUp() {
        counterPreferences = new CounterPreferences(spMockBuilder.createContext());
        showMessageLogic = new ShowMessageLogic(counterPreferences);
    }

    @Test
    public void on42CounterItShouldShowMessage() {
        counterPreferences.setCounter(42);
        Assert.assertTrue(showMessageLogic.shouldShowMessage());
    }

    @Test
    public void onLessThan42ItShouldNotShowMessage() {
        counterPreferences.setCounter(41);
        Assert.assertFalse(showMessageLogic.shouldShowMessage());
    }
}
```

## Alternatives

Create `CounterPreferences` interface and implement Java version for a test by yourself:

- Doesn’t test CounterPreferences implementation
- Requires boilerplate code in tests

Use Mockito:

- Doesn’t test CounterPreferences implementation

Use Robolectric:

- Has test startup delay for a few seconds

Use instrumented test(not a unit test!):

- Requires device for testing

## The library and sample code

You can star the library and learn more info at:
https://github.com/IvanShafran/shared-preferences-mock