# 介绍一下AQS
AQS核心思想是，如果被请求的共享资源空闲，那么就将当前请求资源的线程设置为有效的工作线程，将共享资源设置为锁定状态；如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是CLH队列的变体实现的，将暂时获取不到锁的线程加入到队列中。AQS全称为AbstractQueuedSynchronizer，是Java中的一个抽象类。 AQS是一个用于构建锁、同步器、协作工具类的工具类（框架）。CLH：Craig、Landin and Hagersten队列，是单向链表，AQS中的队列是CLH变体的虚拟双向队列（FIFO），AQS是通过将每条请求共享资源的线程封装成一个节点来实现锁的分配。主要原理图如下： 
![1753338302833-16eca38c-4b5b-4e22-8304-96e284e69d26.webp](./AQS | 小林coding.assert/1753338302833-16eca38c-4b5b-4e22-8304-96e284e69d26.webp)

AQS使用一个Volatile的int类型的成员变量来表示同步状态，通过内置的FIFO队列来完成资源获取的排队工作，通过CAS完成对State值的修改。AQS广泛用于控制并发流程的类，如下图：其中`Sync`是这些类中都有的内部类，其结构如下：可以看到：`Sync`是`AQS`的实现。 `AQS`主要完成的任务：

- 同步状态（比如说计数器）的原子性管理；
- 线程的阻塞和解除阻塞；
- 队列的管理。AQS原理
AQS最核心的就是三大部分：

- 状态：state；
- 控制线程抢锁和配合的FIFO队列（双向链表）；
- 期望协作工具类去实现的获取/释放等重要方法（重写）。状态state

- 这里state的具体含义，会根据具体实现类的不同而不同：比如在Semapore里，他表示剩余许可证的数量；在CountDownLatch里，它表示还需要倒数的数量；在ReentrantLock中，state用来表示“锁”的占有情况，包括可重入计数，当state的值为0的时候，标识该Lock不被任何线程所占有。
- state是volatile修饰的，并被并发修改，所以修改state的方法都需要保证线程安全，比如getState、setState以及compareAndSetState操作来读取和更新这个状态。这些方法都依赖于unsafe类。FIFO队列

- 这个队列用来存放“等待的线程，AQS就是“排队管理器”，当多个线程争用同一把锁时，必须有排队机制将那些没能拿到锁的线程串在一起。当锁释放时，锁管理器就会挑选一个合适的线程来占有这个刚刚释放的锁。
- AQS会维护一个等待的线程队列，把线程都放到这个队列里，这个队列是双向链表形式。实现获取/释放等方法

- 这里的获取和释放方法，是利用AQS的协作工具类里最重要的方法，是由协作类自己去实现的，并且含义各不相同；
- 获取方法：获取操作会以来state变量，经常会阻塞（比如获取不到锁的时候）。在Semaphore中，获取就是acquire方法，作用是获取一个许可证； 而在CountDownLatch里面，获取就是await方法，作用是等待，直到倒数结束；
- 释放方法：在Semaphore中，释放就是release方法，作用是释放一个许可证； 在CountDownLatch里面，获取就是countDown方法，作用是将倒数的数减一；
- 需要每个实现类重写tryAcquire和tryRelease等方法。# CAS 和 AQS 有什么关系？CAS 和 AQS 两者的区别：

- CAS 是一种乐观锁机制，它包含三个操作数：内存位置（V）、预期值（A）和新值（B）。CAS 操作的逻辑是，如果内存位置 V 的值等于预期值 A，则将其更新为新值 B，否则不做任何操作。整个过程是原子性的，通常由硬件指令支持，如在现代处理器上，`cmpxchg` 指令可以实现 CAS 操作。
- AQS 是一个用于构建锁和同步器的框架，许多同步器如 `ReentrantLock`、`Semaphore`、`CountDownLatch` 等都是基于 AQS 构建的。AQS 使用一个 `volatile` 的整数变量 `state` 来表示同步状态，通过内置的 `FIFO` 队列来管理等待线程。它提供了一些基本的操作，如 `acquire`（获取资源）和 `release`（释放资源），这些操作会修改 `state` 的值，并根据 `state` 的值来判断线程是否可以获取或释放资源。AQS 的 `acquire` 操作通常会先尝试获取资源，如果失败，线程将被添加到等待队列中，并阻塞等待。`release` 操作会释放资源，并唤醒等待队列中的线程。CAS 和 AQS 两者的联系：

