---
layout: post
title: Android中的线程和线程池
category: blog
tags: 线程和线程池
---



## Android中的线程

线程，在Android中是非常重要的，主线程处理UI界面，子线程处理耗时操作。如果在主线程中处理耗时操作就会发生ANR，这对一个程序来说是非常致命的，因此耗时操作必须放在子线程中去执行。

在Android系统中，除了Thread外，还有很多AsyncTask、IntentService可以扮演线程角色，另外，HandlerThread也是一种特殊的的线程，尽管它们的表现形式不同于传统线程Thread，但它们的本质依然是传统的线程。AsyncTask的底层用到了线程池，IntentService和HandlerThread底层则是直接使用了线程。

AsyncTask封装了线程池和Handler，它主要是为了方便开发者在子线程中更新UI，实际上还是通过Handler将更新UI的操作从子线程切换到主线程来的。HandlerThread是一种具有消息循环的线程，在它的内部可以使用Handler。IntentService是一个服务，系统对其进行了封装，使其可以更方便的执行后台任务， IntentService内部采用HandlerThread来执行任务，当任务执行完后IntentService会自动退出。从任务执行的角度看，IntentService的作用像是一个后台线程，但IntentService是一种服务，它不容易被系统杀死从而可以尽量保证任务的执行，而如果是一个后台线程，由于这个时候进程中没有活动的四大组件，那么这个进程的优先级就会非常低，会很容易被系统杀死，这就是IntentService的优点。

线程的创建和销毁都需要开销，在系统中我们不能频繁的创建线程，如果我们需要大量的线程时，正确的做法是采用线程池，一个线程池中会缓存一定数量的线程，通过线程池就可以避免因为频繁创建和销毁线程所带来的系统开销。

## 主线程和子线程

主线程是指进程所拥有的线程，默认一个进程只有一个线程，就是主线程，主线程主要处理界面交互相关的逻辑，因为用户随时会和UI界面发生交互，所以主线程必须在任何时候都有较高的响应速度，否则就会产生界面卡顿现象。要保持高响应速度，就要求在主线程中不能执行耗时任务，这时子线程就出场了。除了主线程以外的线程都叫子线程。

Android沿用了Java的线程模型，从Android3.0开始系统要求网络访问必须在子线程中进行，否则会访问失败，并抛出NetworkOnMainThreadException，这样做是为了避免主线程由于被耗时操作阻塞从而出现ANR现象。


## AsyncTask

AsyncTask是一种轻量级的异步任务类，它可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程中更新UI。AsyncTask封装了ThreadPool和Handler，通过AsyncTask可以更方便地执行后台任务以及在主线程中访问UI。

AsyncTask是一个抽象的泛型类，它提供了Params、Progress和Result这三个泛型参数，其中Params表示参数类型，Progress表示后台任务执行进度的类型，Result表示后台任务返回的结果类型。如果AsyncTask不需要传递具体的参数，那么这三个泛型参数可以用Void代替。

```java
class MainAsyncTask extends AsyncTask<Void, Void, Void> {


    @Override
    protected void onPreExecute() {
        super.onPreExecute();
    }

    @Override
    protected Void doInBackground(Void... params) {
        return null;
    }

    @Override
    protected void onProgressUpdate(Void... values) {
        super.onProgressUpdate(values);
    }

    @Override
    protected void onPostExecute(Void aVoid) {
        super.onPostExecute(aVoid);
    }

    @Override
    protected void onCancelled() {
        super.onCancelled();
    }
}
```

基本上我们常用的方法就是上面这几个了。

- onPreExecute 在主线程中执行，在异步任务执行之前，此方法被调用，一般可以做一些准备工作。

- doInBackground 在线程池中执行，此方法用于执行异步任务，在此方法中可以通过publishProgress方法来更新任务的执行进度，publishProgress方法会调用onProgressUpdate方法。另外此方法返回任务的执行结果给onPostExecute方法。

- onProgressUpdate 在主线程中执行，当后台任务的执行进度发生改变时此方法会被调用。

- onPostExecute 在主线程中执行，当异步任务执行完后此方法会被调用，它的参数是后台任务doInBackground的返回值。

- onCancelled 在主线程中执行，当异步任务被取消时，此方法被调用，此时onPostExecute方法将不会再被调用。

AsyncTask在具体使用过程中，也有一些条件限制：

