---
layout: post
title: "onInterceptTouchEvent和onTouchEvent的调用时序"
date: 2014-11-10 23:40:42 +0800
comments: true
categories: [Android]
---
大多数刚接触Android开发的同学可能会对`onInterceptTouchEvent`和`onTouchEvent`这两个方法的调用时序搞不明白，我刚开始也是，只是知其然，但不知其所以然。其实很多事情最重要的是亲手去做，一旦自己亲手去做了，很多事情就会豁然开朗。下面我们通过代码和Log将这两个方法的调用时序给大家展示明白。

`onInterceptTouchEvent`是ViewGroup中的方法，从字面上理解很简单，就是拦截Touch事件，返回**true**表示拦截，返回**false**表示不拦截。

`onTouchEvent`是View中的方法，它也返回一个布尔值，**true**表示对这个Touch事件进行处理，消耗（consume）了这个事件，**false**表示对这个Touch事件不进行处理，没有消耗这个事件，然后将这个事件返回给它的父亲处理。

首先，我们一定要对Android的触控系统（Touch System）充分理解，当我们手指按在手机屏幕上的时候，首先是当前的Activity接收到这个事件，然后通过`dispatchTouchEvent`方法将事件分发给当前的布局（layout），最先接收到事件的是布局最外层的ViewGroup，也就是身为父亲的ViewGroup，然后它再通过自己的`dispatchTouchEvent`方法将这个事件传递给它的孩子（View或ViewGroup），如果此时这个父亲在它的`onInterceptTouchEvent`方法中返回**true**表示它将这个事件拦截了，以后的所有事件都不传递给自己的孩子了，在自己的`onTouchEvent`中返回**true**来处理这个事件，返回**false**交给它的父亲来处理这个事件。反之，以后的事件它都传递给自己的孩子，让自己的孩子来处理这个事件,同时它也监听着。一旦拦截了就将以后所有的事件都不会传递给自己的孩子了，这个过程是不可逆的，直到下一个周期到来（接收到下一个**Down**事件）。Android的触控系统是以接收到一个**Down**事件到接收到一个**Up**或**Cancel**事件为一个周期的。Android的触控系统就是这样将事件层层传递和层层返回的。

<!--more-->

好了，下面我们来看具体的例子，Demo的布局很简单，就是一个自定义的LinearLayout嵌套了一个自定义的View，对应的XML代码如下：
{% codeblock lang:xml %}
<com.jeffreylee.androidtest.CustomLinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/parent"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.jeffreylee.androidtest.CustomView
        android:id="@+id/child"
        android:layout_width="300dp"
        android:layout_height="300dp"
        android:layout_gravity="center"
        android:background="@android:color/holo_red_dark" />
</com.jeffreylee.androidtest.CustomLinearLayout>
{% endcodeblock %}

###父亲CustomLinearLayout`onInterceptTouchEvent`中返回**true**拦截事件

CustomLinearLayout.java
{% codeblock lang:java %}
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.i(TAG, "ParentView has received ACTION_DOWN in onInterceptTouchEvent");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i(TAG, "ParentView has received ACTION_MOVE in onInterceptTouchEvent");
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                Log.i(TAG, "ParentView has received ACTION_UP in onInterceptTouchEvent");
                break;
            default:
                break;
        }
        return true;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.i(TAG, "ParentView has received ACTION_DOWN in onTouchEvent");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i(TAG, "ParentView has received ACTION_MOVE in onTouchEvent");
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                Log.i(TAG, "ParentView has received ACTION_UP in onTouchEvent");
                break;
            default:
                break;
        }
        return true;
    }
{% endcodeblock %}

CustomView.java
{% codeblock lang:java %}
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.i(TAG, "ChildView has received ACTION_DOWN in onTouchEvent");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i(TAG, "ChildView has received ACTION_MOVE in onTouchEvent");
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                Log.i(TAG, "ChildView has received ACTION_UP in onTouchEvent");
                break;
            default:
                break;
        }
        return true;
    }
{% endcodeblock %}
Log如下：
>
I/AndroidTouchSystem(24162): ParentView has received ACTION_DOWN in onInterceptTouchEvent
I/AndroidTouchSystem(24162): ParentView has received ACTION_DOWN in onTouchEvent
I/AndroidTouchSystem(24162): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(24162): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(24162): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(24162): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(24162): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(24162): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(24162): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(24162): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(24162): ParentView has received ACTION_UP in onTouchEvent

