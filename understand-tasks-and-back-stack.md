# Understand Tasks and Back Stack

{% embed url="https://developer.android.com/guide/components/activities/tasks-and-back-stack" %}



## Task and Backstack

### task

* collection of activities that user interact with
* in multi-window environment, each window manages tasks separately 

### backStack

* activities are arranged in a stack, in order in which each activity is opened
* when user presses the Back button, latest activity is finished and popped off the stack
* When all activities are removed from the stack, the task no longer exists
* activities in the back stack are **never rearranged**

![When user presses Back button, the current activity popped off and destroyed &amp; previous activity resumes](https://developer.android.com/images/fundamentals/diagram_backstack.png)

![Task B is in foreground, Task A is in Background lost focus](https://developer.android.com/images/fundamentals/diagram_multitasking.png)

* when task lost focus, it goes to background
* start task A -&gt; go to Home -&gt; select task B ...
* If Task A comes to the foreground, 
  * all three activities in its stack are intact
  * the activity at the top of the stack resumes

#### Default behavior for activities..

* If the user presses the **Back** button, the current activity is popped from the stack and destroyed
  * The previous activity in the stack is resumed
  * When an activity is destroyed, the system _**does not**_ retain the activity's state.
* When Activity A starts Activity B, Activity A is stopped, but the system retains its state
  *  If the user presses the **Back** button while in Activity B, Activity A resumes with its state restored
* When the user leaves a task by pressing the _**Home**_ **button**, the current activity is stopped and its task goes into the background
  * The system retains the state of every activity in the task
  * If the user later resumes the task by selecting the launcher icon that began the task, **the task comes to the foreground and resumes the activity at the top of the stack**
* Activities can be instantiated multiple times, even from other tasks
  * back stack activities are NEVER rearranged

## Managing Tasks

* want activity to **begin a new task** when it is started \(instead of being placed within the current task\)
* when you start an activity, you want to **bring forward an existing instance** of it \(instead of creating a new instance on top of the back stack\)
* you want your **back stack to be cleared** of all activities except for the root activity when the user leaves the task

### Defining launch modes

* two ways to define launch modes
  * Using manifest file
    * `<activity>` elements' `launchMode`
  * Using Intent flags
    * when calling `startActivity()`, you can include a flag in the `Intent` 
    * declares how \(or whether\) the new activity should associate with the current task
  * **Intent flag** is honored over manifest file

#### Using Manifest file

* standard
* singleTop
* singleTask
* singleInstance

#### Using Intent flag

* `FLAG_ACTIVITY_NEW_TASK`
* `FLAG_ACTIVITY_SINGLE_TOP`
* `FLAG_ACTIVITY_CLEAR_TOP`