- AsyncTask的类必须在主线程中加载，也就是第一次访问AsyncTask必须是在主线程，在Android4.1及以上版本已被系统自动完成。

- AsyncTask的对象必须在主线程中创建。

- execute方法必须在主线程中调用。

- 不能在程序中直接调用onPreExecute、onPostExecute、doInBackground和onProgressUpdate方法。

- 一个AsyncTask对象只能执行一次，也就是只能调用一次execute方法，否则会报运行时异常。

- 在Android1.6之前，AsyncTask是串行执行任务的，Android1.6的时候AsyncTask开始采用线程池来处理并行任务，但是从Android3.0开始，为了避免AsyncTask带来的并发错误，AsyncTask又采用一个线程来串行执行任务。但是在Android3.0及以后的版本中，我们仍然可以通过AsyncTask的executeOnExecutor方法来并行执行任务。


## AsyncTask的工作原理

首先我们从它的execute方法开始分析，代码如下：

```java
@MainThread
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }

   
@MainThread
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
        Params... params) {
    if (mStatus != Status.PENDING) {
        switch (mStatus) {
            case RUNNING:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task is already running.");
            case FINISHED:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task has already been executed "
                        + "(a task can be executed only once)");
        }
    }

    mStatus = Status.RUNNING;

    onPreExecute();

    mWorker.mParams = params;
    exec.execute(mFuture);

    return this;
}
```

看源码可以发现，executeOnExecutor方法的第一个参数sDefaultExecutor其实是一个串行的线程池，一个进程中所有的AsyncTask全都在这个串行的线程池中排队执行，executeOnExecutor方法中AsyncTask的onPreExecute方法最先执行，然后线程池开始执行。上面的exec其实就是sDefaultExecutor。


线程池的执行过程：

```java
public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
private static InternalHandler sHandler;

private final WorkerRunnable<Params, Result> mWorker;
private final FutureTask<Result> mFuture;

private static class SerialExecutor implements Executor {
    final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
    Runnable mActive;

    public synchronized void execute(final Runnable r) {
        mTasks.offer(new Runnable() {
            public void run() {
                try {
                    r.run();
                } finally {
                    scheduleNext();
                }
            }
        });
        if (mActive == null) {
            scheduleNext();
        }
    }

    protected synchronized void scheduleNext() {
        if ((mActive = mTasks.poll()) != null) {
            THREAD_POOL_EXECUTOR.execute(mActive);
        }
    }
}
```

从SerialExecutor的实现可以分析AsyncTask的排队执行过程。首先系统会把AsyncTask的Params参数封装为FutureTask对象，FutureTask是一个并发类，相当于Runnable，然后把这个FutureTask传递给SerialExecutor的execute方法去处理，execute方法则是把FutureTask对象插入到任务队列mTasks中，上面的offer方法就是把这个对象添加到队列的最后面。如果这时没有正在活动的AsyncTask任务，就会调用scheduleNext方法来执行下一个AsyncTask任务。当一个AsyncTask任务执行完后会继续执行其它任务直到所有的任务都被执行为止，这么看来，AsyncTask默认是串行执行的。

依然是上面的代码，我们发现AsyncTask内部有两个线程池：SerialExecutor和THREAD_POOL_EXECUTOR，一个Handler：InternalHandler，其中SerialExecutor用于任务的排队，THREAD_POOL_EXECUTOR用于真正执行任务，因为在方法scheduleNext中就是使用THREAD_POOL_EXECUTOR.execute方法来执行任务的，InternalHandler则是用于将执行环境从线程池切换到主线程。

AsyncTask的构造方法：

```java
public AsyncTask() {
    mWorker = new WorkerRunnable<Params, Result>() {
        public Result call() throws Exception {
            mTaskInvoked.set(true);
            Result result = null;
            try {
                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //noinspection unchecked
                result = doInBackground(mParams);
                Binder.flushPendingCommands();
            } catch (Throwable tr) {
                mCancelled.set(true);
                throw tr;
            } finally {
                postResult(result);
            }
            return result;
        }
    };

    mFuture = new FutureTask<Result>(mWorker) {
        @Override
        protected void done() {
            try {
                postResultIfNotInvoked(get());
            } catch (InterruptedException e) {
                android.util.Log.w(LOG_TAG, e);
            } catch (ExecutionException e) {
                throw new RuntimeException("An error occurred while executing doInBackground()",
                        e.getCause());
            } catch (CancellationException e) {
                postResultIfNotInvoked(null);
            }
        }
    };
}
```

