---
layout: post
title: "Processes and Threads"
date: 2014-05-22 14:18:44 +0800
comments: true
categories: [Android]
---
> 注：此为毕业设计中学院要求的翻译与自己所做毕设相关且不少于2万字符英文原始资料的任务，由于自己毕设做的是Android方面的开发，所以决定翻译一下Android官方文档中Training和API Guides中的部分内容。由于水平有限，如有错误，望理解。

##[原文链接](http://developer.android.com/guide/components/processes-and-threads.html)
<br/>

当一个应用程序组件启动并且这个应用程序没有任何其它组件运行时，Android系统会为这个应用程序启动一个新的执行单线程的Linux进程。默认情况下，同一个应用程序的所有组件在同一个进程和线程（称为“主”线程）中运行。如果一个应用程序组件启动并且已经为那个应用程序存在一个进程（因为那个应用程序的另一个组件存在），则这个组件在那个进程内启动并且使用同一个执行线程。然而，你可以将应用程序的不同组件安排运行在不同的进程里，并且你可以为任何进程创建额外的线程。

这份文档讨论了进程和线程在一个Android应用程序里是如何工作的。

<!--more-->

## 进程

默认情况下，同一个应用程序的所有组件运行在同一个进程上并且大多数应用程序不应该改变这一点。然而，如果你发现你需要控制某个组件属于哪个进程，你可以在manifest文件里这样做。

