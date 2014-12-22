---
layout: post
title: "About Android Activity LifeCycle(译)"
date: 2014-01-05 23:22
comments: true
categories: 
---
![Jasmine](http://developer.android.com/images/activity_lifecycle.png)

Activity是应用的一个组件，它提供用户可以交互的屏幕，例如拨打电话，照相，发邮件，看地图。每一个Activity都会被赋予一个窗户用来绘画用户界面。这个窗口通常会占据整个屏幕，但是也可以比屏幕小和悬浮在其他窗口之上。

一个应用程序通常由多个Activity组成，它们之间以一种较为松弛的关系联系在一起。例如，一个Activity作为主要的Activity，它会作为程序启动时第一个展现给用户的Activity。每一个Activity能够启动另外一个Activity，以便去执行其他的动作。每一次一个新的Activity启动，前一个Activity就会停止，但是系统会将它保存在一个栈中（“back stack”）。当一个新的Activity启动时，当前的Activity会被压入栈中，并拿走焦点。这个栈遵守“后进先出”原则，所以，当用户在当前Activity完成对应操作，按下返回按钮的时候，它会从栈顶弹出，并销毁，然后前一个Activity恢复。

当一个Activity因为另一个新的Activity启动而停止时，系统会通过Activity生命周期中的回调函数通知它这个状态的变化。由于状态的改变，Activity会接受到多个回调函数，无论系统是在create，stop，resume还是destroy，并且每一次回调都提供你一个机会去执行适合这个状态变化的操作。例如，当stop时，你的Activity应该释放任何比较大的对象，例如网络和数据库连接。当Activity恢复时，你应该从新获取必要的资源和恢复之前被打断的操作。状态的转换是Activity生命周期的一部分。

通过回调函数去管理Activity的生命周期是实现健壮和灵活应用的关键。Activity的生命周期直接受到其他Activity，Task以及back stack的影响。

一个Activity可以存在于三个状态中：

*Resumed*

该状态下，Activity置于屏幕前景，并拥有聚焦，可以把这个状态理解为正在运行。

*Paused*

该状态下，另一个Activity正置于屏幕前景并拥有聚焦，但是当前Activity仍然可见。一个暂停的Activity是处于完全存活状态的（Activity对象仍然保有内存，它维持状态和成员信息，并保持与Window Manger的关系），但是如果内存不足时，系统有权利关闭该Activity。

*Stopped*

该状态下，该Activity完全被另一个Activity遮挡（该Activity就会进入到背景状态）。进入stopped状态的Activity也是仍然处于存活状态。然而，它对用户是不可见的，同样在系统需要内存的时候，它会被关闭。

如果一个Activity进入paused或者stopped状态，系统可以通过调用finish方法或者终止进程的方式，将Activity从内存中清除。当该Activity从新打开，他必须要重新创建一次。

Activity**完整的生命周期**发生于onCreate()方法和onDestroy()方法之间。Activity应该在onCreate()方法时，设置一些全局状态，例如布局，并且在onDestroy()方法释放所有保持的资源。例如，如果Activity拥有一个线程在后台运行并正在从网络上下载数据，该线程可能是在onCreate()方法中创建的，那么就应该在onDestroy()方法中停止该线程。

Activity的**可见生命周期**发生于onStart()方法和onStop()方法之间。在这段时间，用户可以在屏幕上看见它，并进行交互。例如，onStop()方法被调用发生在当一个新的Activity启动，而当前Activity不在可见时。在这两个方法之间，你可以保有Activity所需要的资源。例如，你可以在onStart()方法调用时，注册一个BroadCartReceiver，让他监控你对UI的操作，并在用户不在看到这个Activity时，在onStop()方法中注销这个BroadCartReceiver。

Activity**前景生命周**发生于onResume()方法和onPause()方法之间。在这段时间中，该Activity处于所有Activity的上方并且拥有用户输入焦点。Activity能够频繁的在前景中出镜和入镜。例如，当设备进入睡眠或者有一个对话框弹出时，在onPause()方法被调用时。因为，这个状态频繁的转变，在这两个方法中的代码要尽量的轻量级，以避免转换太慢，让用户等待。

|Method|Description|Killable after?|Next
| ------------ | ------------- | ------------ |
|onCreate()|当Activity第一次被创建时调用。在这里，你应该做所有关于静态配置的事情-创建View，绑定数据到List等等。这个方法会传入一个Bundle对象，如果Activity中有捕获自身状态，那么它包含了Activity之前的状态。|No|onStart()
|onRestart()| 在Activity停止之后被调用，指明Activity会被重新启动。|No|onStart()
|onStart()|在Activity变成用户可见状态之前调用。如果Activity要变成前景状态，那么onResume()方法会被调用，如果Activity要被隐藏，则onStop()方法会被调用。|No|onResume() or onStop()
|onResume()|在Activity开始与用户交互之前调用。这个时候，Activity会置于栈顶，并伴随着用户输入。|No|onPause()
|onPause()|在系统开始要恢复另一个Activity时被调用。该方法通常被用于提交未保存数据到持久数据中，停止动画和其他会占用CPU资源的操作。并且应该非常迅速的做这些事情，因为下一个Activity只有在当前方法执行完成之后才会恢复。如果Activity回到前端，则onResume()方法跟在onPaused()方法后面，如果对用户不可见，则onStop()方法跟在后面。|Yes|onResume() or onStop()
|onStop()|当Activity对用户不在可见时被调用。onStop()发生的原因是Activity被销毁，或者另一个Activity被恢复并正在覆盖当前Activity。如果紧跟随着的是onRestart()方法，则说明Activity正在恢复与用户的交互。如果跟着的是onDestory()方法，则说明这个Activity要被销毁了。|Yes|onRestart() or onDestroy()
|onDestroy()|在Activity被销毁时调用。这是Activity最后一个会接收到的调用。被调用的原因要么是因为finish()方法在哪个位置调用，或者因为系统因为要保留资源而临时销毁它。可以通过isFinishing()方法来区分两种不同情况。|Yes|nothing




文献参考自：http://developer.android.com/guide/components/activities.html