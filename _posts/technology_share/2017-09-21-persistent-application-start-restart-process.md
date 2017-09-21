---
layout: post
title: persistent应用的启动过程以及重启机制
category: 技术分享
tags: Android、persistent
---



在AndroidManifest.xml定义中，application有这么一个属性android:persistent，根据字面意思来理解就是说该应用是可持久的，也即是常驻的应用。其实就是这么个理解，被android:persistent修饰的应用会在系统启动之后被AMS启动。

在系统启动时，ActivityManagerService会调用systemReady()方法来加载所有persistent属性为true的应用。

```java
public void systemReady(final Runnable goingCallback) {
    
    ......

    synchronized (this) {
        // Only start up encryption-aware persistent apps; once user is
        // unlocked we'll come back around and start unaware apps
        startPersistentApps(PackageManager.MATCH_DIRECT_BOOT_AWARE);

        ......
    }
}

private void startPersistentApps(int matchFlags) {
    if (mFactoryTest == FactoryTest.FACTORY_TEST_LOW_LEVEL) return;

    synchronized (this) {
        try {
            final List<ApplicationInfo> apps = AppGlobals.getPackageManager()
                    .getPersistentApplications(STOCK_PM_FLAGS | matchFlags).getList();
            for (ApplicationInfo app : apps) {
                if (!"android".equals(app.packageName)) {
                    addAppLocked(app, false, null /* ABI override */);
                }
            }
        } catch (RemoteException ex) {
        }
    }
}
```

上面的代码中persistent应用列表是通过PackageManagerService类的getPersistentApplications()方法来获取的，方法实现如下：

```
@Override
public @NonNull ParceledListSlice<ApplicationInfo> getPersistentApplications(int flags) {
    return new ParceledListSlice<>(getPersistentApplicationsInternal(flags));
}

private @NonNull List<ApplicationInfo> getPersistentApplicationsInternal(int flags) {
    final ArrayList<ApplicationInfo> finalList = new ArrayList<ApplicationInfo>();

    // reader
    synchronized (mPackages) {
        final Iterator<PackageParser.Package> i = mPackages.values().iterator();
        final int userId = UserHandle.getCallingUserId();
        while (i.hasNext()) {
            final PackageParser.Package p = i.next();
            if (p.applicationInfo == null) continue;

            final boolean matchesUnaware = ((flags & MATCH_DIRECT_BOOT_UNAWARE) != 0)
                    && !p.applicationInfo.isDirectBootAware();
            final boolean matchesAware = ((flags & MATCH_DIRECT_BOOT_AWARE) != 0)
                    && p.applicationInfo.isDirectBootAware();

            if ((p.applicationInfo.flags & ApplicationInfo.FLAG_PERSISTENT) != 0
                    && (!mSafeMode || isSystemApp(p))
                    && (matchesUnaware || matchesAware)) {
                PackageSetting ps = mSettings.mPackages.get(p.packageName);
                if (ps != null) {
                    ApplicationInfo ai = PackageParser.generateApplicationInfo(p, flags,
                            ps.readUserState(userId), userId);
                    if (ai != null) {
                        finalList.add(ai);
                    }
                }
            }
        }
    }

    return finalList;
}
```

在PackageManagerService中，有一个记录所有的程序包信息的哈希表（mPackages），每个表项中含有ApplicationInfo信息，该信息的flags（int型）数据中有一个专门的bit用于表示persistent。getPersistentApplications()方法会遍历这张表，找出所有persistent包，并返回ArrayList<ApplicationInfo>。不过从代码中我们看到，除了有FLAG_PERSISTENT标志的应用，处于非安全模式或者是系统应用、直接启动的应用，满足这些条件的应用才会被加到persistent列表中。

接着systemReady方法会遍历persistent列表中的ApplicationInfo，然后对包名不为“android”的ApplicationInfo执行addAppLocked方法，看看addAppLocked的实现：

