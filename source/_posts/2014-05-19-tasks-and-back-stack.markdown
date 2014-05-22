---
layout: post
title: "Tasks and Back Stack"
date: 2014-05-19 12:27:49 +0800
comments: true
categories: [Android]
---
> 注：此为毕业设计中学院要求的翻译与自己所做毕设相关且不少于2万字符英文原始资料的任务，由于自己毕设做的是Android方面的开发，所以决定翻译一下Android官方文档中Training和API Guides中的部分内容。由于水平有限，如有错误，望理解。

##[原文链接](http://developer.android.com/guide/components/tasks-and-back-stack.html)
<br/>

一个应用程序通常包含多个[活动](http://developer.android.com/guide/components/activities.html)。每个活动应该围绕一个用户可以执行和开始其它的活动特定的动作来设计。例如，电子邮件应用程序可能有一个展示新邮件的列表的活动。当用户选择一封邮件时，一个新的活动被打开来查看这封邮件。
<!--more-->
一个活动甚至能启动设备上其它应用程序中存在的活动。例如，如果你的应用程序想要发送一封邮件，你可以定义一个意图来执行“发送”任务并包含一些数据，例如邮件地址和消息。另一个应用程序的活动声明自己处理这种意图然后打开邮件。在这种情况下，意图是发送一封邮件，所以邮件应用程序的 "compose" 活动启动(如果有多个活动支持相同的意图)然后系统让用户选择使用哪一个。当邮件被发送出后，你的活动恢复并且似乎邮件的活动看起来是你的应用程序的一部分。尽管这些活动可能来自不同的应用程序，Android 通过保持两个活动在同一任务中维护了这种无缝的用户体验。

任务是当用户执行某项作业时和用户交互的活动的集合。这些活动以被打开的顺序安排在一个栈(“回退栈”)中。

设备的主屏幕是大多数任务的起点。当用户按下应用启动器中的图标(或主屏幕上的快捷方式)时，该应用程序的任务来到前台。如果应用程序不存在任务，然后一个新的任务会被创建并且那个应用程序的"主"活动打开并作为栈中的根活动。

当当前的活动启动另一个活动时，新的活动被压入堆栈的顶部并获取焦点。前一个活动保持在栈中，但被停止了。当一个活动停止时，系统保留其用户接口的当前状态。当用户按下返回按钮时，当前的活动从栈顶弹出(这个活动就被销毁了)并且前一个活动恢复(其之前的UI状态恢复)。栈中的活动从来没有重新排列，仅仅是从栈中压栈和出栈——当被当前的活动启动时压入栈和当用户用后退按钮离开它时弹出栈。因此，回退栈是作为一个“后进先出”的对象结构操作着。图 1 用展示了活动和当前回退栈在每个时间点的过程的时间线可视化了这种行为。 

{% img /images/2014-5-19/1.png 图 1 %}

图1：表示了任务中每一个新的活动是怎样在回退栈中增加了一个项目。当用户按下返回按钮时，当前的活动被销毁，前一个活动恢复。

如果用户继续按返回，然后再堆栈中的每个活动都弹出来显示前一个，直到用户返回到主屏幕(或者任何任务开始时正在运行的活动)。当所有的活动从堆栈中移除后，这个任务就不再存在。

{% img right /images/2014-5-19/2.png 图 2 %}
任务是一个当用户开始新任务或通过 *Home* 按钮去到主屏幕时可以移动到后台的内聚单位。当在后台时，堆栈中所有的活动被停止，但是回退栈的任务保持不变——任务仅仅是失去了焦点当另一个任务替换了它时，如图 2 所示。再一个任务可以返回到“前台”使用户可以在它们停止的地方获得它们。假设，例如，当前的任务(A任务)在它的堆栈里有三个活动——两个在当前这个活动之下。用户按了主 按钮，然后从应用启动器里启动了一个新的任务。当主屏幕出现时，A 任务进入后台。当新的应用程序启动时，系统为那个应用程序启动了一个带有自己的活动堆栈的任务(B任务)。与应用程序交互后，用户再次返回主页并选择最初启动的 A 任务的应用程序。现在 A 任务来到了前台——栈中所有的三个活动完好并且栈顶的活动恢复。此时，用户还可以通过返回主页选择启动了那个任务的应用程序图标(或通过从最近使用程序屏幕选择那个应用程序的任务)切回到 B 任务。这是一个 Android 上多任务的例子。

> 注：多个任务在后台只能保留一次。然而，如果用户同一时间运行多个后台任务，系统可能会为了恢复内存开始销毁后台活动，导致活动状态丢失。详情请参见 [Activity state](http://developer.android.com/guide/components/tasks-and-back-stack.html#ActivityState) 部分。

{% img right /images/2014-5-19/3.png 图 3 %}
因为在回退栈中的活动永远不会重新排列，如果你的应用程序允许用户从多个活动启动特定的活动，那么一个新的活动实例会被创建并且压入栈中(而不是把这个活动之前的任何实例拿到栈顶来)。因此，在你的应用程序中一个活动可能会被多次实例化(即使来自不同的任务)，如图 3 所示。因此，如果用户使用后退按钮向后导航，活动的每个实例(和每个的UI状态)会按它们被打开的顺序显示。然而，如果你不想一个活动被实例化多次你可以改变这种行为。怎么做在接下来关于 [Managing Tasks](http://developer.android.com/guide/components/tasks-and-back-stack.html#ManagingTasks) 的章节中会讨论。

活动和任务的默认行为的总结：

* 当活动 A 启动活动 B 时，活动 A 被停止，但系统会保留它的状态(例如滚动条的位置和输入到表单中的文字)。如果用户在活动 B 中按了后退按钮，活动 A 和它被保留的状态一起恢复。

* 当用户按下主按钮离开任务时，当前的活动被停止并且它的任务进入后台。系统保留任务中每个活动的状态。如果用户之后通过选择启动图标开始那个任务恢复任务，这个任务会进入前台并且恢复在栈顶的那个活动。

* 如果用户按了返回按钮，当前的活动从栈顶弹出并且销毁。栈里前一个活动被恢复。当一个活动被销毁时，系统不会保留活动的状态。

* 活动可以被实例化多次，甚至从其它任务上。

> 设计导航

> 欲了解更多有关应用程序导航在 Android 上是如何工作的，请阅读 Android 设计[导航](http://developer.android.com/design/patterns/navigation.html)指导。

## 保存活动状态

正如上面讨论的，系统的默认行为会保存一个活动的状态当它停止时。这样，当用户导航返回到前一个活动时，它的用户界面展示的是它消失时的样子。然而，你能——并且也应该——使用回调方法主动保留活动的状态，以防活动被销毁后必须重新创建。

当系统停止你的其中一个活动时(例如当一个新的活动启动或任务进入了后台)，系统如果为了恢复系统内存也许会完全地销毁那个活动。当这种情况发生时，活动的状态信息也就丢失了。如果发生了这种情况，系统仍然知道这个活动在回退栈中有一个位置，但是当这个活动被拿到栈顶时系统必须重新创建它(而不是恢复它)。为了避免丢失用户的工作，你应该在活动里通过实现 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle)) 回调方法主动保留用户的工作。

有关更多如何保存活动状态的信息，请参见 [Activities](http://developer.android.com/guide/components/activities.html#SavingActivityState) 文档。

## 任务管理

如上所述，Android 通过将所有的活动逐次地放进同样的任务中和一个“先进先出”的堆栈中的方式来管理任务和回退栈——这对大多数应用程序有效并且你不必担心你的活动和任务是怎么关联的以及它们在回退栈中是怎么存在的。然而，你也许会决定要中断正常的行为。也许你想要应用程序的活动当它启动的时候是开始一个新的任务(而不是被放置在当前的任务内)。或者，当你开始一个新的活动，你想要将已经存在的实例拿到前面来(而不是在回退栈的顶部创建一个新的实例)。或者，你想要你的回退栈当用户离开任务时除了根活动外清除所有的活动。

你可以通过 [\<activity>](http://developer.android.com/guide/topics/manifest/activity-element.html) manifest 元素的属性和传递给 [startActivity()](http://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent\)) 的意图的标志做这些事情和更多这样的事情。

在这方面，你可以使用的主要的 [\<activity>](http://developer.android.com/guide/topics/manifest/activity-element.html) 属性如下：

[taskAffinity](http://developer.android.com/guide/topics/manifest/activity-element.html#aff)

[launchMode](http://developer.android.com/guide/topics/manifest/activity-element.html#lmode)

[allowTaskReparenting](http://developer.android.com/guide/topics/manifest/activity-element.html#reparent)

[clearTaskOnLaunch](http://developer.android.com/guide/topics/manifest/activity-element.html#clear)

[alwaysRetainTaskState](http://developer.android.com/guide/topics/manifest/activity-element.html#always)

[finishOnTaskLaunch](http://developer.android.com/guide/topics/manifest/activity-element.html#finish)

和你可以使用的主要的意图标志如下：

[FLAG_ACTIVITY_NEW_TASK](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK)

[FLAG_ACTIVITY_CLEAR_TOP](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TOP)

[FLAG_ACTIVITY_SINGLE_TOP](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_SINGLE_TOP)

在接下来的章节里，你将会看到如何使用这些清单属性和意图标志来定义活动和任务是如何相关的以及活动在回退栈中是如何表现的。

> 注意：大多数应用程序不应该打断活动和任务的默认行为。如果你确定有必要修改活动的默认行为，在启动和用返回按钮从其它的活动和任务中导航返回时谨慎使用并且确保测试活动的可用性。确保测试导航行为可能与用户预期的行为发生生冲突的情况。

### 定义启动模式

启动模式允许你定义一个活动的新实例如何与当前任务相关联。你可以通过两种方式定义不同的启动模式：

[使用清单文件](http://developer.android.com/guide/components/tasks-and-back-stack.html#ManifestForTasks)

当你在清单文件中声明一个活动时，你可以当活动启动的时候指定活动如何与任务相关联。

[使用意图标志](http://developer.android.com/guide/components/tasks-and-back-stack.html#IntentFlagsForTasks)

当你调用 [startActivity()](http://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent\)) 时，你可以在[意图](http://developer.android.com/reference/android/content/Intent.html)里包含一个声明了新的活动如何(或是否)与当前的任务相关联的标志。

因此，如果活动 A 启动活动 B， 活动 B 可以在其清单定义应该如何与当前任务相关联(如果有的话)和活动 A 还可以要求活动 B 应该如何与当前任务相关联。如果这两个活动都定义了活动 B 应该如何与任务相关联，那么活动 A 的请求(如在意图里定义的)高于活动 B 的请求(如在清单中定义的)。

> 注：一些适用于清单文件的启动模式并不是适用于意图标志，同样，一些适用于意图标志的启动模式也不适用于清单文件。

#### 使用清单文件

当在清单文件声明活动时，你可以用 [\<activity>](http://developer.android.com/guide/topics/manifest/activity-element.html) 元素的[启动模式](http://developer.android.com/guide/topics/manifest/activity-element.html#lmode)属性指定活动应该如何与任务相关联。

该[启动模式](http://developer.android.com/guide/topics/manifest/activity-element.html#lmode)属性指定一个活动应该如何启动进任务中的指令。有四种不同的启动模式你可以给[启动模式](http://developer.android.com/guide/topics/manifest/activity-element.html#lmode)属性赋值：

"standard"(默认模式)

默认。从任务里启动活动和发送意图给活动系统会创建这个活动的一个新实例。活动会被多次实例化，每个实例可以属于不同的任务，并且一个任务可以有多个实例。

"singleTop"

如果活动的一个实例已经在当前任务的栈顶存在，系统会通过调用它的 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 方法给那个实例发送一个意图，而不是创建活动的一个新实例。该活动可以被实例化多次，每个实例可以属于不同的任务，一个任务可以有多个实例(但仅当回退栈顶部的活动不是一个该活动已存在的实例)。

例如，假设一个任务的回退栈由根活动 A 和活动 B，活动 C 和顶部的活动 D 组成（栈的顺序为 A-B-C-D; D在顶部）。一个意图过来请求 D 类型的活动。如果 D 有默认的 "standard" 启动模式，一个类的新实例会被启动并且栈变为 A-B-C-D-D。然而，D 的启动模式为 "singleTop"，D 已存在的实例通过 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 方法接收到这个意图，因为它是在栈的顶部，栈仍然保持为 A-B-C-D。然后，如果一个意图过来请求B类型的活动，则 B 的新实例被添加进堆栈中，即使它的启动模式为 "singleTop"。

> 注意：当活动的一个新实例被创建时，用户可以按返回按钮来返回到前一个活动。但是当一个活动已存在的实例处理一个新意图时，用户不可以按返回键来返回到新意图通过 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 方法到达之前时的状态。

"singleTask"

系统会创建一个新任务并从新任务的根部实例化活动。然而，如果该活动的一个实例已经存在于一个单独的任务，系统会通过调用它的 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 方法发送意图给这个已存在的实例，而不是创建一个新的实例。一个活动在同一时间只能存在一个实例。

> 注意：虽然活动在一个新的任务里启动，但是后退按钮仍然使用户返回到前一个活动。

"singleInstance"

和 "singleTask" 一样，除了系统不会在任务里启动任何其它的活动来持有这个实例。该活动始终是任务里单一且唯一的成员；任何被这个活动启动的活动在一个单独的任务里打开。

再举一个例子，Android 浏览器程序声明网页浏览器的活动应该总是在它自己的任务里打开——通过在 [\<activity>](http://developer.android.com/guide/topics/manifest/activity-element.html) 元素里指定为 **singleTask** 启动模式。这意味着你的应用程序发出一个意图来打开 Android 浏览器时，它的活动不是放置在跟你的应用程序一样的任务里。相反，无论是启动浏览器的新的一个任务还是浏览器有一个运行在后台的任务，那个任务都会被拿到前台来处理这个新的意图。

不管活动是否启动一个新的任务还是启动了它的同一个任务，后退键总是把用户带到前一个活动。然而，如果你启动了一个为 **singleTask** 启动模式的活动，然后如果那个活动在一个后台任务里存在一个实例，那么那整个任务会被带到前台来。此时，回退栈包含所有被拿到前台的活动放在栈顶上。图 4 说明了这种类型的方案。

{% img /images/2014-5-19/4.png 图 4 %}

图4："singleTask"启动模式的活动是如何被添加进回退栈的一个展示。如果活动已经是带有自己回退栈的后台任务的一部分，那么这整个回退栈也会被拿到前台来放在当前任务的顶部。

有关在清单文件中使用启动模式的更多信息，请参阅 [\<activity>](http://developer.android.com/guide/topics/manifest/activity-element.html) 元素的文档，其中有更多关于启动模式属性和可被接受的值的讨论。

> 注：你为活动设置的启动模式属性的行为会被启动活动的带有标志的意图所覆盖，如在下一节讨论的。

#### 使用意图标志

当启动一个活动，你可以通过传递给 [startActivity()](http://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent\)) 一个带有标志的意图来改变活动和它的任务默认的关联行为。你可以使用的改变默认行为的标志如下：

[FLAG_ACTIVITY_NEW_TASK](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK)

启动一个新的任务。如果一个任务已经运行你正在启动的活动，那个任务会被带到前台来并带着上次的状态恢复并且这个活动在 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 方法里接收新的意图。

这将产生和 "singleTask" 启动模式值一样的行为，这在上一节已经讨论过了。

[FLAG_ACTIVITY_SINGLE_TOP](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_SINGLE_TOP)

如果要启动的活动是当前活动(在回退栈的顶部)，那么现有的实例接收 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 方法的调用，而不是创建这个活动的一个新的实例。

这将产生和 "singleTop" 启动模式值一样的行为，这在上一节已经讨论过了。

[FLAG_ACTIVITY_CLEAR_TOP](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TOP)

如果要启动的活动已经在当前任务中运行，然后所有它顶部的其它活动都被销毁并且这个意图被传送来通过 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 方法来恢复这个活动的实例，而不是创建这个活动的一个新的实例。

在[启动模式](http://developer.android.com/guide/topics/manifest/activity-element.html#lmode)属性里没有一个值可以产生这样的的行为。

**FLAG_ACTIVITY_CLEAR_TOP** 是最经常与 **FLAG_ACTIVITY_NEW_TASK** 一起使用的。当一起使用时，这些标志是定位另一个任务中存在的活动和把它放在它所能响应的Intent的某个位置的一种方式。

> 注意：如果指定的活动启动模式为 "standard"，它也能从堆栈中删除并且一个新的实例在它的地方启动来处理传人的意图。这是因为对于一个新的意图当启动模式为 "standard" 时一个新的实例总是被创建。

### 处理亲和性

亲和性指的是一个活动更倾向于属于哪个任务。默认情况下，同一应用程序的所有活动彼此具有亲和性。所以，默认的，同一应用程序的所有活动倾向于在同一个任务中。然而，你可以改变一个活动的默认亲和性。不同应用程序的活动可以共享一个亲和性，或者同一应用程序中的活动可以被赋值为不同任务的亲和性。

你可以用 [\<activity>](http://developer.android.com/guide/topics/manifest/activity-element.html) 元素的 [taskAffinity](http://developer.android.com/guide/topics/manifest/activity-element.html#aff) 属性改变任何已给的活动的亲和性。

[taskAffinity](http://developer.android.com/guide/topics/manifest/activity-element.html#aff) 属性需要一个字符串值，它必须唯一的，来自 <manifest> 元素中声明的默认包名，因为系统使用那个名字来识别应用程序的默认任务亲和性。

亲和性在两种情况下发挥作用：

* 当一个启动活动的意图包括 [FLAG_ACTIVITY_NEW_TASK](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK) 标志时。

一个新的活动，默认情况下，启动进入这个叫做 [startActivity()](http://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent\)) 活动的任务。它被推送到和调用者一样的回退栈里。然而，如果传给 [startActivity()](http://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent\)) 的意图包含 [FLAG_ACTIVITY_NEW_TASK](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK) 标志的话，系统会寻找一个不同的任务来容纳这个新活动。通常情况下，它是一个新的任务，但并不是必须如此。如果有一个已经存在的任务和这个新活动是一样的亲和性，这个活动会启动进入那个任务。如果不一样的话，它开始一个新的任务。

如果这个标志导致一个活动开始一个新的任务并且用户按了 *Home* 按钮离开了它，那么必须有某种方式使用户导航返回到这个任务。一些实体(例如通知管理器)总是在一个外部的任务里启动活动，从来不是作为它们自己的一部分，所以它们总是把 FLAG_ACTIVITY_NEW_TASK 标志放进它们传给 [startActivity()](http://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent\))的意图里。如果你有一个通过一个也许使用了这个标志的外部实体可以调用的活动，注意用户有一种独立的方式来返回那个启动的任务，例如用启动图标(任务的根活动有一个 [CATEGORY_LAUNCHER](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_LAUNCHER) 的意图过滤器；见下面的 [Starting a task](http://developer.android.com/guide/components/tasks-and-back-stack.html#Starting) 章节)。

* 当一个活动它的 [allowTaskReparenting](http://developer.android.com/guide/topics/manifest/activity-element.html#reparent) 属性被设置为 "true"时。

这种情况下，当这个活动有一个亲和性的任务进入前台时，该活动可以从它启动的任务移动到这个任务。

例如，假设一个报告选定城市天气情况的活动被定义为一个旅游应用程序的一部分。它在同样的应用程序里和其它的活动有同样的亲和性(默认的应用亲和性)并且允许它用这个属性重新设置。当你其中的一个活动启动天气报告活动时，它最初和你的活动属于同样的任务。然而，当旅游应用程序的任务进入到前台时，天气报告的活动被重新分配到那个任务并在它里面展示。

> 提示：如果一个apk文件从用户的角度包含不止一个“应用程序”，你可能想使用 [taskAffinity](http://developer.android.com/guide/topics/manifest/activity-element.html#aff) 属性给和每个“应用程序”相关的活动赋予不同的亲和性。

### 清除回退栈

如果用户离开一个任务很长一段时间后，系统将清除这个任务里的所有活动除了根活动。当用户再一次返回这个任务，仅仅只有根活动被恢复。系统这样的行为是因为在一段很长的时间后，用户很可能放弃了他们之前正在做的事情并且返回到这个任务开始一些事情。

有一些活动属性你可以使用来修改这种行为：

[alwaysRetainTaskState](http://developer.android.com/guide/topics/manifest/activity-element.html#always)

如果这个属性在任务的根活动里设置为 "true"，刚刚描述的默认行为就不会发生。任务会保留在它堆栈里的所有活动即使在一段很长的时间之后。

[clearTaskOnLaunch](http://developer.android.com/guide/topics/manifest/activity-element.html#clear)

如果这个属性在任务的根活动里设置为 "true"，堆栈会清除到只剩根活动无论何时用户离开任务和返回任务。换句话说，它正好与 [alwaysRetainTaskState](http://developer.android.com/guide/topics/manifest/activity-element.html#always) 相反。用户总是返回到任务的初始状态，即使是离开任务一会儿。

[finishOnTaskLaunch](http://developer.android.com/guide/topics/manifest/activity-element.html#finish)

这个属性有点像 [clearTaskOnLaunch](http://developer.android.com/guide/topics/manifest/activity-element.html#clear)，但是它运行在一个单一的活动之上，而不是整个任务。它还可能导致任何活动离开，包括根活动。当它被设置为 "true"时，这个活动只会保留当前会话的部分。如果这个用户离开了然后又回到任务，它不再存在。

### 启动任务

你可以设置一个活动作为一个任务的切入点通过给它一个用 "android.intent.action.MAIN" 作为指定动作和 "android.intent.category.LAUNCHER" 作为指定类别的意图过滤器。例如：

{% codeblock lang:xml %}
<activity ... >
    <intent-filter ... >
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    ...
</activity>
{% endcodeblock %}

这样的一个意图过滤器会导致活动的图标和标签在应用程序启动器上展示，给用户一种方式来启动活动和在任务任意时间创建启动之后返回任务。

第二个能力是很重要的：用户必须能够离开一个任务然后通过使用这个活动启动器回来。由于这个原因，这两个启动模式总是在启动任务时标记活动，"singleTask" 和 ""singleInstance" 应该仅当被用于具有 [ACTION_MAIN](http://developer.android.com/reference/android/content/Intent.html#ACTION_MAIN) 和 [CATEGORY_LAUNCHER](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_LAUNCHER) 过滤器的活动。设想一下，例如，如果不要过滤器会发生什么：一个意图启动了一个 "singleTask" 的活动，初始化了一个新的任务，并且用户花了一些时间在该任务上工作。然后用户按了 *Home* 按钮。这个任务被传送到后台并且不可见。现在用户没有办法返回到这个任务了，因为它没有在应用启动器里展示。

对于这些你不想用户能够返回到一个活动的情况，将 [\<activity>](http://developer.android.com/guide/topics/manifest/activity-element.html) 元素的 [finishOnTaskLaunch](http://developer.android.com/guide/topics/manifest/activity-element.html#finish) 设置为 "true"(参见 [Clearing the stack](http://developer.android.com/guide/components/tasks-and-back-stack.html#Clearing))。


























