---
layout: post
title: Android的消息机制
category: blog
tags: 消息机制
---



&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Android的消息机制是指`Handler`的运行机制，Handler是我们经常需要用到的一个东西，所以熟练掌握这个知识点非常有必要。一般我们用Handler来更新UI界面，比如我们有一些耗时的操作需要在子线程中处理，如下载、请求网络数据等，当这些耗时的操作完成后，可能会需要在UI做一些改变，比如请求完数据后需要将结果数据显示在页面相应的控件上，比如下载过程中需要在UI界面显示下载进度等，但是Android系统规定，在子线程中是不能更新UI控件的，否则会出现异常。系统不允许在子线程中访问UI是因为Android的UI控件并不是线程安全的，如果在多线程并发访问可能会导致UI控件的状态不可控。那么当子线程中需要更新UI时，Android的消息机制是怎么把更新UI的操作切换到主线程中的呢？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;虽然关于Android消息机制在实际开发过程中我们开发者一般只需要使用到Handler，但是我们有必要清楚在实际开发过程中，Handler是和`MessageQueue`和`Looper`一起协同工作的。MessageQueue即消息队列，它是用来存储消息的，以队列的形式对消息进行存取操作。Looper即轮询器，用来轮询消息队列中是否有消息存在，MessageQueue是接收到消息后将其存储起来，但它不能去处理消息，而Looper则是无限循环的去MessageQueue查看是否有新消息，有就去处理消息，没有就处于等待状态。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Handler在创建其对象的时候会采用当前线程的Looper来构造消息循环系统，但是它是怎么获取到当前线程的Looper的呢？这里它用到了`ThreadLocal`，ThreadLocal可以在不同的线程互不干扰的存储并提供数据，通过ThreadLocal就可以拿到每个线程的Looper。当然一个线程默认是没有Looper的，如果需要在一个线程中使用Handler就必须为线程创建Looper，而UI线程也就是`ActivityThread`在被创建时就会初始化Looper，所以在UI线程中，默认就可以使用Handler。如果当前线程中没有Looper，就会出现下面的异常：

![输入图片说明](https://static.oschina.net/uploads/img/201705/06222540_fy4q.png "在这里输入图片标题")

解决办法就是为当前线程创建一个Looper，错误提示也告诉我们只需要调用

```java
Looper.prepare();
```

就可以为当前线程创建一个Looper，然后调用

```java
Looper.loop();
```

来开启消息轮询，这样在子线程中就同样可以使用Handler了，比如这样：

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        Looper.prepare();
        new Handler().post(new Runnable() {
            @Override
            public void run() {

            }
        });
        Looper.loop();
    }
}).start();
```


## ThreadLocal的工作原理

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面提到了Handler在创建的时候会获取到当前线程的Looper来构造消息循环系统，而获取Looper时用到了ThreadLocal，ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储后只有在指定的线程中才可以获取到存储的数据，对于其他线程则无法获取到。当某些数据以线程为作用域并且不同线程具有不同的数据副本时就可以考虑使用ThreadLocal，比如Handler获取当前线程的Looper，Looper的作用域就只是当前线程且不同的线程有不同的Looper，所以这时使用ThreadLocal就可以很容易的存取Looper。下面通过代码来看看：

```java
private ThreadLocal<String> mThreadLocal = new ThreadLocal<>();
```

```java
// UI线程中赋值
mThreadLocal.set("shenhuniurou");

// UI线程中取值
Log.d(TAG, "UI线程中mThreadLocal=" + mThreadLocal.get());

new Thread("Thread1") {

    @Override
    public void run() {
        // Thread1线程中赋值
        mThreadLocal.set("shenhuniurou1");

        // Thread1线程中取值
        Log.d(TAG, "UThread1线程中mThreadLocal=" + mThreadLocal.get());
    }
}.start();

