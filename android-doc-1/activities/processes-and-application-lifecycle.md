# Processes and Application Lifecycle

{% embed url="https://developer.android.com/guide/components/activities/process-lifecycle" %}

An unusual and fundamental feature of Android is that an **application process's lifetime is **_**not**_** directly controlled by the application itself**

* Instead, it is determined by the system through a combination of....
  * the parts of the application that the system knows are running
  * how important these things are to the user
  * how much overall memory is available in the system

It's important to understand how different application components -- `Activity`, `Service`, and `BroadcastReceiver`  -- impact the lifetime of the application's process

* Not using these components correctly can result in the system killing the application's process while it is do

### importance hierarchy of process types

* system determines which process to be killed when low on memory

1. foreground process
2. visible process
3. service process
4. cached process

