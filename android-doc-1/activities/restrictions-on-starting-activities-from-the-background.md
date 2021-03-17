# Restrictions on starting activities from the background

{% embed url="https://developer.android.com/guide/components/activities/background-starts" %}

Android 10 \(API level 29\) and higher -- place restriction on when apps can start activities when the app is running in the background

app running a foreground service also regarded as "in the background"

## Why restriction is needed

These restrictions help **minimize interruptions for the user** and **keep the user more in control** of what's shown on their screen.

Instead, **displaying time-sensitive notification** is recommended

* the user is shown a heads-up notification that allows them to respond
* can Respect the user's Do Not Disturb rules



## Exceptions

