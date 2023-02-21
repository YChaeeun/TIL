# Handle Activity State Changes

{% embed url="https://developer.android.com/guide/components/activities/state-changes" %}



When Activity transition happens && How to handle Activity transition

## Configuration changes occurs

* **Occurs by...**
  * changes between portrait and landscape orientations
  * changes to language
  * changes input device
* Activity is **destroyed** & **recreated**
  * old ones -- onPause(), onStop(), onDestroy() will be triggered
  * new ones -- onCreate(), onStart(), onResume() will be triggered
* To save activity's UI state, use combination of methods below!
  * ViewModel
  * onSaveInstanceState() method
  * persistent local storage

### multi-window cases

* although there are two apps that are visible to the user, **only the one** with which the user is interacting is in the foreground and has focus
* When the user switches from app A to app B,&#x20;
  * the system calls `onPause()` on app A
  * &#x20;`onResume()` on app B
  * It switches between these two methods each time the user toggles between apps

## Activity or dialog appears in foreground

* If new activity or dialog appears taking focus, the **covered activity(old one)** **lose focus** so system calls `onPause()` on it
  * when covered activity returns to foreground and regain focus, system calls `onResume()`
* If new one _completely_ covers the activity, covered activity is in state of `onPause()` and `onStop()`
  * &#x20;when same instance of covered activity comes back, system calls `onRestart()`, `onStart()`, `onResume()`
  * if a new instance of covered activity comes back, system calls only `onStart()` and `onResume()`

## User taps Back Button

* the activity transitions through the `onPause()`, `onStop()`, and `onDestroy()` callbacks && also removed from the back stack
* **`onSaveInstanceState()` callback does not fire in this case**
  * Because it is user's clear action that there is no expectation of returning to the same instance of the activity

## System kills app process

* If an app is in the background and the system needs to free up additional memory for a foreground app, **the background app can be killed** by the system to free up more memory
