---
layout: post
keywords: android, handler, message, messagequeue, looper
description: study android message mechanism
title: "Android消息机制"
categories: [Android]
tags: [android, handler, message, messagequeue, looper]
group: Android
icon: file-alt
date: 2014-05-01 00:00:00
---

我们都知道，Android UI是线程不安全的，如果在子线程中尝试进行UI操作，程序就有可能会崩溃。相信大家在日常的工作当中都会经常遇到这个问题，解决的方案应该也是早已烂熟于心，即创建一个Message对象，然后借助Handler发送出去，之后在Handler的handleMessage()方法中获得刚才发送的Message对象，然后在这里进行UI操作就不会再出现崩溃了。

<!--excerpt-->

***

#创建Handler对象

首先来看一下如何创建Handler对象。Handler提供了4个构造函数：

    Handler()；
    Handler(Callback callback)；
    Handler(Looper looper)；
    Handler(Looper looper, Callback callback)；

这4个构造函数分别对应了是否传入Looper实例和是否传入Callback实例。其中前两个函数不传入Looper实例，则此Handler绑定当前线程的Looper对象实例，如Looper实例为空则抛出异常：

    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }

如下代码，创建了两个Handler，其中一个在UI主线程中创建，一个在匿名子线程中创建：

    public class MainActivity extends Activity {  

        private Handler handler1;
        private Handler handler2;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            handler1 = new Handler();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    handler2 = new Handler();
                }
            }).start();
        }
    }

如果现在运行一下程序，你会发现，在子线程中创建的Handler是会导致程序崩溃的，提示的错误信息为 Can't create handler inside thread that has not called Looper.prepare() 。说是不能在没有调用Looper.prepare() 的线程中创建Handler。从Handler构造函数源码可以看出来当Looper.myLooper()返回为空时就会抛出这个异常，先看看Looper.myLooper()函数：

    public static final Looper myLooper() {
        return (Looper)sThreadLocal.get();
    }

这里可看出myLooper()函数会从sThreadLocal中获取一个Looper的实例，这个实例为空会导致程序崩溃。那我们尝试在子线程中先调用一下Looper.prepare()呢，代码如下所示：

    new Thread(new Runnable() {  
        @Override  
        public void run() {  
            Looper.prepare();  
            handler2 = new Handler();  
        }  
    }).start();

果然这样就不会崩溃了，那么Looper.prepare()中做了什么了？

    public static final void prepare() {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper());
    }

可以看出，prepare()函数里new了一个Looper对象的实例set到sThreadLocal中，这样再调用myLooper()就会返回这个Looper实例了。Looper构造函数为：

    private Looper() {
        mQueue = new MessageQueue();
        mRun = true;
        mThread = Thread.currentThread();
    }

可以看出这里Looper是绑定当前线程的，这样Handler再不传入Looper实例的情况下，也是绑定Handler所在的线程的。因此当前线程在创建Handler前必需先调用Looper.prepare()方法，而且每个线程只能调用一次，否则就会报"Only one Looper may be created per thread"错误。

那么为什么我们再UI主线程中创建的Handler没有调用Looper.prepare()方法，却没有报错呢？这是由于在程序启动的时候，系统已经帮我们自动在主线程里调用了Looper.prepare()方法。查看ActivityThread中的main()方法，代码如下所示：

    public static void main(String[] args) {
        SamplingProfilerIntegration.start();  
        CloseGuard.setEnabled(false);  
        Environment.initForCurrentUser();  
        EventLogger.setReporter(new EventLoggingReporter());  
        Process.setArgV0("<pre-initialized>");  
        Looper.prepareMainLooper();  
        ActivityThread thread = new ActivityThread();  
        thread.attach(false);  
        if (sMainThreadHandler == null) {  
            sMainThreadHandler = thread.getHandler();  
        }  
        AsyncTask.init();  
        if (false) {  
            Looper.myLooper().setMessageLogging(new LogPrinter(Log.DEBUG, "ActivityThread"));  
        }  
        Looper.loop();  
        throw new RuntimeException("Main thread loop unexpectedly exited");  
    }

可以看到，在第7行调用了Looper.prepareMainLooper()方法，而这个方法又会再去调用Looper.prepare()方法，代码如下所示：

    public static final void prepareMainLooper() {  
        prepare();  
        setMainLooper(myLooper());  
        if (Process.supportsProcesses()) {  
            myLooper().mQueue.mQuitAllowed = false;  
        }  
    }

因此我们应用程序的主线程中会始终存在一个Looper对象，从而不需要再手动去调用Looper.prepare()方法了。

这样基本就将Handler的创建过程完全搞明白了，总结一下就是在主线程中可以直接创建Handler对象，而在子线程中需要先调用Looper.prepare()才能创建Handler对象。

***

#发送消息

看完了如何创建Handler之后，接下来我们看一下如何发送消息，这个流程相信大家也已经非常熟悉了，new出一个Message对象，然后可以使用setData()方法或arg参数等方式为消息携带一些数据，再借助Handler将消息发送出去就可以了，示例代码如下：

    new Thread(new Runnable() {  
        @Override  
        public void run() {  
            Message message = new Message();  
            message.arg1 = 1;  
            Bundle bundle = new Bundle();  
            bundle.putString("data", "data");  
            message.setData(bundle);  
            handler.sendMessage(message);  
        }  
    }).start();

