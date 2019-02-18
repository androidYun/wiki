Toast源码查看

### Toast使用姿势

Toast.makeText(context,"提示内容", Toast.LENGTH_SHORT).show()

当初刚接触Toast的时候 出现 怎么都不提示，还怀疑源码bug，最终发现show() 忘了 添加，show是一个非常重要的方法，细看源码

### Toast正确使用姿势和优化

1,Context 的使用

```
Toast.makeText(context,"提示内容", Toast.LENGTH_SHORT).show()
```

需要用到context, 这个context 尽量不要用Activity或者Fragment的context, 可以用Application 的context，因为显示Toast的时候，Activity和Fragment没销毁掉，那么Context 被使用的时候，就会报空指针

2，Toast的封装

```java
 private static Toast toast = null;
    public static void showToast(String text) {
        if (TextUtils.isEmpty(text)) return;
        if (toast == null) {
            toast = Toast.makeText(context, text, Toast.LENGTH_SHORT);
        } else {
            toast.setText(text);
        }
        toast.show();
    }
```

避免Toast多次创建

3，Toast源码获知 弹出时间只能是2s 或者 3.5秒

### Toast源码解析

```java
 /**
     * Constructs an empty Toast object.  If looper is null, Looper.myLooper() is used.
     * @hide
     */
    public Toast(@NonNull Context context, @Nullable Looper looper) {
        mContext = context;
        /*
        *一个 TN 如果在activity或者Fragment里面 looper 为空 因为里面会用Looper.myLooper();获取looper
        *如果在子线程里面 不传 looper 就会报错，
        */
        mTN = new TN(context.getPackageName(), looper);
        mTN.mY = context.getResources().getDimensionPixelSize(
                com.android.internal.R.dimen.toast_y_offset);
        mTN.mGravity = context.getResources().getInteger(
                com.android.internal.R.integer.config_toastDefaultGravity);
    }
```

Toast构造方法里面主要 初始化 TN 然后设置TN 坐标 和位置

```java
 private static class TN extends ITransientNotification.Stub {
        private final WindowManager.LayoutParams mParams = new WindowManager.LayoutParams();

        private static final int SHOW = 0;
        private static final int HIDE = 1;
        private static final int CANCEL = 2;
        final Handler mHandler;

        int mGravity;
        int mX, mY;
        float mHorizontalMargin;
        float mVerticalMargin;


        View mView;
        View mNextView;
        int mDuration;

        WindowManager mWM;

        String mPackageName;

        static final long SHORT_DURATION_TIMEOUT = 4000;
        static final long LONG_DURATION_TIMEOUT = 7000;

        TN(String packageName, @Nullable Looper looper) {
            // XXX This should be changed to use a Dialog, with a Theme.Toast
            // defined that sets up the layout params appropriately.
            final WindowManager.LayoutParams params = mParams;
            params.height = WindowManager.LayoutParams.WRAP_CONTENT;
            params.width = WindowManager.LayoutParams.WRAP_CONTENT;
            params.format = PixelFormat.TRANSLUCENT;
            params.windowAnimations = com.android.internal.R.style.Animation_Toast;
            params.type = WindowManager.LayoutParams.TYPE_TOAST;
            params.setTitle("Toast");
            params.flags = WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON
                    | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                    | WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE;

            mPackageName = packageName;
//初始化looper
            if (looper == null) {
                // Use Looper.myLooper() if looper is not specified.
                looper = Looper.myLooper();
                if (looper == null) {
                    throw new RuntimeException(
                            "Can't toast on a thread that has not called Looper.prepare()");
                }
            }
            //创建一个Handler
            mHandler = new Handler(looper, null) {
                @Override
                public void handleMessage(Message msg) {
                    switch (msg.what) {
                    //去显示Toast
                        case SHOW: {
                            IBinder token = (IBinder) msg.obj;
                            handleShow(token);
                            break;
                        }
                        //隐藏Toast
                        case HIDE: {
                            handleHide();
                            // Don't do this in handleHide() because it is also invoked by
                            // handleShow()
                            mNextView = null;
                            break;
                        }
                        //取消显示
                        case CANCEL: {
                            handleHide();
                            // Don't do this in handleHide() because it is also invoked by
                            // handleShow()
                            mNextView = null;
                            try {
                                getService().cancelToast(mPackageName, TN.this);
                            } catch (RemoteException e) {
                            }
                            break;
                        }
                    }
                }
            };
        }

```

