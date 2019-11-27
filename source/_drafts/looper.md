---
title: looper
tags:  源码学习
---
先看下类注释 我觉得阅读类注释是个好习惯
```java
/**
  * Class used to run a message loop for a thread.  Threads by default do
  * not have a message loop associated with them; to create one, call
  * {@link #prepare} in the thread that is to run the loop, and then
  * {@link #loop} to have it process messages until the loop is stopped.
  *
  * <p>Most interaction with a message loop is through the
  * {@link Handler} class.
  *
  * <p>This is a typical example of the implementation of a Looper thread,
  * using the separation of {@link #prepare} and {@link #loop} to create an
  * initial Handler to communicate with the Looper.
  *
  * <pre>
  *  class LooperThread extends Thread {
  *      public Handler mHandler;
  *
  *      public void run() {
  *          Looper.prepare();
  *
  *          mHandler = new Handler() {
  *              public void handleMessage(Message msg) {
  *                  // process incoming messages here
  *              }
  *          };
  *
  *          Looper.loop();
  *      }
  *  }</pre>
  */
```
```java
    /** Initialize the current thread as a looper.
      * This gives you a chance to create handlers that then reference
      * this looper, before actually starting the loop. Be sure to call
      * {@link #loop()} after calling this method, and end it by calling
      * {@link #quit()}.
      */
    public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

```java
    /**
     * Initialize the current thread as a looper, marking it as an
     * application's main looper. The main looper for your application
     * is created by the Android environment, so you should never need
     * to call this function yourself.  See also: {@link #prepare()}
     */
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

```java
    /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        ....
    }    
```