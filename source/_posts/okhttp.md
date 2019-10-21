---
title: okhttp3 的工作原理与源码分析
tags: []
categories: []
toc: false
date: 2019-03-08 14:08:49
---

OkHttpClient 应该被设计为一个单例，以便复用连接池与线程池，避免造成资源不必要的资源浪费

### okhttp 的连接池设计

为了避免在请求同一个连接时在重新建立连接上产生的时延，Okhttp设计了socket连接 复用机制。

1. 每次有新的请求到来时，会先去连接池中查找当前请求的连接是否已经存在。

2. 如果连接存在（连接相同的标准是：），直接复用。

3. 如果不存在，则重新创建一个连接，然后将连接放进连接池,在放进连接池时会通过`线程池`对连接池进行清理与维护，用来释放已经`空闲超时`的连接.


#### okhttp 的连接池的清理逻辑

 直接上源码

1. 如果连接池中不存在当前请求的连接，则先判断当前连接池是否在执行清理操作，如果没有先执行连接池清理操作。然后将新的请求添加到连接池中
```java
void put(RealConnection connection) {
    assert (Thread.holdsLock(this));
    if (!cleanupRunning) { //如果不是是在清理中，则执行清理操作
      cleanupRunning = true;
      executor.execute(cleanupRunnable); //有线程池管理，执行清理操作
    }
    connections.add(connection);//添加连接到线程池
}
```

2. 执行清理操作，这里要注意两点：如果清理返回的结果为 -1，那么表示连接池中没有连接，那么清理线程也就没有一直循环下去的必要了。如果在清理的过程中发现有空闲连接，但是空闲连接没有达到超时时间，那么当前执行清理的线程会休眠（时间为达到超时的时间），如果在此休眠期间没有重新调用该连接，那么超时时间之后会执行清理操作。

```java
private final Runnable cleanupRunnable = new Runnable() {
    @Override public void run() {
      while (true) {
        long waitNanos = cleanup(System.nanoTime());//执行清理操作 返回需要等待的时间
        if (waitNanos == -1) return; //如果返回-1表示连接池为空跳出循环
        if (waitNanos > 0) { //如果等待超时的时间大于0，那么当前线程会清理休眠等待时间
          long waitMillis = waitNanos / 1000000L;
          waitNanos -= (waitMillis * 1000000L);
          synchronized (ConnectionPool.this) {
            try {
              ConnectionPool.this.wait(waitMillis, (int) waitNanos);
            } catch (InterruptedException ignored) {
            }
          }
        }
      }
    }
  };
```
3.清理操作主要由cleanup执行，clean的时候分以下几个步骤：

	1.首先先找到连接池中空闲时间最长的连接，以及空闲连接数
	
	2.其次判断当前空闲连接数如果大于默认的连接数，则直接移除当前的空闲连接，关闭连接sokect

	3.如果当前空闲时间大于默认的空闲时间，也直接移除当前的空闲连接，关闭连接sokect

	4.如果当前连接池中存在空闲连接，那么直接返回超时时间差，返回之后，当前线程会休眠当前这个时间差，然后继续下一次清理

	5.如果没有空闲线程，直接返回默认的超时时间，当前清理线程会休眠

	6.如果没有连接，则当前清理线程会退出。

```java
long cleanup(long now) {
    int inUseConnectionCount = 0; //正在使用中连接的个数
    int idleConnectionCount = 0; //空闲连接的个数
    RealConnection longestIdleConnection = null; //空闲时间最长的连接
    long longestIdleDurationNs = Long.MIN_VALUE; //最长的空闲时间

    // Find either a connection to evict, or the time that the next eviction is due.
    synchronized (this) {
      for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
        RealConnection connection = i.next();

        // If the connection is in use, keep searching.
        if (pruneAndGetAllocationCount(connection, now) > 0) {//如果当前线程在使用
          inUseConnectionCount++;
          continue;
        }

        idleConnectionCount++; //走到这里表示已经发现了空闲连接

        // If the connection is ready to be evicted, we're done.
        long idleDurationNs = now - connection.idleAtNanos;
        if (idleDurationNs > longestIdleDurationNs) {
          longestIdleDurationNs = idleDurationNs;
          longestIdleConnection = connection;
        }
      }
	//循环到此处会得到空闲时间最长的连接

      if (longestIdleDurationNs >= this.keepAliveDurationNs //如果空闲时间大于默认的时间 5分钟
          || idleConnectionCount > this.maxIdleConnections) {//空闲连接数大于最大空闲数 5个
        // We've found a connection to evict. Remove it from the list, then close it below (outside
        // of the synchronized block).
        connections.remove(longestIdleConnection);//直接将该连接移出连接池
      } else if (idleConnectionCount > 0) {//如果空闲的个数大于0
        // A connection will be ready to evict soon.
        return keepAliveDurationNs - longestIdleDurationNs; //返回下次清理还需等待的时间
      } else if (inUseConnectionCount > 0) { //如果使用的连接数大于0 直接返回默认的超时时间
        // All connections are in use. It'll be at least the keep alive duration 'til we run again.
        return keepAliveDurationNs;
      } else {//如果没有连接，清理线程将退出执行
        // No connections, idle or in use.
        cleanupRunning = false;
        return -1;
      }
    }

    closeQuietly(longestIdleConnection.socket()); //关闭当前的连接socket

    // Cleanup again immediately.
    return 0;
  }
```

来一张连接池的流程图：**（为什么是5，默认超时时间是 5分钟，空闲链接为5，可以自定义）**
![image.png](/images/2019/03/09/749ee470-4283-11e9-a1b6-4f48496f0a49.png)
![image.png](/images/2019/03/09/7c841250-4283-11e9-a1b6-4f48496f0a49.png)


### okhttp 的Cache设计













<p id="div-blue">
数据结构：private final Deque<RealConnection> connections = new ArrayDeque<>();
</p>
<p id="div-blue">完美的邂逅，这个纯属扯淡</p>

<p id="div-red"><font color="#0000CD">完美的邂逅</font></p>

<p id="div-yellow"> <font color="#0000CD">完美的邂逅</font></p>

<p id="div-green">完美的邂逅</p>

<p id="div-purple">完美的邂逅</p>

<p id="div-green">完美的邂逅</p>

<p id="div-purple">完美的邂逅</p>

<p id="div-green">完美的邂逅</p>

<p id="div-purple">完美的邂逅</p>

<p id="div-green">完美的邂逅</p>

<p id="div-purple">完美的邂逅</p>