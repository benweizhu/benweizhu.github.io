---
layout: post
title: "Android Service(译)"
date: 2014-01-11 23:51
comments: true
categories: 
---
Service是Android应用程序中的一个组件，它能够长时间在后台运行，并且不提供界面。另一个应用程序的组件可以启动一个Service并持续在后台运行即便用户切换到别的应用程序。而且，组件可以和Service绑定并与它进行交互，甚至执行进程间通信（IPC）。例如，一个Service可以处理网络通信，播放音乐，执行IO操作，或者和启动它的组件交互，这些都是在后台进行。

使用Service有两种方式：

##启动Start

当一个应用组件（例如Activity）调用startService()方法时，可以启动一个Service。一旦被启动，一个Service会无止境的运行在后台，即使启动它的组件被销毁了。通常启动一个Service会去执行一个单独的操作，并且不会返回任何的结果给调用者。例如，它可以下载或者上传文件。当这件事情完成，Service自己会停止。

##绑定Bound

当一个应用组件调用bindService()方法时，可以绑定一个Service。一个被绑定的Service提供了一个CS模式的接口，它允许组件与Service进行交互，发送请求，返回结果，与进程间通信。被绑定的Service会一直运行只要有其他应用组件绑定它。多个组件可以绑定到同一个Service，但是当所有的组件都解除绑定，该Service就会被销毁。

对应于这两种方式，有两个回调函数去实现，onStartCommand()对应启动方式，onBind()对应绑定方式。

无论你的应用程序是启动还是绑定一个Service还是两种方式有用到，对于任何一个应用程序都可以使用该Service（即便是另外一个应用程序）。同理，任何组件也可以使用一个Activity（通过Intent）。然而，你可以在Manifest文件中将Service定义为私有的，并禁止其他应用访问它。

Service运行在宿主进程的的主线程中——Service不会创建它自己的进程，也不会运行在分开的进程中（除非你特别指定）。这意味着，如果你的Service要做任何占用CPU或者阻塞操作（例如播放音乐或者网络操作），你应该单独为Service创建一个线程。通过使用单独的线程，你会减小应用程序未响应（ANR-Application Not Responding）错误，并且让应用程序主线程保持用户与Activity进行交互。

创建Service，你必须创建一个Service的子类（或者一个已经存在的子类）。在实现类中，你需要重写一些回调函数，这些回调函数处理着Service的生命周期，并且提供一个机制让组件去绑定Service。

**onStartCommand()**

该方法被调用，当另一个组件，例如Activity通过调用startService()，请求一个Service启动。一旦该方法被执行，Service会启动，并在后台无止境运行。如果你实现这个方法，你就应该负责停止它（通过调用stopSelf()或者stopService()，但如果是绑定，就不用实现这些方法）。

**onBind()**

该方法被调用，当一个组件想要绑定该Service（例如执行RPC），需要调用bindService()。在这个实现方法中，你需要提供一个接口（返回一个IBinder）给调用者，以便调用者与Service进行交互。你必须实现这方法，但是如果你不希望该Service被绑定，那么你可以放回一个null。

**onCreate()**

该方法被调用，当Service被第一次创建，要去执行一次性的配置过程（在该方法被调用的要么是onStartCommand()或者onBind()）。如果一个Service已经在运行，则这个方法不会再被调用。

**onDestroy()**

当Service不在被使用时，系统会去调这个方法。你的Service应该执行一些清理资源的工作，例如线程，已注册的监听器等。它是Service最后被调用的方法。

Android系统会强制停止一个Service，仅仅当内存不足且系统必须会已经有用户焦点的Activity恢复一些系统资源时。如果Service绑定到一个拥有用户焦点的Activity，那么一般不会被杀掉，如果该Service被定义在后台运行，那么它几乎不会被杀掉。否则，如果Service被启动且长时间运行，那么系统会降低它在后台任务中的位置，然后该Service会变成最容易被杀掉的服务。如果你的Service被启动，那么你必须将它优雅的设计成能够被系统重启。如果系统杀掉你的Service，它会在资源够用的时候立刻重启你的Service（虽然它也依赖于onStartCommand()方法返回的值）。

##LifeCycle

![Jasmine](http://developer.android.com/images/service_lifecycle.png)

Service的整个生命周期发生在onCreate()方法和onDestory()方法之间，就像Activity一样，分别进行一些初始化工作和清理资源工作。且无论是启动Service还是绑定Service，这两个方法都会被调用到。

Service活动周期开始于onStartCommand()或者onBind()方法，每一个方法都会被传入一个Intent对象。

如果Service以启动方式开始，那么活动周期与整个生命周期同时结束。如果Service是绑定的，那么活动周期是以onUnbind()方法结束。
{% codeblock Service lang:java %}
import android.app.Service;
import android.content.Intent;
import android.os.IBinder;

public class ExampleService extends Service {
    int mStartMode;       // indicates how to behave if the service is killed
    IBinder mBinder;      // interface for clients that bind
    boolean mAllowRebind; // indicates whether onRebind should be used

   @Override
    public void onCreate() {
        // The service is being created
    }

   @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // The service is starting, due to a call to startService()
        return mStartMode;
    }

   @Override
    public IBinder onBind(Intent intent) {
        // A client is binding to the service with bindService()
        return mBinder;
    }

   @Override
    public boolean onUnbind(Intent intent) {
        // All clients have unbound with unbindService()
        return mAllowRebind;
    }

   @Override
    public void onRebind(Intent intent) {
        // A client is binding to the service with bindService(),
        // after onUnbind() has already been called
    }

   @Override
    public void onDestroy() {
        // The service is no longer used and is being destroyed
    }
}
{% endcodeblock %}


文献参考自：http://developer.android.com/guide/components/services.html