这个WorkerRunnable的call方法什么时候被调用？还记得线程池刚执行时传递了一个参数mFuture吗？这个mFuture就是AsyncTask构造方法中的这个mFuture，之前我们说过，它是一个FutureTask对象，创建这个FutureTask对象时，我们将这个mWorker传给了它，FutureTask对象在等待执行的列队中最后被执行时会调用其run方法，我们再看看FutureTask的run方法：

```java
public void run() {
    if (state != NEW ||
        !U.compareAndSwapObject(this, RUNNER, null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

可以看到在run方法中有调用c.call()，这个c就是FutureTask中的全局变量callable，我们先找这个callable：

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```

看，在FutureTask的构造方法中，结合我们上面所说，这个callable就是AsyncTask构造方法中的mWorker。

```java
mWorker = new WorkerRunnable<Params, Result>() {
    public Result call() throws Exception {
        mTaskInvoked.set(true);
        Result result = null;
        try {
            Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
            //noinspection unchecked
            result = doInBackground(mParams);
            Binder.flushPendingCommands();
        } catch (Throwable tr) {
            mCancelled.set(true);
            throw tr;
        } finally {
            postResult(result);
        }
        return result;
    }
};
```

也就是说mWorker的call方法最终也会在线程池中执行。在该方法中，首先将mTaskInvoked设为true，表示当前任务已经被调用过了，然后再执行AsyncTask的doInBackground方法，接着将其返回值传递给postResult方法。

```java
private Result postResult(Result result) {
    @SuppressWarnings("unchecked")
    Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
            new AsyncTaskResult<Result>(this, result));
    message.sendToTarget();
    return result;
}
```

postResult方法会通过InternalHandler对象发送一个MESSAGE_POST_RESULT消息，InternalHandler类定义如下：

```java
private static class InternalHandler extends Handler {
    public InternalHandler() {
        super(Looper.getMainLooper());
    }

    @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
    @Override
    public void handleMessage(Message msg) {
        AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
        switch (msg.what) {
            case MESSAGE_POST_RESULT:
                // There is only one result
                result.mTask.finish(result.mData[0]);
                break;
            case MESSAGE_POST_PROGRESS:
                result.mTask.onProgressUpdate(result.mData);
                break;
        }
    }
}
```

由于InternalHandler是一个静态类，为了能将执行环境切换到主线程，这就要求InternalHandler必须在主线程中创建（原理在‘Android的消息机制’中说过），由于静态成员会在加载类的时候进行初始化，因此这就变相要求AsyncTask的类必须在主线程中加载。接着上面的，发送MESSAGE_POST_RESULT后，调用了AsyncTask的finish方法：

```java
private void finish(Result result) {
    if (isCancelled()) {
        onCancelled(result);
    } else {
        onPostExecute(result);
    }
    mStatus = Status.FINISHED;
}
```

上面这段代码就很容易理解了，如果AsyncTask被取消了，就调用onCancelled方法，否则调用onPostExecute方法，并设置状态为结束状态，而且doInBackground方法的返回值也被传给了onPostExecute方法。

到这AsyncTask的工作过程就分析完了。

> Android3.0及以上版本默认是串行执行的，但是也可以并行执行，采用AsyncTask的executeOnExecutor方法，但是这个方法是从Android3.0新添加的方法，不能在低版本上使用。


## HandlerThread

HandlerThread继承了Thread，它是一种具有消息循环可以使用Handler的Thread，它的实现很简单，就是在run方法中通过Looper.prepare()来创建消息队列，并通过Looper.loop()来开启消息循环，这样在实际使用中就允许在HandlerThread中创建Handler了。run方法如下：

```java
 @Override
public void run() {
    mTid = Process.myTid();
    Looper.prepare();
    synchronized (this) {
        mLooper = Looper.myLooper();
        notifyAll();
    }
    Process.setThreadPriority(mPriority);
    onLooperPrepared();
    Looper.loop();
    mTid = -1;
}
```