new Thread("Thread2") {

    @Override
    public void run() {
        // Thread2线程中赋值
        mThreadLocal.set("shenhuniurou2");

        // Thread2线程中取值
        Log.d(TAG, "UThread1线程中mThreadLocal=" + mThreadLocal.get());
    }
}.start();
```

日志输出如图：

![输入图片说明](https://static.oschina.net/uploads/img/201705/06223155_ZY7Y.png "在这里输入图片标题")

上面的代码我们发现，在不同的线程访问同一个ThreadLocal对象，获取到的值却不一样。这是因为不同线程访问同一个ThreadLocal的get方法，ThreadLocal内部会从各自的线程中取一个数组，然后再从数组中根据当前ThreadLocal的索引去查找出对应的value值，所以不同线程的数组是不同的。

![输入图片说明](https://static.oschina.net/uploads/img/201705/02233218_qBCs.png "在这里输入图片标题")

## Handler的工作原理

Handler在消息机制中的工作主要包括消息的发送和接收，发送消息有两种方式，一种是采用post方式：

```java
handler.post(Runnable) 
or 
handler.postDelayed(Runnable)
```

另一种采用sendMessage方式：

```java
handler.sendMessage(Message)
```

实际上post的方式最终也是通过sendMessage的方式完成的，我们可以看看post方式的源码：

```java
public final boolean post(Runnable r) {
   return  sendMessageDelayed(getPostMessage(r), 0);
}

private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
```

接下来我们看看Handler类中sendMessage方式的工作过程。我们发现所有sendMessage方法最后都会调用到下面这个方法去：

```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
```

而Handler中持有消息队列的引用，在enqueueMessage方法中将消息添加到队列中：

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

上面这段代码在调用MessageQueue的enqueueMessage方法前，为消息msg设置一个target，即当前的Handler类，这个后面会有用。我们可以到MessageQueue这个类中去看看enqueueMessage这个方法：

```java
boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }

    synchronized (this) {
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```

从这段代码中我们可以发现，消息队列虽然名为队列，可它内部的数据结构实际上是一个单链表。消息中的Runnable其实就是一个回调函数，当Looper处理完消息后，消息中的Runnable或者Handler的`handleMessage`方法就会被调用，而Looper是运行在创建Handler所在的线程中，这样Runnable或者handleMessage中的事务处理就被切换到创建Handler的线程了。

我们可以看到Handler发送消息的过程实际上仅仅是往消息队列中添加了一条消息。而Looper一直在轮询查看是否有新消息：

```java
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        final Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        final long traceTag = me.mTraceTag;
        if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
            Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
        }
        try {
            msg.target.dispatchMessage(msg);
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked();
    }
}
```

可以看到，Looper取消息是调用了队列的next方法，然后调用

```java
msg.target.dispatchMessage(msg);
```

前面说过这个消息的target就是Handler，所以是调用Handler的dispatchMessage方法：

```java
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
```

msg的callback其实就是post发送消息时传递的那个Runnable，如果有发送消息时有Runnable，就调用Runnable的run方法：

```java
private static void handleCallback(Message message) {
    message.callback.run();
}
```

若没有就检查mCallback是否为空，不为空就调用mCallback的handleMessage方法。mCallback是一个接口，当我们不想派生Handler的子类时可以采用实现Callback方式来实现。使用方法是当前类实现Handler.Callback，然后重写handleMessage方法。

```java
public class MainActivity extends AppCompatActivity implements Handler.Callback {