组件元素每种类型的清单项——[\<activity>](http://developer.android.com/guide/topics/manifest/activity-element.html),[\<service>](http://developer.android.com/guide/topics/manifest/service-element.html),[\<receiver>](http://developer.android.com/guide/topics/manifest/receiver-element.html)和[\<provider>](http://developer.android.com/guide/topics/manifest/provider-element.html)——支持一个 android:process 属性可以指定一个那个组件运行的进程。你可以设置这个属性以便每个组件运行在它自己的进程上或者一些组件共享一个进程而其它的组件没有。你还可以设置 android:process 以便不同应用程序的组件运行在同一个进程里——只要这些应用程序共享相同的Linux用户ID和相同的证书签名。

[\<application>](http://developer.android.com/guide/topics/manifest/application-element.html)元素也支持 android:process 属性，设置一个默认值应用于所有的组件。

当内存不足和被其它进程要求更直接服务用户时，Android也许在某些时候决定关闭进程。在被杀进程中运行的应用程序组件因此被破坏。一个进程再次启动这些组件当再次有工作给它们做时。

当决定杀死哪些进程时，Android系统衡量它们对用户的相对重要性。例如，更容易关闭一个持有在屏幕上看不见的活动的进程，相对于持有可见活动的进程来说。因此，决定是否要终止一个进程，取决于在那个进程里运行的组件的状态。用于决定哪些进程终止的规则将在下面讨论。

### 进程生命周期

Android系统试图尽可能地保持一个应用程序的进程，但是最终为了新的或更重要的进程还是需要移除旧的进程来回收内存。要决定保持哪些进程和杀死哪些进程，系统基于运行在进程里的组件和这些组件的状态将每个进程放进了“重要性层级”。最低重要性的进程最先被消除，然后是这些次重要性的进程，然后依次类推，如果有必要回收系统资源。

有五个级别的重要性层级。下面的列表按重要性的顺序展示了不同类型的进程（第一个进程是最重要的且最后一个被杀死的）。

1.前台进程

一个用户当前正在做什么需要的进程。一个进程被认为是在前台如果下列任何情况为真：

* 它持有一个用户正在进行交互的[Activity](http://developer.android.com/reference/android/app/Activity.html)（[Activity](http://developer.android.com/reference/android/app/Activity.html)的[onResume()](http://developer.android.com/reference/android/app/Activity.html#onResume(\))方法被调用）。

* 它持有一个绑定到用户正在进行交互的活动的[Service](http://developer.android.com/reference/android/app/Service.html)。

* 它持有一个正在运行在前台的[Service](http://developer.android.com/reference/android/app/Service.html)——这个服务调用了[startForeground()](http://developer.android.com/reference/android/app/Service.html#startForeground(int, android.app.Notification\))方法。

* 它持有一个正在执行Service生命周期中的一个回调函数的[Service](http://developer.android.com/reference/android/app/Service.html)（[onCreate()](http://developer.android.com/reference/android/app/Service.html#onCreate(\))，[onStart()](http://developer.android.com/reference/android/app/Service.html#onStart(android.content.Intent, int\))，或者[onDestroy()](http://developer.android.com/reference/android/app/Service.html#onDestroy(\))）。

* 它持有一个正在执行[onReceive()](http://developer.android.com/reference/android/content/BroadcastReceiver.html#onReceive(android.content.Context, android.content.Intent\))方法的[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)。

通常，只有少数前台进程在任何给定的时间存在。它们被杀死仅仅是作为最后的手段——如果内存是如此之低以至于它们都不能继续运行。一般来说，在这一点上，该设备已经达到了内存分页状态，所以需要杀死某些前台进程来保持用户界面能够响应。

2.可见进程

一个没有任何前台组件，但是仍然能够影响用户在屏幕上看到什么的进程。一个进程被认为是可见的如果下列任何一个情况为真：

* 它持有一个不在前台的[Activity](http://developer.android.com/reference/android/app/Activity.html)，但是仍然对用户是可见的（它的[onPause()](http://developer.android.com/reference/android/app/Activity.html#onPause(\))方法被调用）。这可能会发生，例如，如果前台活动启动一个对话框，它允许前一个活动在它后面能看到。

* 它持有一个绑定到可见的（或前台的）活动的[Service](http://developer.android.com/reference/android/app/Service.html)。

一个可见进程被认为是及其重要的并且不会被杀死除非这样做是需要保持所有前台进程的运行。

3.Service 进程

一个正在运行着被[startService()](http://developer.android.com/reference/android/content/Context.html#startService(android.content.Intent\))方法启动的服务的进程并且不属于任何上两个更高的类别中的一个。尽管服务进程不会直接连接到用户看到的任何东西，它们一般都做用户关心的事情（例如在后台播放音乐或者在网络上下载数据），因此系统保持它们运行除非没有足够的内存保留它们和前台进程以及可见进程一起运行。

4.后台进程

一个持有对用户当前不可见活动的进程（活动的[onStop()](http://developer.android.com/reference/android/app/Activity.html#onStop(\))方法被调用）。这些进程对用户体验没有直接影响，并且系统可以在任何时候杀死它们来为前台、可见或是服务进程回收内存。通常情况下有很多正在运行的后台进程，因此它们保持着一个LRU（最近最少使用）清单来确保最经常被用户看见的活动的进程最后一个被杀死。如果活动正确地实现了它的生命周期方法，并且保存其当前状态，杀死它的进程不会对用户体验有明显的效果，因为当用户导航返回这个活动时，活动恢复了其所有的可见状态。请参阅有关保存和恢复状态信息的[Activities](http://developer.android.com/guide/components/activities.html#SavingActivityState)文档。

5.空进程

一个不再持有任何活动的应用程序组件的进程。保持这种进程活着的唯一原因是高速缓存的目的，以提高一个需要运行在它里面的组件的下次的启动时间。为了平衡进程缓存和底层内核缓存之间的整个系统资源系统会经常杀死这些进程。

Android 基于进程中当前活动的组件的重要性把一个进程排在它能排到的最高层级上。例如，如果一个进程持有一个服务和可见的活动，这个进程会被排为可见进程而不是服务进程。

另外，一个进程的排名可能会上升因为其它进程都依赖于它——一个正在服务其它进程的进程永远不可能比它正在服务的进程的排名低。例如，如果进程A中一个的content provider正在服务进程B的一个客户，或者如果进程A中的一个服务绑定到了进程B中的一个组件中，进程A总是被认为至少跟进程B一样重要。

因为运行一个服务的进程比运行后台活动的进程的排名高，所以一个初始化了长期操作的活动为那个操作启动一个服务可能会做得很好，而不是简单地创建一个工作线程——特别是如果那个操作很可能拖垮这个活动。例如，一个向网站正在上传图片的活动应该启动一个服务去执行上传以便上传可以在后台继续即使用户离开了这个活动。使用服务保证了操作至少具有“服务进程”的优先级，不管活动发生了什么。广播接收器应该采用服务而不是简单地把耗时的操作放在一个线程里是同样的道理。

## 线程

当应用程序启动时，系统会创建一个执行应用程序的线程，称为“主线程”。这个线程是非常重要的因为它负责给合适的用户界面部件调度事件，包括绘画事件。它也是应用程序和Android UI工具包里的组件([android.widget](http://developer.android.com/reference/android/widget/package-summary.html)和[android.view](http://developer.android.com/reference/android/view/package-summary.html)包里的组件)交互的线程。这样，主线程有时也被称为UI线程。

系统不会为一个组件的每个实例创建一个单独的线程。所有运行在同一个进程中的组件都在UI线程里初始化，并且系统调用每个从那个线程分发的组件。因此，响应系统的回调方法（例如用[onKeyDown()](http://developer.android.com/reference/android/view/View.html#onKeyDown(int, android.view.KeyEvent\)) 来报告用户活动或是一个生命周期回调方法总是在进程中的UI线程里运行。

例如，当用户触摸屏幕上一个按钮时，应用程序的UI线程给这个部件调度触摸事件，从而设置其按下状态和将一个无效请求发送到事件队列中。UI 线程出队队列中的请求并通知这个部件它应该重绘自己。

当应用程序响应用户交互进行深入细致的工作时，这种单一的线程模型可以产生性能差除非正确地实现应用程序。特别是，如果一切都发生在UI线程，执行长时间的操作如网络访问和数据库查询将会阻塞整个UI。当这个线程被阻塞时，没有事件可以被分发出来，包括绘制事件。从用户的角度来看，应用程序似乎挂起。更糟的是，如果UI线程被阻塞超过几秒钟（目前大约是5秒）会给用户呈现臭名昭著的"[application not responding](http://developer.android.com/guide/practices/responsiveness.html)" (ANR) 对话框。用户也许会决定退出程序并且如果他们不开心的话会卸载程序。 

此外，Android UI工具包不是线程安全的。所以，不能在一个工作线程里操作UI——对用户界面的所有操作必须都在UI线程里执行。因此，Android的单线程模型有简单的两条规则：

1. 不要阻塞UI线程。

2. 不要从UI线程以外的线程里访问Android UI工具包。

### 工作线程

因为上述的单线程模型，对不阻塞UI线程的应用程序UI的响应是至关重要的。如果你有不是瞬时的操作，你应该确保在单独的线程（“后台”或“工作”线程）里执行它们。

例如，下面是在一个单独的线程里下载一张图片并显示在[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)里的点击监听器的一段代码：

{% codeblock lang:java %}
public void onClick(View v) {
    new Thread(new Runnable() {
        public void run() {
            Bitmap b = loadImageFromNetwork("http://example.com/image.png");
            mImageView.setImageBitmap(b);
        }
    }).start();
}
{% endcodeblock %}

起初，这似乎工作得很好，因为它会创建一个新的线程去处理网络操作。然而，它违反了单线程模型的第二条规则：不要在UI线程以外的线程里访问Android UI工具包——这个样例在工作线程里而不是UI线程里修改[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)。这可能导致不确定的和意外的行为，这追查起来可能是困难且耗时的。

要解决这个问题，Android提供了几种方法从其它线程里来访问UI线程。以下是一个可以帮助的方法的列表：

* [Activity.runOnUiThread(Runnable)](http://developer.android.com/reference/android/app/Activity.html#runOnUiThread(java.lang.Runnable\))

* [View.post(Runnable)](http://developer.android.com/reference/android/view/View.html#post(java.lang.Runnable\))

* [View.postDelayed(Runnable, long)](http://developer.android.com/reference/android/view/View.html#postDelayed(java.lang.Runnable, long\))

例如，你可以使用[View.post(Runnable)](http://developer.android.com/reference/android/view/View.html#post(java.lang.Runnable\))方法修复上面的代码：

{% codeblock lang:java %}
public void onClick(View v) {
    new Thread(new Runnable() {
        public void run() {
            final Bitmap bitmap = loadImageFromNetwork("http://example.com/image.png");
            mImageView.post(new Runnable() {
                public void run() {
                    mImageView.setImageBitmap(bitmap);
                }
            });
        }
    }).start();
}
{% endcodeblock %}

现在这个实现是线程安全的：网络操作在一个单独的线程里完成而[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)在UI线程里操作。

然而，随着操作的复杂性的增加，这种代码可以变得复杂且难以维护。为了处理和一个工作线程更复杂的操作，你可以考虑在工作线程里用[Handler](http://developer.android.com/reference/android/os/Handler.html)来处理从UI线程里传递的消息。但是，最好的解决方法是用[Asynctask](http://developer.android.com/reference/android/os/AsyncTask.html)的扩展类，它简化了需要与UI交互的工作线程任务的执行。

### 使用 AsyncTask

[Asynctask](http://developer.android.com/reference/android/os/AsyncTask.html)允许在用户界面上异步工作。它在一个工作线程里执行阻塞操作然后将结果发布到UI线程上来，而不需要自己来处理线程和/或 handlers。

要使用它，你必须继承[Asynctask](http://developer.android.com/reference/android/os/AsyncTask.html)并实现[doInBackground()](http://developer.android.com/reference/android/os/AsyncTask.html#doInBackground(Params...\))回调方法，它在一个后台线程池里运行。要更新UI，你应该实现[onPostExecute()](http://developer.android.com/reference/android/os/AsyncTask.html#onPostExecute(Result\))方法，它传递[doInBackground()](http://developer.android.com/reference/android/os/AsyncTask.html#doInBackground(Params...\))方法里的结果且运行在UI 线程里，所以你可以安全地更新你的UI。然后你可以通过在UI线程里调用[execute()](http://developer.android.com/reference/android/os/AsyncTask.html#execute(Params...\))方法运行这个任务。

例如，你可以这么用[Asynctask](http://developer.android.com/reference/android/os/AsyncTask.html)实现上一个例子：

{% codeblock lang:java %}
public void onClick(View v) {
    new DownloadImageTask().execute("http://example.com/image.png");
}

private class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {
    /** The system calls this to perform work in a worker thread and
      * delivers it the parameters given to AsyncTask.execute() */
    protected Bitmap doInBackground(String... urls) {
        return loadImageFromNetwork(urls[0]);
    }
    
    /** The system calls this to perform work in the UI thread and delivers
      * the result from doInBackground() */
    protected void onPostExecute(Bitmap result) {
        mImageView.setImageBitmap(result);
    }
}
{% endcodeblock %}

现在UI是安全的且代码是简单的，因为它将这项工作分离成了应该在工作线程里完成的部分和应该在UI线程里完成的部分。

要充分了解如何使用这个类你应该阅读[Asynctask](http://developer.android.com/reference/android/os/AsyncTask.html)参考，但下面是它是如何工作的简要概述：

* 你可以指定参数的类型，进展值和任务的最终值，泛型使用。

* [doInBackground()](http://developer.android.com/reference/android/os/AsyncTask.html#doInBackground(Params...\))方法在工作线程里会自动执行。

* [onPreExecute()](http://developer.android.com/reference/android/os/AsyncTask.html#onPreExecute(\)),[onPostExecute()](http://developer.android.com/reference/android/os/AsyncTask.html#onPostExecute(Result\))和[onProgressUpdate()](http://developer.android.com/reference/android/os/AsyncTask.html#onProgressUpdate(Progress...\))方法都在UI线程里调用。

* [doInBackground()](http://developer.android.com/reference/android/os/AsyncTask.html#doInBackground(Params...\))方法返回的值被发送到[onPostExecute()](http://developer.android.com/reference/android/os/AsyncTask.html#onPostExecute(Result\))方法里。

* 你可以在[doInBackground()](http://developer.android.com/reference/android/os/AsyncTask.html#doInBackground(Params...\))方法里在任何时间调用[publishProgress()](http://developer.android.com/reference/android/os/AsyncTask.html#publishProgress(Progress...\))来在UI线程里执行[onProgressUpdate()](http://developer.android.com/reference/android/os/AsyncTask.html#onProgressUpdate(Progress...\))方法。

* 你可以在任何时间，从任何线程里取消任务。

> 注意：当你使用一个在活动里由于[runtime configuration change](http://developer.android.com/guide/topics/resources/runtime-changes.html)（例如当用户改变了屏幕方向）而意外重新启动的工作线程时，你可能遇到的另一个问题，它也许会销毁你的工作线程。要看如何在这些重启动的活动之一期间保持你的任务和当活动被销毁时如何正确地取消任务，请参阅[Shelves](http://code.google.com/p/shelves/)样例中的源代码。

### 线程安全的方法

在某些情况下，你实现的方法可能会从多个线程里调用，因此写入必须是线程安全的。

这主要适用于可以被称为远程的方法——例如[bound service](http://developer.android.com/guide/components/bound-services.html)里的方法。当一个实现了[IBinder](http://developer.android.com/reference/android/os/IBinder.html)方法的调用源自于这个[IBinder](http://developer.android.com/reference/android/os/IBinder.html)运行的同样的进程中时，这个方法就在调用者的线程里执行。然而，当这个调用源自于另一个进程时，这个方法就在系统维护的和运行[IBinder](http://developer.android.com/reference/android/os/IBinder.html)同一个进程里的线程池里选择一个线程运行（它不会在进程的 UI 线程里执行）。 例如，尽管一个服务的[onBind()](http://developer.android.com/reference/android/app/Service.html#onBind(android.content.Intent\))方法会从这个服务的进程的UI线程调用，但是在[onBind()](http://developer.android.com/reference/android/app/Service.html#onBind(android.content.Intent\))返回（例如，一个实现了RPC方法的子类）的这个对象里实现的方法会再线程池里调用。因为一个服务可以有多个客户，池中的多个线程可以同时占用同一个[IBinder](http://developer.android.com/reference/android/os/IBinder.html)。因此，[IBinder](http://developer.android.com/reference/android/os/IBinder.html)方法必须被实现为线程安全的。

类似的，一个content provider可以接收其它进程的数据请求。尽管[ContentResolver](http://developer.android.com/reference/android/content/ContentResolver.html) 和[ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html)类隐藏了进程间通信是如何管理的细节，响应这些请求的[ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html)方法——[query()](http://developer.android.com/reference/android/content/ContentProvider.html#query(android.net.Uri, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String\)), [insert()](http://developer.android.com/reference/android/content/ContentProvider.html#insert(android.net.Uri, android.content.ContentValues\)), [delete()](http://developer.android.com/reference/android/content/ContentProvider.html#delete(android.net.Uri, java.lang.String, java.lang.String[]\)), [update()](http://developer.android.com/reference/android/content/ContentProvider.html#update(android.net.Uri, android.content.ContentValues, java.lang.String, java.lang.String[]\))，和[getType()](http://developer.android.com/reference/android/content/ContentProvider.html#getType(android.net.Uri\))方法——在这个content provider的进程的一个线程池里调用，而不是这个进程的UI线程里调用。因为这些方法可以同时被任意数量的线程调用，所以它们也必须被实现为线程安全的。

### 进程间通信

Android提供了一种机制，使用远程过程调用（RPC）来进程间通信（IPC），其中一个方法被一个activity或其它应用程序组件调用，但是远端执行（另一个进程中），并返回任何结果给调用者。这需要分解一个方法的调用和它的数据到一个操作系统能理解的层级，从本地进程和地址空间传送到元曾进程和空间地址，然后重新组装和在那里重新再次调用。然后返回值在相反的方向上传输。Android提供了所有执行这些IPC交易的代码，所以你可以专注于定义和实现RPC编程接口。

要执行IPC，你的应用程序必须使用[bindService()](http://developer.android.com/reference/android/content/Context.html#bindService(android.content.Intent, android.content.ServiceConnection, int\))绑定到一个服务。欲了解更多信息，请参阅[Services](http://developer.android.com/guide/components/services.html)开发指南。





















