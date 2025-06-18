In the article we take a look at the unit test evolution from beginner to pro-level. “Rate us” dialog is a popular feature, and it will be a good example. Typical “rate us” dialog has requirements like:

- Should be shown after some condition or user action
- Has “rate us” button that leads to Google Play
- Has “remind me later” button that schedules dialog to show after some time(2 months in our case)
- Has “never show again” button that hides the dialog forever

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ejyxu1oeoslyezenakok.png)

# Zero version: no unit tests

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l6dtiw4z982odyqn3jls.png)

At the beginning of the Android community, unit tests were not so popular. There are several arguments behind this:

- Tests require time to be implemented and maintained
- Apps were mostly simple
- The framework itself isn’t friendly for writing unit tests

# First version: my first unit test

But unfortunately, “rate us” dialog is a too vital feature to ignore unit tests. The more users leave reviews, the more new users your app gets. Also, it should not be annoying. Otherwise, people will leave 1-star reviews.

I’ve implemented a sample app that follows the logic described in the introduction. To trigger dialog, a user should click button two times. [Check code in the repository.](https://github.com/IvanShafran/android-unit-tests-evolution-rate-us-dialog)

To code first unit test, I had to do several things.

# Learn a little about JUnit Framework

> Feel free to skip this chapter if you are familiar with JUnit.

JUnit is the most popular testing framework for Java. It’s included in dependencies by default to all new Android projects:

```groovy
dependencies {
    testImplementation 'junit:junit:4.12'
}
```

To write a test, you should create a class in test folder which is created by default and contains ExampleUnitTest.java. Usually, if devs test SomeClass , devs will name class with tests SomeClassTest . Moreover, most times, it belongs to the same Java package.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cjfj2ao4l40oe5p7kujf.png)

Let’s see a simple test. You should annotate all test methods with org.junit.Test . Android Studio will automatically show the run test button. assertEquals will throw an exception if arguments are not equal. JUnit marks test as passed if it ends without any exception.

# Abstract an Android from business logic

In Android, you can’t write unit tests for a class that uses the Android framework. But wait… WHAT???!

Yes, the framework requires a specific environment and you can’t run it on any JVM. In unit tests, all Android classes are mocked to throw an exception. The best that we can do is to force it to return default values instead of throwing an exception.

```groovy
// In application module build.gradle
android.testOptions {
    unitTests.returnDefaultValues = true
}
```

Back to the dialog, we certainly use Android classes like Activity/Fragment, View, Dialog, and others for “rate us” feature. Therefore we can’t write unit tests without a bit of effort.

First, I created an interface for every Android dependency which I use for “rate us” showing logic.

```java
interface BuyPreferences {
    fun incrementBuyCount()
    fun getBuyCount(): Int
}

class BuyPreferencesImpl(context: Context) : BuyPreferences {
    // ...
    private val sharedPreferences: SharedPreferences = context.getSharedPreferences(...)

    override fun incrementBuyCount() {
        val count = getBuyCount()
        sharedPreferences.edit().putInt(BUY_COUNT_KEY, count + 1).apply()
    }

    override fun getBuyCount() = sharedPreferences.getInt(BUY_COUNT_KEY, 0)
}
```

Second, I created an interface for every Java dependency that can’t be used directly in tests. More specifically, it is System.currentTimeMillis() .

```kotlin
interface Time {
    fun getCurrentTimeMillis(): Long
}

class TimeImpl : Time {
    override fun getCurrentTimeMillis() = System.currentTimeMillis()
}
```

Last, I applied the dependency inversion principle toShowRateUsLogic.

```kotlin
class ShowRateUsLogic(
    private val rateUsPreferences: RateUsPreferences,
    private val buyPreferences: BuyPreferences,
    private val time: Time
) {
    fun shouldShowRateUs(): Boolean {
        val timeFromLastShown = time.getCurrentTimeMillis() - rateUsPreferences.getLastShownTimeMillis()
        return when {
            // User doesn't want to see "rate us" again
            rateUsPreferences.isNeverShownAgainClicked() -> false
            // User already rated the app
            rateUsPreferences.isRateNowClicked() -> false
            // "Rate us" should be shown after 2 "buy" clicked
            buyPreferences.getBuyCount() < 2 -> false
            // Show "rate us" only first time or if passed two months since last shown time
            timeFromLastShown < TimeUnit.DAYS.toMillis(60) -> false
            else -> true
        }
    }
}
```

# Mocks

Now we should write mock classes for unit tests. I’ll show one mock below. You can check all the mocks [here](https://github.com/IvanShafran/android-unit-tests-evolution-rate-us-dialog/tree/master/app/src/test/java/com/github/ivanshafran/unit_tests_evolution/v1).

```java
public class BuyPreferencesMock implements BuyPreferences {
    private int count;

    @Override public void incrementBuyCount() {
        ++count;
    }

    @Override public int getBuyCount() {
        return count;
    }
}
```

# First unit test

After hardworking, it is a pleasure to code the first unit test :)