    @Override
    public boolean handleMessage(Message msg) {
        return false;
    }

}
```

最后如果mCallback为空，就调用Handler的handleMessage方法：

```java
private Handler handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
    }
};
```




## MessageQueue的工作原理

消息队列的主要工作包括消息的添加和读取，分别使用的`enqueueMesaage`和`next`方法，上面已经说过enqueueMessage方法了，就是把一条消息插入到单链表中，这里我们看一下next方法：

```java
Message next() {
    // Return here if the message loop has already quit and been disposed.
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    final long ptr = mPtr;
    if (ptr == 0) {
        return null;
    }

    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }

        nativePollOnce(ptr, nextPollTimeoutMillis);

        synchronized (this) {
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    // Next message is not ready.  Set a timeout to wake up when it is ready.
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    msg.markInUse();
                    return msg;
                }
            } else {
                // No more messages.
                nextPollTimeoutMillis = -1;
            }

        }
    }
}
```

可以看到，next方法和loop方法一样，是一个无线循环的方法，如果消息队列中没有消息，next方法会一直阻塞在这里，当有新消息来时，next方法会返回该消息并将其从单链表中移除。


## Looper的工作原理

Looper轮询器的工作就是轮询消息，有消息就处理，否则就阻塞，它的构造方法如下：

```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```

首先它会创建一个MessageQueue，然后将当前线程保存起来。前面也有提到如何创建一个Looper，即调用Looper的prepare方法来为当前线程创建Looper：

```java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

如果当前线程的ThreadLocal已经存在了Looper，调用prepare时则会抛出异常，每个线程只能创建一个Looper。创建完之后又将Looper保存到了ThreadLocal中。除了prepare方法外，Looper还提供了prepareMainLooper方法，该方法主要是给主线程也就是ActivityThread创建Looper使用的，但其本质还是通过prepare方法实现的：

```java
public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
```

Looper也是可以退出的，Looper提供了`quit`和`quitSafely`方法来退出Looper，二者最终都是调用队列的quit方法，只不过参数不同：

```java
mQueue.quit(flag);
```

```java
void quit(boolean safe) {
    if (!mQuitAllowed) {
        throw new IllegalStateException("Main thread not allowed to quit.");
    }

    synchronized (this) {
        if (mQuitting) {
            return;
        }
        mQuitting = true;

        if (safe) {
            removeAllFutureMessagesLocked();
        } else {
            removeAllMessagesLocked();
        }

        // We can assume mPtr != 0 because mQuitting was previously false.
        nativeWake(mPtr);
    }
}
```

quit是直接退出Looper，而quitSafely只是设定一个退出标记，然后把消息队列中的已有消息处理完毕后才安全退出。其实也就是下面这两个方法的区别：

```java
private void removeAllMessagesLocked() {
    Message p = mMessages;
    while (p != null) {
        Message n = p.next;
        p.recycleUnchecked();
        p = n;
    }
    mMessages = null;
}

private void removeAllFutureMessagesLocked() {
    final long now = SystemClock.uptimeMillis();
    Message p = mMessages;
    if (p != null) {
        if (p.when > now) {
            removeAllMessagesLocked();
        } else {
            Message n;
            for (;;) {
                n = p.next;
                if (n == null) {
                    return;
                }
                if (n.when > now) {
                    break;
                }
                p = n;
            }
            p.next = null;
            do {
                p = n;
                n = p.next;
                p.recycleUnchecked();
            } while (n != null);
        }
    }
}
```

Looper退出后，通过Handler发送消息会失败，在子线程中如果手动创建Looper，在事情处理完之后要调用quit方法来终止消息循环，否则该子线程会一直处于等待状态，如果退出Looper以后，该线程就会终止。

Looper开启消息循环的方法是loop，上面也提到过，它是无限循环的，能停止循环的就是消息队列的next方法返回null，当Looper调用了quit方法时，Looper会调用消息队列的quit或者quitSafely方法来通知消息队列退出，当消息队列被标记为退出状态时，它的next方法就会返回null，退出标记就是这个变量`mQuitting`，我们看到在队列的quit方法出，它被标记为true了。

到这里Handler、MessageQueue、Looper的工作原理都理清了，下面用一张图说明Android消息机制的工作原理：

![输入图片说明](https://static.oschina.net/uploads/img/201705/06213809_eVsZ.png "在这里输入图片标题")

相信看到这里，你已经对第一段中我们抛出的问题“Android的消息机制是怎么把更新UI的操作切换到主线程中的呢？”有答案了吧。




