# Introduction to Activities

{% embed url="https://developer.android.com/guide/components/activities/intro-activities" %}



## Activity

* Android system initiates code in an [`Activity`](https://developer.android.com/reference/android/app/Activity) instance by invoking specific callback methods that correspond to specific stages of its lifecycle.
* The [`Activity`](https://developer.android.com/reference/android/app/Activity) class is designed to facilitate user's non-deterministically journey -- entry points can be vary in one app
  * eg\) If app A invokes to start app B, the calling app A **invokes an activity** in app B, not the whole app B as an atomic whole.
* Activity provides window -- draws UI
* Each activity is loosely bounded to the other activity
* Activities must be registered information in app's manifest



## Configuring the manifest

* android:name
  * class name of activity
* Intent Filter
  * help launching activity based on **explicit** or **implicit** request
  * explicit -- knows specific activity to launch
  * implicit -- just knows what to do in any activity \(then system UI asks user to choose app that can perform the job\)
  * &lt;action&gt; &lt;category&gt; &lt;data&gt;
* Permission
  * android:permission
  * A parent activity **cannot launch** a child activity unless both activities have the same permissions in their manifest.
    * if parent activity has &lt;uses-permission&gt;, child must have a matching &lt;uses-permission&gt; element
  * [https://developer.android.com/training/articles/security-tips](https://developer.android.com/training/articles/security-tips)



## Managing the activity lifecycle

The Activity has a number of states, and series of **callback** help user to **handle transitions between states**.

![Simplest Android Activity Lifecycle - Stack Overflow](https://i.stack.imgur.com/1byIg.png)

### onCreate\(\)

* fires when system creates your activity
* need to implement : initialize the essential components
  * create views
  * bind date to lists
  * setContentView\(\) -- define the layout for activity's user interface

### onStart\(\)

* As [`onCreate()`](https://developer.android.com/reference/android/app/Activity#onCreate%28android.os.Bundle%29) exits, the activity enters Started state
* activity's **final preparation** for coming to the foreground and becoming interactive

### onResume\(\)

* just before the activity starts interaction with the user
* At this point, the activity is **at the TOP of the activity stack**
* captures all user input
* **Most of an app’s core functionality is implemented in the** [**`onResume()`**](https://developer.android.com/reference/android/app/Activity#onResume%28%29) **method**

### **onPause\(\)**

* invoked when activity loses focus
  * ex\) when user tab Back or Recents button
* This means your activity is still partially visible, but soon, user will leave the activity / activity will soon enter Stopped or Resumed state
* may do
  * continue updating the UI
* should **NOT**
  * save application or user data
  * make network call
  * execute database transaction
* NEXT CALLBACK
  * onStop
  * onResume 

### onStop\(\)

* activity is no longer visible to the user
  * the activity is being destroyed
  * a new activity is starting
  * an existing activity is entering Resumed state && is covering the stopped activity
* NEXT CALLBACK
  * onRestart
  * onDestroy

### onRestart\(\)

* The system invokes this when stopped activity is about to restart
* **Restores the state** of the activity from the time that it was stopped
* NEXT CALLBACK
  * always onStart\(\)

### onDestroy\(\)

* activity is destroyed
* implemented to ensure that **all of an activity’s resources are released** when the activity, or the process containing it, is **destroyed**