```java
public class ShowRateUsLogicTest {
    private RateUsPreferencesMock rateUsPreferences;
    private BuyPreferencesMock buyPreferences;
    private TimeMock time;
    private ShowRateUsLogic showRateUsLogic;

    @Test public void test1() {
        rateUsPreferences = new RateUsPreferencesMock();
        buyPreferences = new BuyPreferencesMock();
        time = new TimeMock();
        showRateUsLogic = new ShowRateUsLogic(rateUsPreferences, buyPreferences, time);

        buyPreferences.incrementBuyCount();
        time.setCurrentTimeMillis(new Date(2019, 6, 7).getTime());

        Assert.assertFalse(showRateUsLogic.shouldShowRateUs());
    }
}
```


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/oobec0ox610c7ssoi5sb.png)

To be honest, it is not. But I wrote it in the “my first unit test” style. And I’ll fix it in the next chapters.

# Second version: code cleaning

The first unit test is cool but if no one can understand it, then it is useless. Therefore I’ve refactored test class:

1. setUp is marked with @Before annotation. It makes the method to be invoked before every unit test. We’ll move the common test code to setUp method.
2. A good test method has a meaningful name. It’s better for reading and also it appears in test reports. We’ll rename test1 to onFirstCheckAndOneClickItShouldNotShow .
3. I’ve added a more complicated test. The test name is too short to express all the information. That’s why we’ll add comments to the method body.
4. For the last step, we’ll delete unnecessary and wrong time set.

```java
public class ShowRateUsLogicTest {
    // property declaration is skipped
    @Before public void setUp() {
        rateUsPreferences = new RateUsPreferencesMock();
        buyPreferences = new BuyPreferencesMock();
        time = new TimeMock();
        showRateUsLogic = new ShowRateUsLogic(rateUsPreferences, buyPreferences, time);
    }

    @Test public void onFirstCheckAndOneClickItShouldNotShow() {
        buyPreferences.incrementBuyCount();

        Assert.assertFalse(showRateUsLogic.shouldShowRateUs());
    }

    @Test public void onThreeClicksAndItShouldShow() {
        // clicked three times
        buyPreferences.incrementBuyCount();
        buyPreferences.incrementBuyCount();
        buyPreferences.incrementBuyCount();
        // set first dialog show time
        final Calendar calendar = Calendar.getInstance();
        calendar.set(2019, Calendar.JULY, 7);
        rateUsPreferences.setLastShownTimeMillis(calendar.getTimeInMillis());
        // set current time to be 90 days after first show
        time.setCurrentTimeMillis(calendar.getTimeInMillis() + TimeUnit.DAYS.toMillis(90));

        Assert.assertTrue(showRateUsLogic.shouldShowRateUs());
    }
}
```

# Third version: mocking libraries

As mentioned, we can not use Android classes in unit tests. But almost all our classes do it. And it’s quite boring to create an interface, an implementation and a mock for every class.

Fortunately, there is an alternative way. We can use mocking libraries.

## Mockito

Directly to the most frequently used API:

- Use Mockito.mock(Class) to mock any interface or class
- Use Mockito.when(instance.method()).thenReturn(value) to mock method call
- Use Mockito.verify(instance).method() to check if method was called

## [Shared preferences mock](https://github.com/IvanShafran/shared-preferences-mock)

In my practice, shared preferences class is the most used Android dependency in business logic. Therefore I’ve implemented shared preferences mock library which mimics Android implementation. Now you can use shared preferences in unit tests with one additional line of code ;)

```java
public class ShowRateUsLogicTest {
    // property declaration is skipped
    @Before public void setUp() {
        final Context mockedContext = new SPMockBuilder().createContext();
        rateUsPreferences = new RateUsPreferencesImpl(mockedContext);
        buyPreferences = new BuyPreferencesImpl(mockedContext);
        time = Mockito.mock(Time.class);
        showRateUsLogic = new ShowRateUsLogic(rateUsPreferences, buyPreferences, time);
    }

    // first test code leaves the same
    
    // second test code changed only in time mocking
    @Test public void onThreeClicksAndItShouldShow() {
        // ...

        // set current time to be 90 days after first show
        Mockito.when(time.getCurrentTimeMillis()).thenReturn(calendar.getTimeInMillis() + TimeUnit.DAYS.toMillis(90));

        Assert.assertTrue(showRateUsLogic.shouldShowRateUs());
    }
}
```

# Fourth version: Kotlin

Kotlin is a good language for Android development. And also, it can bring improvements to unit tests code.

1. We’ll rename the test method name using spaces enclosed in backticks
2. We’ll move the common preparation code to function with default arguments
3. We’ll delete unnecessary comments because we can use named arguments
4. We’ll use mockito-kotlin cause it has a more idiomatic and compact syntax