```
final ProcessRecord addAppLocked(ApplicationInfo info, boolean isolated, String abiOverride) {
    ProcessRecord app;
    if (!isolated) {
        app = getProcessRecordLocked(info.processName, info.uid, true);
    } else {
        app = null;
    }

    if (app == null) {
        app = newProcessRecordLocked(info, null, isolated, 0);
        updateLruProcessLocked(app, false, null);
        updateOomAdjLocked();
    }

    // This package really, really can not be stopped.
    try {
        AppGlobals.getPackageManager().setPackageStoppedState(
                info.packageName, false, UserHandle.getUserId(app.uid));
    } catch (RemoteException e) {
    } catch (IllegalArgumentException e) {
        Slog.w(TAG, "Failed trying to unstop package "
                + info.packageName + ": " + e);
    }

    if ((info.flags & PERSISTENT_MASK) == PERSISTENT_MASK) {
        app.persistent = true;
        app.maxAdj = ProcessList.PERSISTENT_PROC_ADJ;
    }
    if (app.thread == null && mPersistentStartingProcesses.indexOf(app) < 0) {
        mPersistentStartingProcesses.add(app);
        startProcessLocked(app, "added application", app.processName, abiOverride,
                null /* entryPoint */, null /* entryPointArgs */);
    }

    return app;
}
```


这个方法的作用主要是添加一个与App进程对应的ProcessRecord节点，如果这个节点已经添加过了，那么是不会重复添加的。在添加节点的动作完成以后，addAppLocked()还会检查App进程是否已经启动好了，如果尚未开始启动，此时就会调用startProcessLocked()来启动这个进程。既然addAppLocked()试图确认App“正在正常运作”或者“将被正常启动”，那么其对应的package就不可能处于stopped状态，这就是上面代码调用setPackageStoppedState(..., false,...)以及注释“This package really, really can not be stopped.”的作用。


因为启动过程异步，所以对于正在启动但尚未启动完成的ApplicationInfo，AMS会把他们添加到一个缓冲列表中也就是mPersistentStartingProcesses这个变量中记录，启动一个进程则是调用startProcessLocked方法，其中启动代码如下：

```
Process.ProcessStartResult startResult = Process.start(entryPoint,
                    app.processName, uid, uid, gids, debugFlags, mountExternal,
                    app.info.targetSdkVersion, app.info.seinfo, requiredAbi, instructionSet,
                    app.info.dataDir, entryPointArgs);
```

一旦启动完成，这个用户进程会被attach到系统中，这个我们看ActivityThread中的main方法就可知：

```
ActivityThread thread = new ActivityThread();
thread.attach(false);
```


```
private void attach(boolean system) {
    sCurrentActivityThread = this;
    mSystemThread = system;
    if (!system) {
        ViewRootImpl.addFirstDrawHandler(new Runnable() {
            @Override
            public void run() {
                ensureJitEnabled();
            }
        });
        android.ddm.DdmHandleAppName.setAppName("<pre-initialized>",
                                                UserHandle.myUserId());
        RuntimeInit.setApplicationObject(mAppThread.asBinder());
        final IActivityManager mgr = ActivityManagerNative.getDefault();
        try {
            mgr.attachApplication(mAppThread);
        } catch (RemoteException ex) {
            throw ex.rethrowFromSystemServer();
        }
        ......
    } else {
        ......
    }

    ......
}
```

attach方法中会判断不是系统应用就会调用AMS的attachApplication方法(AMS继承自ActivityManagerNative)，在attach过程中，ActivityThread会将对应的application attach到AM中去，交与AM去管理，在这个方法中传递了一个变量mAppThread，它是一个ApplicationThread对象，ApplicationThread是定义在ActivityThread类中的继承ApplicationThreadNative的私有类，它继承自Binder且实现了IApplicationThread接口，所以在AMS中的关于ApplicationThread的参数都是IApplicationThread的形式，mAppThread可以看作是当前进程主线程的核心，它负责处理本进程与其他进程(主要是AM)之间的通信，同时通过attachApplication将mAppThread的代理Binder传递给AM。接着ApplicationInfo走到attachApplicationLocked方法。


```
@Override
public final void attachApplication(IApplicationThread thread) {
    synchronized (this) {
        int callingPid = Binder.getCallingPid();
        final long origId = Binder.clearCallingIdentity();
        attachApplicationLocked(thread, callingPid);
        Binder.restoreCallingIdentity(origId);
    }
}
```