可是这里Handler到底是把Message发送到哪里去了呢？为什么之后又可以在Handler的handleMessage()方法中重新得到这条Message呢？看来又需要通过阅读源码才能解除我们心中的疑惑了，Handler中提供了很多个发送消息的方法，其中除了sendMessageAtFrontOfQueue()方法之外，其它的发送消息方法最终都会辗转调用到sendMessageAtTime()方法中，这个方法的源码如下所示：

    public boolean sendMessageAtTime(Message msg, long uptimeMillis)  
    {  
        boolean sent = false;  
        MessageQueue queue = mQueue;  
        if (queue != null) {  
            msg.target = this;  
            sent = queue.enqueueMessage(msg, uptimeMillis);  
        }  
        else {  
            RuntimeException e = new RuntimeException(  
                this + " sendMessageAtTime() called with no mQueue");  
            Log.w("Looper", e.getMessage(), e);  
        }  
        return sent;  
    }

sendMessageAtTime()方法接收两个参数，其中msg参数就是我们发送的Message对象，而uptimeMillis参数则表示发送消息的时间，它的值等于自系统开机到当前时间的毫秒数再加上延迟时间，如果你调用的不是sendMessageDelayed()方法，延迟时间就为0，然后将这两个参数都传递到MessageQueue的enqueueMessage()方法中。这个MessageQueue又是什么东西呢？其实从名字上就可以看出了，它是一个消息队列，用于将所有收到的消息以队列的形式进行排列，并提供入队和出队的方法。这个类是在Looper的构造函数中创建的，因此一个Looper也就对应了一个MessageQueue。
那么enqueueMessage()方法毫无疑问就是入队的方法了，我们来看下这个方法的源码：

    final boolean enqueueMessage(Message msg, long when) {  
        if (msg.when != 0) {  
            throw new AndroidRuntimeException(msg + " This message is already in use.");  
        }  
        if (msg.target == null && !mQuitAllowed) {  
            throw new RuntimeException("Main thread not allowed to quit");  
        }  
        synchronized (this) {  
            if (mQuiting) {  
                RuntimeException e = new RuntimeException(msg.target + " sending message to a Handler on a dead thread");  
                Log.w("MessageQueue", e.getMessage(), e);  
                return false;  
            } else if (msg.target == null) {  
                mQuiting = true;  
            }  
            msg.when = when;  
            Message p = mMessages;  
            if (p == null || when == 0 || when < p.when) {  
                msg.next = p;  
                mMessages = msg;  
                this.notify();  
            } else {  
                Message prev = null;  
                while (p != null && p.when <= when) {  
                    prev = p;  
                    p = p.next;  
                }  
                msg.next = prev.next;  
                prev.next = msg;  
                this.notify();  
            }  
        }  
        return true;  
    }

首先你要知道，MessageQueue并没有使用一个集合把所有的消息都保存起来，它只使用了一个mMessages对象表示当前待处理的消息。然后观察上面的代码的16~31行我们就可以看出，所谓的入队其实就是将所有的消息按时间来进行排序，这个时间当然就是我们刚才介绍的uptimeMillis参数。具体的操作方法就根据时间的顺序调用msg.next，从而为每一个消息指定它的下一个消息是什么。当然如果你是通过sendMessageAtFrontOfQueue()方法来发送消息的，它也会调用enqueueMessage()来让消息入队，只不过时间为0，这时会把mMessages赋值为新入队的这条消息，然后将这条消息的next指定为刚才的mMessages，这样也就完成了添加消息到队列头部的操作。

现在入队操作我们就已经看明白了，那出队操作是在哪里进行的呢?这个就需要看一看Looper.loop()方法的源码了，如下所示：

    public static final void loop() {  
        Looper me = myLooper();  
        MessageQueue queue = me.mQueue;  
        while (true) {  
            Message msg = queue.next(); // might block  
            if (msg != null) {  
                if (msg.target == null) {  
                    return;  
                }  
                if (me.mLogging!= null) me.mLogging.println(  
                        ">>>>> Dispatching to " + msg.target + " "  
                        + msg.callback + ": " + msg.what  
                        );  
                msg.target.dispatchMessage(msg);  
                if (me.mLogging!= null) me.mLogging.println(  
                        "<<<<< Finished to    " + msg.target + " "  
                        + msg.callback);  
                msg.recycle();  
            }  
        }  
    }

