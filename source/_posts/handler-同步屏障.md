---
title: handler 同步屏障
date: 2019-10-11 20:39:24
tags: android
---
  今天看来一篇文章 [app卡顿分析](https://juejin.im/post/5d837cd1e51d4561cb5ddf66)里面分析了view的绘制流程 提到在view.requestLayout的时候会通过handler发送一个同步屏障消息提高ui事件的优先级 去看下handler的源码学习一下

`handler`消息 分同步消息和异步消息  相当于消息分了等级 在开启同步屏障的时候 异步消息会有高的优先级 在消息队列里会优先被取出 核心方法下面这些
<!-- more -->
### 发起消息屏障
```java
   private int postSyncBarrier(long when) {
        // Enqueue a new sync barrier token.
        // We don't need to wake the queue because the purpose of a barrier is to stall it.
        synchronized (this) {
            final int token = mNextBarrierToken++;
            final Message msg = Message.obtain();
            msg.markInUse();
            msg.when = when;
            msg.arg1 = token;

            Message prev = null;
            Message p = mMessages;
            if (when != 0) {
                while (p != null && p.when <= when) {
                    prev = p;
                    p = p.next;
                }
            }
            if (prev != null) { // invariant: p == prev.next
                msg.next = p;
                prev.next = msg;
            } else {
                msg.next = p;
                mMessages = msg;
            }
            return token;
        }
    }
```
### 移除消息屏障
```java
    public void removeSyncBarrier(int token) {
        // Remove a sync barrier token from the queue.
        // If the queue is no longer stalled by a barrier then wake it.
        synchronized (this) {
            Message prev = null;
            Message p = mMessages;
            while (p != null && (p.target != null || p.arg1 != token)) {
                prev = p;
                p = p.next;
            }
            if (p == null) {
                throw new IllegalStateException("The specified message queue synchronization "
                        + " barrier token has not been posted or has already been removed.");
            }
            final boolean needWake;
            if (prev != null) {
                prev.next = p.next;
                needWake = false;
            } else {
                mMessages = p.next;
                needWake = mMessages == null || mMessages.target != null;
            }
            p.recycleUnchecked();

            // If the loop is quitting then it is already awake.
            // We can assume mPtr != 0 when mQuitting is false.
            if (needWake && !mQuitting) {
                nativeWake(mPtr);
            }
        }
    }
```
### 取下一条消息时 判断是否处于同步屏障 是的话先处理异步消息
`MessageQueue#next`
```java
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
                            // @@@ 只是从messages里移除了异步消息 没有移除屏障消息
                            prevMsg.next = msg.next;
                        } else {
                            // @@@ 这种情况下 从messages里移除了消息
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
                ....
                ....
```
### 消息入列
```java
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
```
有两点没看懂

1.入异步消息队列时 needWake参数的作用 
2.还有一点疑问 如何让同步屏障的消息 一直处于队列中呢？ 猜测是才取消息的时候链表操作里面有处理 暂时还看不懂

- 关于第一个问题 这篇文章里讲的比较清除 [文章连接](https://blog.csdn.net/qingtiantianqing/article/details/72783952)(不过这篇文章本身也是转载的😄)
- 关于第二点 又仔细看来下next方法里针对异步消息的处理 对于开启同步屏障的流程 prevMsg 不为空 所以没有改变messages 导致屏障消息还在链表中

      // Indicates whether next() is blocked waiting in pollOnce() with a non-zero timeout.
      private boolean mBlocked;

      // The next barrier token.
      // Barriers are indicated by messages with a null target whose arg1 field carries the token.
      private int mNextBarrierToken;