```
private final boolean attachApplicationLocked(IApplicationThread thread, int pid) {

    ......

    // If this application record is still attached to a previous
    // process, clean it up now.
    if (app.thread != null) {
        handleAppDiedLocked(app, true, true);
    }

    final String processName = app.processName;
    try {
    	// 注册进程死亡监听器
        AppDeathRecipient adr = new AppDeathRecipient(app, pid, thread);
        thread.asBinder().linkToDeath(adr, 0);
        app.deathRecipient = adr;
    } catch (RemoteException e) {
        app.resetPackageList(mProcessStats);
        startProcessLocked(app, "link fail", processName);
        return false;
    }

    ......
  
    thread.bindApplication(processName, appInfo, providers, app.instrumentationClass,
            profilerInfo, app.instrumentationArguments, app.instrumentationWatcher,
            app.instrumentationUiAutomationConnection, testMode,
            mBinderTransactionTrackingEnabled, enableTrackAllocation,
            isRestrictedBackupMode || !normalMode, app.persistent,
            new Configuration(mConfiguration), app.compat,
            getCommonServicesLocked(app.isolated),
            mCoreSettingsObserver.getCoreSettingsLocked());

    // Remove this record from the list of starting applications.
    mPersistentStartingProcesses.remove(app);

    ......

    return true;
}
```

在attachApplicationLocked方法中AMS调用到了IPC通信调用mAppThread的bindApplication方法，然后将该ProcessRecord节点在mPersistentStartingProcesses列表中移除。
这里又回调到了ApplicationThread类中的bindApplication方法：

```
public final void bindApplication(String processName, ApplicationInfo appInfo,
                List<ProviderInfo> providers, ComponentName instrumentationName,
                ProfilerInfo profilerInfo, Bundle instrumentationArgs,
                IInstrumentationWatcher instrumentationWatcher,
                IUiAutomationConnection instrumentationUiConnection, int debugMode,
                boolean enableBinderTracking, boolean trackAllocation,
                boolean isRestrictedBackupMode, boolean persistent, Configuration config,
                CompatibilityInfo compatInfo, Map<String, IBinder> services, Bundle coreSettings) {

    if (services != null) {
        // Setup the service cache in the ServiceManager
        ServiceManager.initServiceCache(services);
    }

    setCoreSettings(coreSettings);

    AppBindData data = new AppBindData();
    data.processName = processName;
    data.appInfo = appInfo;
    data.providers = providers;
    data.instrumentationName = instrumentationName;
    data.instrumentationArgs = instrumentationArgs;
    data.instrumentationWatcher = instrumentationWatcher;
    data.instrumentationUiAutomationConnection = instrumentationUiConnection;
    data.debugMode = debugMode;
    data.enableBinderTracking = enableBinderTracking;
    data.trackAllocation = trackAllocation;
    data.restrictedBackupMode = isRestrictedBackupMode;
    data.persistent = persistent;
    data.config = config;
    data.compatInfo = compatInfo;
    data.initProfilerInfo = profilerInfo;
    sendMessage(H.BIND_APPLICATION, data);
}
```

上述代码中通过消息机制向ActivityThread自身维护的handler发送BIND_APPLICATION消息。ActivityThread自身维护的handler对消息BIND_APPLICATION的处理调用了handleBindApplication方法，而这个方法中我们可以看到下面这条代码：

```
mInstrumentation.callApplicationOnCreate(app);
```

这句作用是调用persistent应用的Applicaiton中的onCreate方法。


上面说了persistent应用的启动过程，既然是persistent的，那么当用应用出现异常时它也需要自动重启。这里Android系统中实现自动重启的做法是这样的，我们回头看看上面attachApplicationLocked方法中在bindApplication之前，会构建一个AppDeathRecipient，相当于一个监听器，然后把这个监听器跟persistent进程绑在一起，由AMS来监听，当persistent进程意外死亡时，AMS就能知道，并且会尝试重新启动这个应用。


AppDeathRecipient的实现如下：


```
private final class AppDeathRecipient implements IBinder.DeathRecipient {
    final ProcessRecord mApp;
    final int mPid;
    final IApplicationThread mAppThread;

    AppDeathRecipient(ProcessRecord app, int pid,
            IApplicationThread thread) {
        if (DEBUG_ALL) Slog.v(
            TAG, "New death recipient " + this
            + " for thread " + thread.asBinder());
        mApp = app;
        mPid = pid;
        mAppThread = thread;
    }

    @Override
    public void binderDied() {
        if (DEBUG_ALL) Slog.v(
            TAG, "Death received in " + this
            + " for thread " + mAppThread.asBinder());
        synchronized(ActivityManagerService.this) {
            appDiedLocked(mApp, mPid, mAppThread, true);
        }
    }
}
```

