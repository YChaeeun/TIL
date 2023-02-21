# Test your app's activities

{% embed url="https://developer.android.com/guide/components/activities/testing" %}

## ActivityScenario

* placing your app's activities in particular states
* you can place your activity in states that simulate the device-level events
* provides **thread safety**, **synchronizing events** between your test's instrumentation thread and the thread that runs your activity under test
* all methods within `ActivityScenario` is **blocking calls**
  * the API requires you to run them in the _instrumentation thread_

### **create a activity**

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
    }
}
```

* now activity is in `RESUMED` state
* free to interact with activity's `View` elements using Espresso UI tests

### **create a activity with `ActivityScenarioRule`**

* automatically call `ActivityScenario.launch` before each test&#x20;
* automatically call `ActivityScenario.close` at test teardown

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @get:Rule var activityScenarioRule = activityScenarioRule<MyActivity>()
    
    @Test fun testEvent() {
        val scenario = activityScenarioRule.scenario
    }
}
```

### Drive the activity to a new state

* **moveToState()**
  * it's interrupted by another app or a system action
  * drive the activity to a different state -- ex) CREATED, STARTED

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
        scenario.moveToState(State.CREATED)
    }
}
```

### Determine the current activity state

* get value of `state` field within `ActivityScenario` object
* particularly helpful to check the state of an activity under test if the activity _redirects to another activity_ or _finishes itself_

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
        scenario.onActivity { activity ->
            startActivity(Intent(activity, MyOtherActivity::class.java))
        }
        
        val originalActivityState = scenario.state
    }
}
```

### Recreate the activity

* when system destroy an activity due to low resources of device, app requires to recreate activity
* `ActivityScenario` class maintains...
  * activity's **saved instance state**&#x20;
  * any objects annotated using `@NonConfigurationInstance`

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
        senario.recreate()
    }
}
```

### Retrieve activity results

* get value of `result` field in `AndroidScenario` object
  * to retrieve **result code** or **data** associated with finished activity
* This method might throws **runtime exception** if method times out
  * This is blocking call
  * `ActivityScenario` doesn't call `finish()` on the activity

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
        onView(withId(R.id.finish_button)).perform(click())
        
        // Activity under test is now finished
        
        val resultCode = scenario.result.resultCode
        val resultData = scenario.result.resultData
    }
}
```

### Trigger actions in the activity

* to trigger actions under test, use **Espresso** view matchers to interact with elements in view
* If you need to call a method on the activity itself, you can do so safely by implementing `ActivityAction`

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
        
        // Espresso
        onView(withId(R.id.refresh)).perform(click())
        
        // ActivityAction
        scenario.onActivity { activity ->
          activity.handleSwipeToRefresh()
        }
    }
}
```

> In your test class, don't keep references to the objects that you pass into `onActivity()`. These references consume system resources, and the references themselves might be stale because the framework can recreate an activity that's passed into the callback method.