###父亲CustomLinearLayout`onInterceptTouchEvent`中返回**false**不拦截事件，孩子CustomView`onTouchEvent`中返回false不处理事件

CustomLinearLayout.java
{% codeblock lang:java %}
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.i(TAG, "ParentView has received ACTION_DOWN in onInterceptTouchEvent");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i(TAG, "ParentView has received ACTION_MOVE in onInterceptTouchEvent");
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                Log.i(TAG, "ParentView has received ACTION_UP in onInterceptTouchEvent");
                break;
            default:
                break;
        }
        return false;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.i(TAG, "ParentView has received ACTION_DOWN in onTouchEvent");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i(TAG, "ParentView has received ACTION_MOVE in onTouchEvent");
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                Log.i(TAG, "ParentView has received ACTION_UP in onTouchEvent");
                break;
            default:
                break;
        }
        return true;
    }
{% endcodeblock %}

CustomView.java
{% codeblock lang:java %}
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.i(TAG, "ChildView has received ACTION_DOWN in onTouchEvent");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i(TAG, "ChildView has received ACTION_MOVE in onTouchEvent");
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                Log.i(TAG, "ChildView has received ACTION_UP in onTouchEvent");
                break;
            default:
                break;
        }
        return false;
    }
{% endcodeblock %}
Log如下：
>
I/AndroidTouchSystem(25705): ParentView has received ACTION_DOWN in onInterceptTouchEvent
I/AndroidTouchSystem(25705): ChildView has received ACTION_DOWN in onTouchEvent
I/AndroidTouchSystem(25705): ParentView has received ACTION_DOWN in onTouchEvent
I/AndroidTouchSystem(25705): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(25705): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(25705): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(25705): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(25705): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(25705): ParentView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(25705): ParentView has received ACTION_UP in onTouchEvent

###父亲CustomLinearLayout`onInterceptTouchEvent`中返回**false**不拦截事件，孩子CustomView`onTouchEvent`中返回true消耗事件

CustomLinearLayout.java
{% codeblock lang:java %}
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.i(TAG, "ParentView has received ACTION_DOWN in onInterceptTouchEvent");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i(TAG, "ParentView has received ACTION_MOVE in onInterceptTouchEvent");
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                Log.i(TAG, "ParentView has received ACTION_UP in onInterceptTouchEvent");
                break;
            default:
                break;
        }
        return false;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.i(TAG, "ParentView has received ACTION_DOWN in onTouchEvent");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i(TAG, "ParentView has received ACTION_MOVE in onTouchEvent");
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                Log.i(TAG, "ParentView has received ACTION_UP in onTouchEvent");
                break;
            default:
                break;
        }
        return true;
    }
{% endcodeblock %}

CustomView.java
{% codeblock lang:java %}
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.i(TAG, "ChildView has received ACTION_DOWN in onTouchEvent");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i(TAG, "ChildView has received ACTION_MOVE in onTouchEvent");
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                Log.i(TAG, "ChildView has received ACTION_UP in onTouchEvent");
                break;
            default:
                break;
        }
        return true;
    }
{% endcodeblock %}
Log如下：
>
I/AndroidTouchSystem(26236): ParentView has received ACTION_DOWN in onInterceptTouchEvent
I/AndroidTouchSystem(26236): ChildView has received ACTION_DOWN in onTouchEvent
I/AndroidTouchSystem(26236): ParentView has received ACTION_MOVE in onInterceptTouchEvent
I/AndroidTouchSystem(26236): ChildView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(26236): ParentView has received ACTION_MOVE in onInterceptTouchEvent
I/AndroidTouchSystem(26236): ChildView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(26236): ParentView has received ACTION_MOVE in onInterceptTouchEvent
I/AndroidTouchSystem(26236): ChildView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(26236): ParentView has received ACTION_MOVE in onInterceptTouchEvent
I/AndroidTouchSystem(26236): ChildView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(26236): ParentView has received ACTION_MOVE in onInterceptTouchEvent
I/AndroidTouchSystem(26236): ParentView has received ACTION_MOVE in onInterceptTouchEvent
I/AndroidTouchSystem(26236): ChildView has received ACTION_MOVE in onTouchEvent
I/AndroidTouchSystem(26236): ParentView has received ACTION_UP in onInterceptTouchEvent
I/AndroidTouchSystem(26236): ChildView has received ACTION_UP in onTouchEvent

同学们看完代码和Log后对`onInterceptTouchEvent`和`onTouchEvent`的调用时序应该就会很明白了吧。