- **CAS 为 AQS 提供原子操作支持**：AQS 内部使用 CAS 操作来更新 `state` 变量，以实现线程安全的状态修改。在 `acquire` 操作中，当线程尝试获取资源时，会使用 CAS 操作尝试将 `state` 从一个值更新为另一个值，如果更新失败，说明资源已被占用，线程会进入等待队列。在 `release` 操作中，当线程释放资源时，也会使用 CAS 操作将 `state` 恢复到相应的值，以保证状态更新的原子性。# 如何用 AQS 实现一个可重入的公平锁？AQS 实现一个可重入的公平锁的详细步骤：

- **继承 AbstractQueuedSynchronizer**：创建一个内部类继承自 `AbstractQueuedSynchronizer`，重写 `tryAcquire`、`tryRelease`、`isHeldExclusively` 等方法，这些方法将用于实现锁的获取、释放和判断锁是否被当前线程持有。
- **实现可重入逻辑**：在 `tryAcquire` 方法中，检查当前线程是否已经持有锁，如果是，则增加锁的持有次数（通过 `state` 变量）；如果不是，尝试使用 CAS操作来获取锁。
- **实现公平性**：在 `tryAcquire` 方法中，按照队列顺序来获取锁，即先检查等待队列中是否有线程在等待，如果有，当前线程必须进入队列等待，而不是直接竞争锁。
- **创建锁的外部类**：创建一个外部类，内部持有 `AbstractQueuedSynchronizer` 的子类对象，并提供 `lock` 和 `unlock` 方法，这些方法将调用 `AbstractQueuedSynchronizer` 子类中的方法。
```java
import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;

public class FairReentrantLock {
    private static class Sync extends AbstractQueuedSynchronizer {
        // 判断锁是否被当前线程持有
        @Override
        protected boolean isHeldExclusively() {
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        // 尝试获取锁
        @Override
        protected boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                // 公平性检查：检查队列中是否有前驱节点，如果有，则当前线程不能获取锁
                if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            } else if (current == getExclusiveOwnerThread()) {
                // 可重入逻辑：如果是当前线程持有锁，则增加持有次数
                int nextc = c + acquires;
                if (nextc < 0) {
                    throw new Error("Maximum lock count exceeded");
                }
                setState(nextc);
                return true;
            }
            return false;
        }

        // 尝试释放锁
        @Override
        protected boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread()) {
                throw new IllegalMonitorStateException();
            }
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

        // 提供一个条件变量，用于实现更复杂的同步需求，这里只是简单实现
        Condition newCondition() {
            return new ConditionObject();
        }
    }

    private final Sync sync = new Sync();

    // 加锁方法
    public void lock() {
        sync.acquire(1);
    }

    // 解锁方法
    public void unlock() {
        sync.release(1);
    }

    // 判断当前线程是否持有锁
    public boolean isLocked() {
        return sync.isHeldExclusively();
    }

    // 提供一个条件变量，用于实现更复杂的同步需求，这里只是简单实现
    public Condition newCondition() {
        return sync.newCondition();
    }
}

```
代码解释：内部类 Sync

- `isHeldExclusively`：使用 `getExclusiveOwnerThread` 方法检查当前锁是否被当前线程持有。
- `tryAcquire`：
- 首先获取当前锁的状态 `c`。
- 如果 `c` 为 0，表示锁未被持有，此时进行公平性检查，通过 `hasQueuedPredecessors` 检查是否有前驱节点在等待队列中。如果没有，使用 `compareAndSetState` 尝试将状态设置为 `acquires`（通常为 1），并设置当前线程为锁的持有线程。
- 如果 `c` 不为 0，说明锁已被持有，检查是否为当前线程持有。如果是，增加锁的持有次数（可重入），但要防止溢出。
- `tryRelease`：
- 先将状态减 `releases`（通常为 1）。
- 检查当前线程是否为锁的持有线程，如果不是，抛出异常。
- 如果状态减为 0，说明锁被完全释放，将持有线程设为 `null`。
- `newCondition`：创建一个 `ConditionObject` 用于更复杂的同步操作，如等待 / 通知机制。外部类 FairReentrantLock

- `lock` 方法：调用 `sync.acquire(1)` 尝试获取锁。
- `unlock` 方法：调用 `sync.release(1)` 释放锁。
- `isLocked` 方法：调用 `sync.isHeldExclusively` 判断锁是否被当前线程持有。
- `newCondition` 方法：调用 `sync.newCondition` 提供条件变量。> 来自: [Java并发编程面试题 | 小林coding](https://xiaolincoding.com/interview/juc.html#jvm%E5%AF%B9synchornized%E7%9A%84%E4%BC%98%E5%8C%96)

​
​