需要显示View 的地方 肯定要有Handler 因为Toast能在自线程中显示，说明 肯定中间有一个一个东西处理，媒介，所以必然离不开handler

```java
    public void show() {
        if (mNextView == null) {
            throw new RuntimeException("setView must have been called");
        }
//获取通知服务NotificationManagerService  NotificationManagerService继承与 SystemService，所以当
//activity被销毁，app退出，Toast都能显示，那肯定属于系统消息范畴
        INotificationManager service = getService();
        String pkg = mContext.getOpPackageName();
        TN tn = mTN;
        tn.mNextView = mNextView;
        try {
        //把需要弹出的Toast需要的参数 放入 queue 堆中 queue 是先进先出 
            service.enqueueToast(pkg, tn, mDuration);
        } catch (RemoteException e) {
            // Empty
        }
    }
```

```java
  @Override
        public void enqueueToast(String pkg, ITransientNotification callback, int duration)
        {
            if (DBG) {
                Slog.i(TAG, "enqueueToast pkg=" + pkg + " callback=" + callback
                        + " duration=" + duration);
            }

            if (pkg == null || callback == null) {
                Slog.e(TAG, "Not doing toast. pkg=" + pkg + " callback=" + callback);
                return ;
            }
            final boolean isSystemToast = isCallerSystemOrPhone() || ("android".equals(pkg));
            final boolean isPackageSuspended =
                    isPackageSuspendedForUser(pkg, Binder.getCallingUid());
//判断是否是系统消息
            if (ENABLE_BLOCKED_TOASTS && !isSystemToast &&
                    (!areNotificationsEnabledForPackage(pkg, Binder.getCallingUid())
                            || isPackageSuspended)) {
                Slog.e(TAG, "Suppressing toast from package " + pkg
                        + (isPackageSuspended
                                ? " due to package suspended by administrator."
                                : " by user request."));
                return;
            }

            synchronized (mToastQueue) {
                int callingPid = Binder.getCallingPid();
                long callingId = Binder.clearCallingIdentity();
                try {
                    ToastRecord record;
                    int index;
                    // All packages aside from the android package can enqueue one toast at a time
                    if (!isSystemToast) {
                        index = indexOfToastPackageLocked(pkg);
                    } else {
                        index = indexOfToastLocked(pkg, callback);
                    }

                    // If the package already has a toast, we update its toast
                    // in the queue, we don't move it to the end of the queue.
                    if (index >= 0) {
                        record = mToastQueue.get(index);
                        record.update(duration);
                        try {
                            record.callback.hide();
                        } catch (RemoteException e) {
                        }
                        record.update(callback);
                    } else {
                        Binder token = new Binder();
                        mWindowManagerInternal.addWindowToken(token, TYPE_TOAST, DEFAULT_DISPLAY);
                        record = new ToastRecord(callingPid, pkg, callback, duration, token);
                        mToastQueue.add(record);
                        index = mToastQueue.size() - 1;
                    }
                    keepProcessAliveIfNeededLocked(callingPid);
                    // If it's at index 0, it's the current toast.  It doesn't matter if it's
                    // new or just been updated.  Call back and tell it to show itself.
                    // If the callback fails, this will remove it from the list, so don't
                    // assume that it's valid after this.
                    if (index == 0) {
                        showNextToastLocked();
                    }
                } finally {
                    Binder.restoreCallingIdentity(callingId);
                }
            }
        }
        @GuardedBy("mToastQueue")
    void showNextToastLocked() {
        ToastRecord record = mToastQueue.get(0);
        while (record != null) {
            if (DBG) Slog.d(TAG, "Show pkg=" + record.pkg + " callback=" + record.callback);
            try {
                record.callback.show(record.token);
                scheduleDurationReachedLocked(record);
                return;
            } catch (RemoteException e) {
                Slog.w(TAG, "Object died trying to show notification " + record.callback
                        + " in package " + record.pkg);
                // remove it from the list and let the process die
                int index = mToastQueue.indexOf(record);
                if (index >= 0) {
                    mToastQueue.remove(index);
                }
                keepProcessAliveIfNeededLocked(record.pid);
                if (mToastQueue.size() > 0) {
                    record = mToastQueue.get(0);
                } else {
                    record = null;
                }
            }
        }
    }

        @Override
        public void cancelToast(String pkg, ITransientNotification callback) {
            Slog.i(TAG, "cancelToast pkg=" + pkg + " callback=" + callback);

            if (pkg == null || callback == null) {
                Slog.e(TAG, "Not cancelling notification. pkg=" + pkg + " callback=" + callback);
                return ;
            }

            synchronized (mToastQueue) {
                long callingId = Binder.clearCallingIdentity();
                try {
                    int index = indexOfToastLocked(pkg, callback);
                    if (index >= 0) {
                        cancelToastLocked(index);
                    } else {
                        Slog.w(TAG, "Toast already cancelled. pkg=" + pkg
                                + " callback=" + callback);
                    }
                } finally {
                    Binder.restoreCallingIdentity(callingId);
                }
            }
        }

```