从run方法我们可以发现HandlerThread和普通的Thread有很大区别，普通的Thread主要用于在run方法中执行一个耗时任务，而HandlerThread在内部创建了消息队列，外界需要通过Handler的消息方式来通知HandlerThread执行一个具体的任务。由于HandlerThread的run方法是一个无限循环，所以当明确不需要再使用HandlerThread时，我们可以通过它的quit或者quitSafely方法来终止线程的执行。


## IntentService


IntentService是一种特殊的Service，它继承了Service并且它是一个抽象类，因此必须创建它的子类才能使用IntentService。IntentService可用于执行后台耗时的任务，当任务执行完毕后它会自动停止，同时因为它是Service，这导致它的优先级比单纯的要高很多，所以它不容易被系统杀死。

IntentService的onCreate方法：

```java
@Override
public void onCreate() {
    // TODO: It would be nice to have an option to hold a partial wakelock
    // during processing, and to have a static startService(Context, Intent)
    // method that would launch the service & hand off a wakelock.

    super.onCreate();
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();

    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
}
```

当IntentService第一次被启动时，会创建一个HandlerThread，然后使用它的Looper来构造一个Handler对象mServiceHandler，这样，通过mServiceHandler发送的消息最终都会在HandlerThread中执行，这样看来，IntentService也适合用于执行后台任务。

跟Service一样，每次启动，IntentService都会调用onStartCommand方法：

```java
/**
 * You should not override this method for your IntentService. Instead,
 * override {@link #onHandleIntent}, which the system calls when the IntentService
 * receives a start request.
 * @see android.app.Service#onStartCommand
 */
@Override
public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
    onStart(intent, startId);
    return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
}
```

我们看官方给我们解释的内容：不应该重写onStartCommand这个方法，而是重写onHandleIntent方法，因为每次Intentservice接收到启动请求时都会调用onHandleIntent方法。在onStartCommand方法中每次都会调用onSatrt方法

```java
@Override
public void onStart(@Nullable Intent intent, int startId) {
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    mServiceHandler.sendMessage(msg);
}
```

也就是每次启动就发送一条消息（包含了后台任务的Intent）到HanderThead去处理，而ServiceHandler收到消息后会将Intent传递给onHandleIntent方法去处理。

```java
private final class ServiceHandler extends Handler {
    public ServiceHandler(Looper looper) {
        super(looper);
    }

    @Override
    public void handleMessage(Message msg) {
        onHandleIntent((Intent)msg.obj);
        stopSelf(msg.arg1);
    }
}
```

> 这个Intent对象的内容和外界的startService(intent)中的intent的内容是完全一致的，通过这个Intent对象就可以解析出外界启动IntentService时所传递的参数，然后根据参数区分具体的后台任务。

当onHandleIntent方法执行完后，IntentService就会通过stopSelf(int startId)方法来尝试停止服务，为什么说是尝试呢？这就是stopSelf方法和stopSelf(int startId)方法的区别，stopSelf方法会立即停止服务，而这个时候可能还有其他消息未处理完，stopSelf(int startId)方法则会等到所有消息处理完后才停止服务。stopSelf(int startId)在尝试停止服务之前会判断最近启动服务的次数是否和startId相等，相等就立即停止服务，否则就不停止。

另外，由于每执行一个后台任务就启动一次IntentService，而IntentService内部则是通过消息的方式向HandlerThread请求执行任务，Handler中的Looper是顺序处理消息的，所以IntentService也是顺序执行后台任务的，当有多个后台任务时，这些任务也会按照外界发起的顺序依次执行。执行完最后一个时IntentService停止。


## Android中的线程池

使用线程池的好处：

- 重用线程池中的线程，避免因为线程的创建和销毁所带来的性能开销

- 能有效控制线程池的最大并发数，避免大量的线程之间因为互相抢占系统资源而导致阻塞现象

- 能够对线程进行简单管理，并提供定时执行以及指定间隔循环执行等功能

#### ThreadPoolExecutor

ThreadPoolExecutor是线程池的真正实现，它的构造方法提供一系列参数来配置线程池，各个参数的含义：

- corePoolSize：线程池的核心线程数，默认情况下，核心线程会在线程池中一直存活，即使它们处于闲置状态，如果将ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，那么闲置的核心线程在等待新任务到来时会有超时策略，这个时间间隔由keepAliveTime指定，当等待时间超过keepAliveTime指定的时长后，核心线程就会被终止。

