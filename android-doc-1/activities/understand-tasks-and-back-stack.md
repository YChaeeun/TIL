# Understand Tasks and Back Stack

{% embed url="https://developer.android.com/guide/components/activities/tasks-and-back-stack" %}



## Task and Backstack

### task

* collection of activities that user interact with
* in multi-window environment, each window manages tasks separately&#x20;

### backStack

* activities are arranged in a stack, in order in which each activity is opened
* when user presses the Back button, latest activity is finished and popped off the stack
* When all activities are removed from the stack, the task no longer exists
* activities in the back stack are **never rearranged**

![When user presses Back button, the current activity popped off and destroyed & previous activity resumes](https://developer.android.com/images/fundamentals/diagram\_backstack.png)

![Task B is in foreground, Task A is in Background lost focus](https://developer.android.com/images/fundamentals/diagram\_multitasking.png)

* when task lost focus, it goes to background
* start task A -> go to Home -> select task B ...
* If Task A comes to the foreground,&#x20;
  * all activities in its stack are intact
  * the activity at the top of the stack resumes

#### Default behavior for activities..

* If the user presses the **Back** button, the current activity is popped from the stack and destroyed
  * The previous activity in the stack is resumed
  * When an activity is destroyed, the system _**does not**_ retain the activity's state.
* When Activity A starts Activity B, Activity A is stopped, but the system retains Activity A's state
  * &#x20;If the user presses the **Back** button while in Activity B, Activity A resumes with its state restored
* When the user leaves a task by pressing the _**Home**_** button**, the current activity is stopped and its task goes into the background
  * The system retains the state of every activity in the task
  * If the user later resumes the task by selecting the launcher icon that began the task, **the task comes to the foreground and resumes the activity at the top of the stack**
* Activities can be instantiated multiple times, even from other tasks
  * back stack activities are NEVER rearranged

## Managing Tasks

* want activity to **begin a new task** when it is started (instead of being placed within the current task)
* when you start an activity, you want to **bring forward an existing instance** of it (instead of creating a new instance on top of the back stack)
* you want your **back stack to be cleared** **of all activities** except for the root activity when the user leaves the task

### Defining launch modes

* two ways to define launch modes
  * Using manifest file
    * `<activity>` elements' `launchMode` attribute
  * Using Intent flags
    * when calling `startActivity()`, you can include a flag in the `Intent`&#x20;
    * declares how (or whether) the new activity should associate with the current task
  * **Intent flag** is honored over manifest file

#### Using Manifest file

* `standard (default)`
  * The system creates a new instance of the activity in the task
* `singleTop`
  * if an instance of the **activity already exists at the top of stack**, the system call  its`onNewIntent()` rather than creating new instance
  * user can not go back to the state of activity just before `onNewIntent()` by pressing Back buttons - it will finish the activity&#x20;
  * ex) stack A-B-C; root A, top C
    * if C has `standard` launch mode, the stack; A-B-C-C
    * if C has `singleTop` launch mode, , C will receive intents through `onNewIntent()`  & the stack; A-B-C
    * if even though B has `singleTop` launch mode, since B is not at the top of stack, B will be just added to the stack; A-B-C-B
*   `singleTask`

    * the system creates a new task & instantiates the activity **at the root of the new task**
    * However, **only one instance** of the activity can exist at a time
      * if an instance of activity is already exists in a separate Task, the system r**outes the intent to the existing instance** through a call to its `onNewIntent()` method
    * If the activity is already a part of a background task with its own back stack, then the entire back stack also comes forward, on top of the current task
      * when task holding Activity Y is in the Background
      * If foreground task Activity 2 starts activity Y, the whole task holding Activity Y will come forward, on the top of current task



    <img src="https://developer.android.com/images/fundamentals/diagram_backstack_singletask_multiactivity.png" alt="" data-size="original">


* `singleInstance`
  * same as `singleTask` except that the system doesn't launch any other activities into the task holding the instance

#### Using Intent flag

* `FLAG_ACTIVITY_NEW_TASK`
  * same as `singleTask` launch mode
  * Start the activity in a new task
  * If a task is already running for the activity you are now starting, that task is brought to the foreground with its last state restored&#x20;
  * the activity receives the new intent in `onNewIntent()`
* `FLAG_ACTIVITY_SINGLE_TOP`
  * same as `singleTop`
  * If the activity being started is the current activity (at the top of the back stack), then the existing instance receives a call to `onNewIntent()`, instead of creating a new instance of the activity.
* `FLAG_ACTIVITY_CLEAR_TOP`
  * &#x20;If the activity being started is already running in the current task, then instead of launching a new instance of that activity, **all of the other activities on top of it are destroyed**&#x20;
  * this intent is delivered to the resumed instance of the activity (now on top), through `onNewIntent()`
  * ex) A-B-C-D, root A top D
    * call B with this flag, clear top of B (C-D)
    * stack A-B (onNewIntent)
* `FLAG_ACTIVITY_CLEAR_TOP` is most often used in conjunction with `FLAG_ACTIVITY_NEW_TASK`
  * When used together, these flags are a way of locating an existing activity in another task and putting it in a position where it can respond to the intent

### Handling affinities