最重要的一句代码就是 record.callback.show(record.token); callback 就是 TN 实现的ITransientNotification.Stub 这个类有两个实现放发 就是 show(token) 和hide(token)

我们看看 show里面怎么实现的

```java
  /**
         * schedule handleShow into the right thread
         */
        @Override
        public void show(IBinder windowToken) {
            if (localLOGV) Log.v(TAG, "SHOW: " + this);
            mHandler.obtainMessage(SHOW, windowToken).sendToTarget();
        }
```

就是用handler 发送显示行业隐藏的Message

```java
  public void handleShow(IBinder windowToken) {
            if (localLOGV) Log.v(TAG, "HANDLE SHOW: " + this + " mView=" + mView
                    + " mNextView=" + mNextView);
            // If a cancel/hide is pending - no need to show - at this point
            // the window token is already invalid and no need to do any work.
            if (mHandler.hasMessages(CANCEL) || mHandler.hasMessages(HIDE)) {
                return;
            }
            if (mView != mNextView) {
                // remove the old view if necessary
                handleHide();
                mView = mNextView;
                Context context = mView.getContext().getApplicationContext();
                String packageName = mView.getContext().getOpPackageName();
                if (context == null) {
                    context = mView.getContext();
                }
                mWM = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
                // We can resolve the Gravity here by using the Locale for getting
                // the layout direction
                final Configuration config = mView.getContext().getResources().getConfiguration();
                final int gravity = Gravity.getAbsoluteGravity(mGravity, config.getLayoutDirection());
                mParams.gravity = gravity;
                if ((gravity & Gravity.HORIZONTAL_GRAVITY_MASK) == Gravity.FILL_HORIZONTAL) {
                    mParams.horizontalWeight = 1.0f;
                }
                if ((gravity & Gravity.VERTICAL_GRAVITY_MASK) == Gravity.FILL_VERTICAL) {
                    mParams.verticalWeight = 1.0f;
                }
                mParams.x = mX;
                mParams.y = mY;
                mParams.verticalMargin = mVerticalMargin;
                mParams.horizontalMargin = mHorizontalMargin;
                mParams.packageName = packageName;
                mParams.hideTimeoutMilliseconds = mDuration ==
                    Toast.LENGTH_LONG ? LONG_DURATION_TIMEOUT : SHORT_DURATION_TIMEOUT;
                mParams.token = windowToken;
                if (mView.getParent() != null) {
                    if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
                    mWM.removeView(mView);
                }
                if (localLOGV) Log.v(TAG, "ADD! " + mView + " in " + this);
                // Since the notification manager service cancels the token right
                // after it notifies us to cancel the toast there is an inherent
                // race and we may attempt to add a window after the token has been
                // invalidated. Let us hedge against that.
                try {
                    mWM.addView(mView, mParams);
                    trySendAccessibilityEvent();
                } catch (WindowManager.BadTokenException e) {
                    /* ignore */
                }
            }
        }
        
```

handlerShow() 里面主要实现 先判断是否有cancel或者hide的消息 如果有这样的消息 就显示 ，否则显示 先移除 当前Parent的的子View