当persistent应用意外退出时，系统会回调AppDeathRecipient的binderDied方法，这个方法中只会执行appDiedLocked这个方法，而最终会执行handleAppDiedLocked这个方法：

```
private final void handleAppDiedLocked(ProcessRecord app,
            boolean restarting, boolean allowRestart) {
    int pid = app.pid;
    boolean kept = cleanUpApplicationRecordLocked(app, restarting, allowRestart, -1,
            false /*replacingPid*/);
    if (!kept && !restarting) {
        removeLruProcessLocked(app);
        if (pid > 0) {
            ProcessList.remove(pid);
        }
    }

    if (mProfileProc == app) {
        clearProfilerLocked();
    }

    // Remove this application's activities from active lists.
    boolean hasVisibleActivities = mStackSupervisor.handleAppDiedLocked(app);

    app.activities.clear();

    if (app.instrumentationClass != null) {
        Slog.w(TAG, "Crash of app " + app.processName
              + " running instrumentation " + app.instrumentationClass);
        Bundle info = new Bundle();
        info.putString("shortMsg", "Process crashed.");
        finishInstrumentationLocked(app, Activity.RESULT_CANCELED, info);
    }

    if (!restarting && hasVisibleActivities
            && !mStackSupervisor.resumeFocusedStackTopActivityLocked()) {
        // If there was nothing to resume, and we are not already restarting this process, but
        // there is a visible activity that is hosted by the process...  then make sure all
        // visible activities are running, taking care of restarting this process.
        mStackSupervisor.ensureActivitiesVisibleLocked(null, 0, !PRESERVE_WINDOWS);
    }
}
```

这个方法的作用就是将进程从ActivityManager中移除，其中的变量kept是根据方法cleanUpApplicationRecordLocked的结果，其意义是是否要保留这个进程，我们看看它的实现：

```
private final boolean cleanUpApplicationRecordLocked(ProcessRecord app,
            boolean restarting, boolean allowRestart, int index, boolean replacingPid) {
    
    ......

    if (!app.persistent || app.isolated) {
        if (DEBUG_PROCESSES || DEBUG_CLEANUP) Slog.v(TAG_CLEANUP,
                "Removing non-persistent process during cleanup: " + app);
        if (!replacingPid) {
            removeProcessNameLocked(app.processName, app.uid);
        }
        if (mHeavyWeightProcess == app) {
            mHandler.sendMessage(mHandler.obtainMessage(CANCEL_HEAVY_NOTIFICATION_MSG,
                    mHeavyWeightProcess.userId, 0));
            mHeavyWeightProcess = null;
        }
    } else if (!app.removed) {
        // This app is persistent, so we need to keep its record around.
        // If it is not already on the pending app list, add it there
        // and start a new process for it.
        if (mPersistentStartingProcesses.indexOf(app) < 0) {
            mPersistentStartingProcesses.add(app);
            restart = true;
        }
    }
   	......

    if (restart && !app.isolated) {
        // We have components that still need to be running in the
        // process, so re-launch it.
        if (index < 0) {
            ProcessList.remove(app.pid);
        }
        addProcessNameLocked(app);
        startProcessLocked(app, "restart", app.processName);
        return true;
    } else if (app.pid > 0 && app.pid != MY_PID) {
        // Goodbye!
        boolean removed;
        synchronized (mPidsSelfLocked) {
            mPidsSelfLocked.remove(app.pid);
            mHandler.removeMessages(PROC_START_TIMEOUT_MSG, app);
        }
        mBatteryStatsService.noteProcessFinish(app.processName, app.info.uid);
        if (app.isolated) {
            mBatteryStatsService.removeIsolatedUid(app.uid, app.info.uid);
        }
        app.setPid(0);
    }
    return false;
}
```

我们看到如果是persistent应用且还没有被移除，AMS会重新将它添加到mPersistentStartingProcesses这个启动缓存列表中，并再调用startProcessLocked方法重启进程。到此，persistent应用的重启机制也就说完了。



参考文章：

- [说说Android应用的persistent属性](https://my.oschina.net/youranhongcha/blog/269591)
- [android persistent属性研究](http://blog.csdn.net/windskier/article/details/6560925)