可以看到，这个方法从第4行开始，进入了一个死循环，然后不断地调用的MessageQueue的next()方法，我想你已经猜到了，这个next()方法就是消息队列的出队方法。不过由于这个方法的代码稍微有点长，我就不贴出来了，它的简单逻辑就是如果当前MessageQueue中存在mMessages(即待处理消息)，就将这个消息出队，然后让下一条消息成为mMessages，否则就进入一个阻塞状态，一直等到有新的消息入队。继续看loop()方法的第14行，每当有一个消息出队，就将它传递到msg.target的dispatchMessage()方法中，那这里msg.target又是什么呢？其实就是Handler啦，你观察一下上面sendMessageAtTime()方法的第6行就可以看出来了。接下来当然就要看一看Handler中dispatchMessage()方法的源码了，如下所示：

    public void dispatchMessage(Message msg) {  
        if (msg.callback != null) {  
            handleCallback(msg);  
        } else {  
            if (mCallback != null) {  
                if (mCallback.handleMessage(msg)) {  
                    return;  
                }  
            }  
            handleMessage(msg);  
        }  
    }

在第5行进行判断，如果mCallback不为空，则调用mCallback的handleMessage()方法，否则直接调用Handler的handleMessage()方法，并将消息对象作为参数传递过去。这样我相信大家就都明白了为什么handleMessage()方法中可以获取到之前发送的消息了吧！

因此，一个最标准的异步消息处理线程的写法应该是这样：

    class LooperThread extends Thread {  
      public Handler mHandler;  
    
      public void run() {  
          Looper.prepare();  
    
          mHandler = new Handler() {  
              public void handleMessage(Message msg) {  
                  // process incoming messages here  
              }  
          };  
    
          Looper.loop();  
      }  
    }

那么我们还是要来继续分析一下，为什么使用异步消息处理的方式就可以对UI进行操作了呢？这是由于Handler总是依附于创建时所在的线程，比如我们的Handler是在主线程中创建的，而在子线程中又无法直接对UI进行操作，于是我们就通过一系列的发送消息、入队、出队等环节，最后调用到了Handler的handleMessage()方法中，这时的handleMessage()方法已经是在主线程中运行的，因而我们当然可以在这里进行UI操作了。

那么在子线程中创建的Handler能不能直接对UI进行操作呢？也是可以的，只要通过构造函数将UI主线程的Looper对象传入Handler即可：

    class Looper2Thread extends Thread {  
      public Handler mHandler;  

      public void run() {
          mHandler = new Handler(Looper.getMainLooper()) {  
              public void handleMessage(Message msg) {  
                  // process incoming messages here  
              }  
          };  
      }  
    }

另外除了发送消息之外，我们还有以下几种方法可以在子线程中进行UI操作：

    1. Handler的post()方法
    
    2. View的post()方法
    
    3. Activity的runOnUiThread()方法

我们先来看下Handler中的post()方法，代码如下所示：

    public final boolean post(Runnable r)  
    {
       return  sendMessageDelayed(getPostMessage(r), 0);  
    }

原来这里还是调用了sendMessageDelayed()方法去发送一条消息啊，并且还使用了getPostMessage()方法将Runnable对象转换成了一条消息，我们来看下这个方法的源码：

    private final Message getPostMessage(Runnable r) {  
        Message m = Message.obtain();  
        m.callback = r;  
        return m;  
    }

在这个方法中将消息的callback字段的值指定为传入的Runnable对象。咦？这个callback字段看起来有些眼熟啊，喔！在Handler的dispatchMessage()方法中原来有做一个检查，如果Message的callback等于null才会去调用handleMessage()方法，否则就调用handleCallback()方法。那我们快来看下handleCallback()方法中的代码吧：

    private final void handleCallback(Message message) {  
        message.callback.run();  
    }

也太简单了！竟然就是直接调用了一开始传入的Runnable对象的run()方法。因此在子线程中通过Handler的post()方法进行UI操作就可以这么写：

    public class MainActivity extends Activity {  
      
        private Handler handler;  
      
        @Override  
        protected void onCreate(Bundle savedInstanceState) {  
            super.onCreate(savedInstanceState);  
            setContentView(R.layout.activity_main);  
            handler = new Handler();  
            new Thread(new Runnable() {  
                @Override  
                public void run() {  
                    handler.post(new Runnable() {  
                        @Override  
                        public void run() {  
                            // 在这里进行UI操作  
                        }  
                    });  
                }  
            }).start();  
        }  
    }

虽然写法上相差很多，但是原理是完全一样的，我们在Runnable对象的run()方法里更新UI，效果完全等同于在handleMessage()方法中更新UI。

然后再来看一下View中的post()方法，代码如下所示：

    public boolean post(Runnable action) {  
        Handler handler;  
        if (mAttachInfo != null) {  
            handler = mAttachInfo.mHandler;  
        } else {  
            ViewRoot.getRunQueue().post(action);  
            return true;  
        }  
        return handler.post(action);  
    }

原来就是调用了Handler中的post()方法，我相信已经没有什么必要再做解释了。
最后再来看一下Activity中的runOnUiThread()方法，代码如下所示：

    public final void runOnUiThread(Runnable action) {  
        if (Thread.currentThread() != mUiThread) {  
            mHandler.post(action);  
        } else {  
            action.run();  
        }  
    }

如果当前的线程不等于UI线程(主线程)，就去调用Handler的post()方法，否则就直接调用Runnable对象的run()方法。

***

#参考:

[Android异步消息处理机制完全解析，带你从源码的角度彻底理解](http://blog.csdn.net/guolin_blog/article/details/9991569)