# Understand the Activity Lifecycle

{% embed url="https://developer.android.com/guide/components/activities/activity-lifecycle" %}

## Lifecycle Callback

Each callback allows you to perform specific work that's appropriate to a given change of state

Doing the right work at the right time and handling transitions properly make your app more robust and performant

**Good implementation of the lifecycle callback helps to avoid....**

* Crashing
* Consuming valuable system resources meaninglessly 
* Losing user's progress -- can't know what to do when they leave and return at a later time
* Crashing & Losing when screen rotates

Putting code directly into the activity lifecycle callbacks \(ex. onStart\(\)\) is  **NOT recommended!**

Instead add logic into a independent, lifecycle-aware component

* can reuse the component without duplicates

## onCreate\(\) - Created state

**MUST Implement**

when activity enters Created state, the system invokes onCreate\(\) call back

* declare user interface layout
  * setContentView\(\)
  * Instead using xml file, you create new View - ViewGroup, then pass the root ViewGroup to setContentView
* perform basic application startup logic 
  * should happen **only once** for the entire life of the activity
  * ex\) binding data to list / associate with ViewModel / instantiate class-scope variable
* receives `savedInstanceState`
  * `Bundle`containing activity's previously saved state
  * **override fun onRestoreInstanceState\(\)**
    * call only when data is previously saved by **override fun onSaveInstanceState\(\)**
    * onSaveInstanceState\(\) invoked when activity may be temporarily destroyed
    * Bundle is same as the one used in onCreate\(\)
* ON\_CREATE event for @OnLifecycleEvent annotation



## onStart\(\) - Started state

when activity enters Started state, the system invokes onStart\(\) call back

* visible to user
* prepares for the activity to enter the foreground and become interactive
  * app initializes the code that maintains the UI
* completes very quickly
* If you initialize something after the ON\_START event, release or terminate it after the ON\_STOP event

## onResume\(\) - Resumed state

when activity enters Resumed state, activity comes to the foreground, the system invokes onResume\(\) call back

* Interacts with user
* App stays in this state, until something happens to take focus away
  * ex\) receiving phone call / user navigates to other app / device screen turning off
* Initialize components that you release during onPause\(\) 
* If you initialize after the ON\_RESUME event, release after the ON\_PAUSE event

## onPause\(\) 

