---
layout: post
title: "Activity Lifecycle"
categories: Android
---
![enter image description here](https://i.imgur.com/3HpooIP.png)

*Activities in the system are managed as an activity stack. When a new activity is started, it is placed on the top of the stack and becomes the running activity -- the previous activity always remains below it in the stack, and will not come to the foreground again until the new activity exits.*

**An activity has essentially four states:**

 1. If an activity in the foreground of the screen (at the top of the stack), it is active or running.
 2. If an activity has lost focus but is still visible (that is, a new non-full-sized or transparent activity has focus on top of your activity), it is paused. A paused activity is completely alive (it maintains all state and member information and remains attached to the window manager), but can be killed by the system in extreme low memory situations.
 3. If an activity is completely obscured by another activity, it is stopped. It still retains all state and member information, however, it is no longer visible to the user so its window is hidden and it will often be killed by the system when memory is needed elsewhere.
 4. If an activity is paused or stopped, the system can drop the activity from memory by either asking it to finish, or simply killing its process. When it is displayed again to the user, it must be completely restarted and restored to its previous state.


![Activity Lifecycle](https://i.imgur.com/3HpooIP.png)