```kotlin
class ShowRateUsLogicTest {
    // property declaration and setup are skipped
    private fun prepareConditions(
        buyClickedTimes: Int = 0, 
        isNeverShownAgainClicked: Boolean = false,
        isRateNowClicked: Boolean = false, 
        lastShownTimeMillis: Long = 0, 
        currentTimeMillis: Long = 0
    ) {
        repeat(buyClickedTimes) { buyPreferences.incrementBuyCount() }
        if (isNeverShownAgainClicked) rateUsPreferences.setNeverShownAgainClicked()
        if (isRateNowClicked) rateUsPreferences.setRateNowClickedClicked()
        rateUsPreferences.setLastShownTimeMillis(lastShownTimeMillis)
        whenever(time.getCurrentTimeMillis()).thenReturn(currentTimeMillis)
    }

    @Test fun onFirstCheckAndOneClickItShouldNotShow() {
        prepareConditions(buyClickedTimes = 1)

        Assert.assertFalse(showRateUsLogic.shouldShowRateUs())
    }

    @Test fun onThreeClicksAndItShouldShow() {
        prepareConditions(
            buyClickedTimes = 3,
            lastShownTimeMillis = SOME_DAY_IN_MILLIS,
            currentTimeMillis = SOME_DAY_IN_MILLIS + MORE_THAN_TWO_MONTHS
        )

        Assert.assertTrue(showRateUsLogic.shouldShowRateUs())
    }
}
```

# Fifth version: Spek

[Spek ](https://github.com/spekframework/spek)is a unit testing framework for Kotlin which supports Specification and Gherkin style.

Personally, the crucial features of Spek are:

Ability to structure test due to condition
Ability to construct tests on the go(cause test code is a lambda, not a method)
I intentionally will not describe syntax because it is easy to understand. And if you are interested in Spek then check out this [link](https://spekframework.org/core-concepts/).

```kotlin
class ShowRateUsLogicTest : Spek({
    // property declaration, setup and preparation are skipped
    describe("show rate us logic") {
        context("first conditions checks") {
            context("buy clicked once") {
                beforeEachTest {
                    prepareConditions(buyClickedTimes = 1)
                }

                it("should not show 'rate us'") {
                    Assert.assertFalse(showRateUsLogic.shouldShowRateUs())
                }
            }

            context("buy clicked two times") {
                beforeEachTest {
                    prepareConditions(buyClickedTimes = 2)
                }

                it("should show 'rate us'") {
                    Assert.assertTrue(showRateUsLogic.shouldShowRateUs())
                }
            }
        }

        context("'rate us' was shown already, and user clicked 'show me later' on the dialog") {
            context("less than two months passed and user clicks buy") {
                beforeEachTest {
                    prepareConditions(
                        buyClickedTimes = 3,
                        lastShownTimeMillis = SOME_DAY_IN_MILLIS,
                        currentTimeMillis = SOME_DAY_IN_MILLIS + LESS_THAN_TWO_MONTHS
                    )
                }

                it("should not show 'rate us' again") {
                    Assert.assertFalse(showRateUsLogic.shouldShowRateUs())
                }
            }

            context("more than two months passed and user clicks buy") {
                beforeEachTest {
                    prepareConditions(
                        buyClickedTimes = 3,
                        lastShownTimeMillis = SOME_DAY_IN_MILLIS,
                        currentTimeMillis = SOME_DAY_IN_MILLIS + MORE_THAN_TWO_MONTHS
                    )
                }

                it("should show 'rate us' again") {
                    Assert.assertTrue(showRateUsLogic.shouldShowRateUs())
                }
            }
        }
    }
})
```

Moreover, Spek generates a structured test report in Android Studio.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hqn3r8pjpftxjj1vb9xy.png)

# Bonus part

The article about unit tests in Android will not be full without several mentions. If you have more links to mention, please share them in comments and I’ll add them to the article.

## Robolectric

> Robolectric is a framework that brings fast and reliable unit tests to Android. Tests run inside the JVM on your workstation in seconds.

It is not a pure unit testing but allows us to test Android APIs without launching a device or emulator. On the other hand, it has a bigger test run time.

## Assertion frameworks

‘Rate us’ dialog logic has boolean return value, therefore we used simple assertTrue or assertFalse. But for more complicated tests, there’s not enough flexibility in default assertions.

> Hamcrest is a framework for writing matcher objects allowing ‘match’ rules to be defined declaratively.

```
assertThat(Math.sqrt(-1), is(notANumber()))
```

> AssertJ — fluent assertions java library.

```
assertThat(frodo.getName()).isEqualTo("Frodo")
```

> Truth makes your test assertions and failure messages more readable. Similar to AssertJ, it natively supports many JDK and Guava types, and it is extensible to others.

```
assertThat(notificationText).contains("testuser@google.com")
```