- maximumPoolSize：线程池所能容纳的最大线程数，当活动线程数达到这个数值后，后续的新任务将会被阻塞。

- keepAliveTime：非核心线程闲置时的超时时长，超过这个时长，非核心线程就会被回收，当allowCoreThreadTimeOut设置为true时，keepAliveTime同样会作用于核心线程。

- unit：用于指定keepAliveTime参数的时间单位，这是一个枚举，常用的有TimeUnit.MILLISECONDS、TimeUnit.SECONDS、TimeUnit.MINUTES。

- workQueue：线程池中的任务队列，通过线程池的execute方法提交的Runnable对象会存储在这个参数中。

- threadFactory：线程工厂，为线程池提供创建新线程的功能。ThreadFactory是一个接口，它只有一个方法：newThread(Runnable r)

- RejectedExecutionHandler handler：这个参数不常用，当线程池无法执行新任务时，可能是由于任务队列已满或者是无法成功执行任务，这时，ThreadPoolExecutor会调用handler的rejectedExecution方法来通知调用者，默认情况下，rejectedExecution方法会直接抛出一个RejectedExecutionException，ThreadPoolExecutor为RejectedExecutionHandler提供了几个可选值，CallerRunsPolicy、AbortPolicy、DiscardPolicy、DiscardOldestPolicy，它们都是RejectedExecutionHandler的实现类，其中AbortPolicy是默认值。


**ThreadPoolExecutor执行任务时遵循下列规则：**

* 如果线程池中的线程数量未达到核心线程的数量，那么会直接启动一个核心线程来执行任务。

* 如果线程池中的线程数量已达到或超过核心线程的数量，那么任务会被插入到任务队列中排队等待执行。

* 如果在上面这条中无法将任务插入到任务队列中，一般是因为任务队列已满，这时如果线程数量未达到线程池规定的最大值，那么会会立刻启动一个非核心线程来执行任务。

* 如果上面这条种线程数量已经达到线程池规定的最大值，那么久拒绝执行此任务，ThreadPoolExecutor就会调用RejectedExecutionHandler的rejectedExecution方法来通知调用者。

下面这是AsyncTask的线程池配置：

```java
static {
    ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
            CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
            sPoolWorkQueue, sThreadFactory);
    threadPoolExecutor.allowCoreThreadTimeOut(true);
    THREAD_POOL_EXECUTOR = threadPoolExecutor;
}
```

规则基本是这样，以后我们自定义线程池也可以参照这样：

核心线程数 = CPU核心数 + 1
线程池最大线程数 = CPU核心数 * 2 + 1
核心线程无超时机制，非核心线程在闲置时的超时时间 = 1s
任务队列容量 = 128

#### 线程池分类

Android中常见的四种线程池：

* FixedThreadPool：是一种线程数量固定的线程池，当线程处于空闲状态时不会被回收，除非线程池被关闭。当所有的线程都处于活动状态时新任务会处于等待状态，直到有线程空闲出来。由于FixedThreadPool只有核心线程，并且这些核心线程不会被回收，所以它能够更快速的响应外界的请求。它的任务队列也没有大小限制。

* CachedThreadPool：是一种线程数量不定的线程池，它只有非核心线程，并且最大线程数为Integer.MAX_VALUE，这种线程池中的空闲线程都有超时机制，时长为60s。和FixedThreadPool不同的是，CachedThreadPool的任务队列其实相当于一个空集合，无法存储任务，所以任何任务都会立即被执行。这个特性决定了CachedThreadPool适合执行大量的耗时较少的任务，当整个线程池都处于闲置状态时，线程池中的线程都会超时而被停止，此时，CachedThreadPool中实际是没有任何线程的，几乎不占任何系统资源。

* ScheduledThreadPool：它的核心线程数是固定的，非核心线程数没有限制，并且当非核心线程闲置时会被立刻回收。这类线程池主要用于执行定时任务和具有固定周期的重复任务。

* SingleThreadPool：它只有一个核心线程，它确保所有的任务都在同一个线程中按顺序执行，SingleThreadPool的意义在于统一所有的外界任务到一个线程中，使得在这些任务之间不需要处理线程同步的问题。

这四种线程池的创建方式：

![四种线程池的创建方式](http://upload-images.jianshu.io/upload_images/1159224-2675584dd2a40788.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

除了系统这四种线程池，还可以根据实际需要灵活地配置线程池。





