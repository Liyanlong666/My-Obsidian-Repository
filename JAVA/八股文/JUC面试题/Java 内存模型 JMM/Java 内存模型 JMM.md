## [Java 内存模型](https://javabetter.cn/sidebar/sanfene/javathread.html#java-%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B)
### [20.说一下你对 Java 内存模型的理解？](https://javabetter.cn/sidebar/sanfene/javathread.html#_20-%E8%AF%B4%E4%B8%80%E4%B8%8B%E4%BD%A0%E5%AF%B9-java-%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B%E7%9A%84%E7%90%86%E8%A7%A3)
推荐阅读：[说说 Java 的内存模型](https://javabetter.cn/thread/jmm.html)
Java 内存模型（JMM）是一个抽象模型，主要用来定义多线程中变量的访问规则，可以解决变量的可见性、有序性和原子性问题，确保在并发环境中安全地访问共享变量。
![image.jpg](./Java 内存模型 JMM.assert/1738590537895-5b45a4ae-8917-48c9-8ad3-60c385ea0b13.jpeg)

深入浅出 Java 多线程：Java内存模型
线程之间的共享变量存储在`主内存`中，每个线程都有一个私有的`本地内存`，本地内存中存储了共享变量的副本，用来进行线程内部的读写操作。

- 当一个线程更改了本地内存中共享变量的副本后，它需要将这些更改刷新到主内存中，以确保其他线程可以看到这些更改。
- 当一个线程需要读取共享变量时，它可能首先从本地内存中读取。如果本地内存中的副本是过时的，线程将从主内存中重新加载共享变量的最新值到本地内存中。本地内存是 JMM 中的一个抽象概念，并不真实存在。实际上，本地内存可能对应于 CPU 缓存、寄存器或者其他硬件和编译器优化。
![image.jpg](./Java 内存模型 JMM.assert/1738590538466-c83acbce-b311-492f-977e-08b3dc077e0d.png)

三分恶面渣逆袭：实际线程工作模型
对于一个双核 CPU 的系统架构，每个核都有自己的控制器和运算器，其中控制器包含一组寄存器和操作控制器，运算器执行算术逻辅运算。
每个核都有自己的一级缓存，在有些架构里面还有一个所有 CPU 共享的二级缓存。
Java 内存模型里面的本地内存，可能对应的是 L1 缓存或者 L2 缓存或者 CPU 寄存器。
#### [为什么线程要用自己的内存？](https://javabetter.cn/sidebar/sanfene/javathread.html#%E4%B8%BA%E4%BB%80%E4%B9%88%E7%BA%BF%E7%A8%8B%E8%A6%81%E7%94%A8%E8%87%AA%E5%B7%B1%E7%9A%84%E5%86%85%E5%AD%98)
第一，在多线程环境中，如果所有线程都直接操作主内存中的共享变量，会引发更多的内存访问竞争，这不仅影响性能，还增加了线程安全问题的复杂度。通过让每个线程使用本地内存，可以减少对主内存的直接访问和竞争，从而提高程序的并发性能。
第二，现代 CPU 为了优化执行效率，可能会对指令进行乱序执行（指令重排序）。使用本地内存（CPU 缓存和寄存器）可以在不影响最终执行结果的前提下，使得 CPU 有更大的自由度来乱序执行指令，从而提高执行效率。
### [21.说说你对原子性、可见性、有序性的理解？](https://javabetter.cn/sidebar/sanfene/javathread.html#_21-%E8%AF%B4%E8%AF%B4%E4%BD%A0%E5%AF%B9%E5%8E%9F%E5%AD%90%E6%80%A7%E3%80%81%E5%8F%AF%E8%A7%81%E6%80%A7%E3%80%81%E6%9C%89%E5%BA%8F%E6%80%A7%E7%9A%84%E7%90%86%E8%A7%A3)

- **原子性**：指的是一个操作是不可分割的，要么全部执行成功，要么完全不执行。
- **可见性**：指的是一个线程对共享变量的修改，能够被其他线程及时看见。
- **有序性**：指的是程序代码的执行顺序与代码中的顺序一致。在没有同步机制的情况下，编译器可能会对指令进行重排序，以优化性能。这种重排序可能会导致多线程的执行结果与预期不符。#### [分析下面几行代码的原子性？](https://javabetter.cn/sidebar/sanfene/javathread.html#%E5%88%86%E6%9E%90%E4%B8%8B%E9%9D%A2%E5%87%A0%E8%A1%8C%E4%BB%A3%E7%A0%81%E7%9A%84%E5%8E%9F%E5%AD%90%E6%80%A7)

```java
int i = 2;
int j = i;
i++;
i = i + 1;
```

- 第 1 句是基本类型赋值，是原子性操作。
- 第 2 句先读 i 的值，再赋值到 j，两步操作，不能保证原子性。
- 第 3 和第 4 句其实是等效的，先读取 i 的值，再+1，最后赋值到 i，三步操作了，不能保证原子性。#### [原子性、可见性、有序性都应该怎么保证呢？](https://javabetter.cn/sidebar/sanfene/javathread.html#%E5%8E%9F%E5%AD%90%E6%80%A7%E3%80%81%E5%8F%AF%E8%A7%81%E6%80%A7%E3%80%81%E6%9C%89%E5%BA%8F%E6%80%A7%E9%83%BD%E5%BA%94%E8%AF%A5%E6%80%8E%E4%B9%88%E4%BF%9D%E8%AF%81%E5%91%A2)

- 原子性：JMM 只能保证基本的原子性，如果要保证一个代码块的原子性，需要使用`synchronized `。
- 可见性：Java 是利用`volatile`关键字来保证可见性的，除此之外，`final`和`synchronized`也能保证可见性。
- 有序性：`synchronized`或者`volatile`都可以保证多线程之间操作的有序性。#### [i++是原子操作吗？](https://javabetter.cn/sidebar/sanfene/javathread.html#i-%E6%98%AF%E5%8E%9F%E5%AD%90%E6%93%8D%E4%BD%9C%E5%90%97)
i++ 不是一个原子操作，它包括三个步骤：

1. 从内存中读取 i 的值。
2. 对 i 进行加 1 操作。
3. 将新的值写入内存。假如两个线程同时对 i 进行 i++ 操作时，可能会发生以下情况：

1. 线程 A 读取 i 的值（假设 i 的初始值为 1）。
2. 线程 B 也读取 i 的值（值仍然是 1）。
3. 线程 A 将 i 增加到 2，并将其写回内存。
4. 线程 B 也将 i 增加到 2，并将其写回内存。尽管进行了两次递增操作，i 的值只增加了 1 而不是 2。可以使用 synchronized 或 AtomicInteger 确保操作的原子性。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的京东同学 4 云实习面试原题：i++是原子操作吗
### [22.那说说什么是指令重排？](https://javabetter.cn/sidebar/sanfene/javathread.html#_22-%E9%82%A3%E8%AF%B4%E8%AF%B4%E4%BB%80%E4%B9%88%E6%98%AF%E6%8C%87%E4%BB%A4%E9%87%8D%E6%8E%92)
在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序。重排序分 3 种类型。

1. 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
2. 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism，ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应 机器指令的执行顺序。
3. 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。从 Java 源代码到最终实际执行的指令序列，会分别经历下面 3 种重排序，如图：
![image.jpg](./Java 内存模型 JMM.assert/1738590538240-24d48ca2-b0f0-4a25-902c-cc1014253103.png)

多级指令重排
我们比较熟悉的双重校验单例模式就是一个经典的指令重排的例子，`Singleton instance=new Singleton()；`对应的 JVM 指令分为三步：分配内存空间-->初始化对象--->对象指向分配的内存空间，但是经过了编译器的指令重排序，第二步和第三步就可能会重排序。
![image.jpg](./Java 内存模型 JMM.assert/1738590538403-aa058cdc-d496-48e4-adb6-c51e05af0ea3.png)

双重校验单例模式异常情形
JMM 属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。
### [23.指令重排有限制吗？happens-before 了解吗？](https://javabetter.cn/sidebar/sanfene/javathread.html#_23-%E6%8C%87%E4%BB%A4%E9%87%8D%E6%8E%92%E6%9C%89%E9%99%90%E5%88%B6%E5%90%97-happens-before-%E4%BA%86%E8%A7%A3%E5%90%97)
指令重排也是有一些限制的，有两个规则`happens-before`和`as-if-serial`来约束。
happens-before 的定义：

- 如果一个操作 happens-before 另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
- 两个操作之间存在 happens-before 关系，并不意味着 Java 平台的具体实现必须要按照 happens-before 关系指定的顺序来执行。如果重排序之后的执行结果，与按 happens-before 关系来执行的结果一致，那么这种重排序并不非法happens-before 和我们息息相关的有六大规则：
![image.jpg](./Java 内存模型 JMM.assert/1738590538218-2bba4e07-6e0d-4984-ad6e-fe78414cc6ff.png)

happens-before六大规则

- **程序顺序规则**：一个线程中的每个操作，happens-before 于该线程中的任意后续操作。
- **监视器锁规则**：对一个锁的解锁，happens-before 于随后对这个锁的加锁。
- **volatile 变量规则**：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读。
- **传递性**：如果 A happens-before B，且 B happens-before C，那么 A happens-before C。
- **start()规则**：如果线程 A 执行操作 ThreadB.start()（启动线程 B），那么 A 线程的 ThreadB.start()操作 happens-before 于线程 B 中的任意操作。
- **join()规则**：如果线程 A 执行操作 ThreadB.join()并成功返回，那么线程 B 中的任意操作 happens-before 于线程 A 从 ThreadB.join()操作成功返回。### [24.as-if-serial 又是什么？单线程的程序一定是顺序的吗？](https://javabetter.cn/sidebar/sanfene/javathread.html#_24-as-if-serial-%E5%8F%88%E6%98%AF%E4%BB%80%E4%B9%88-%E5%8D%95%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%A8%8B%E5%BA%8F%E4%B8%80%E5%AE%9A%E6%98%AF%E9%A1%BA%E5%BA%8F%E7%9A%84%E5%90%97)
as-if-serial 语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），**单线程程序的执行结果不能被改变**。编译器、runtime 和处理器都必须遵守 as-if-serial 语义。
为了遵守 as-if-serial 语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序。为了具体说明，请看下面计算圆面积的代码示例。

```java
double pi = 3.14;   // A
double r = 1.0;   // B
double area = pi * r * r;   // C
```
上面 3 个操作的数据依赖关系：
![image.jpg](./Java 内存模型 JMM.assert/1738590538161-394fe32e-1b3d-4054-a5c1-fca495eb6abd.png)

A 和 C 之间存在数据依赖关系，同时 B 和 C 之间也存在数据依赖关系。因此在最终执行的指令序列中，C 不能被重排序到 A 和 B 的前面（C 排到 A 和 B 的前面，程序的结果将会被改变）。但 A 和 B 之间没有数据依赖关系，编译器和处理器可以重排序 A 和 B 之间的执行顺序。
所以最终，程序可能会有两种执行顺序：
![image.jpg](./Java 内存模型 JMM.assert/1738590538456-5bd7e3db-f63d-44f2-9105-4e11d8559ca7.png)

两种执行结果
as-if-serial 语义把单线程程序保护了起来，遵守 as-if-serial 语义的编译器、runtime 和处理器共同编织了这么一个“楚门的世界”：单线程程序是按程序的“顺序”来执行的。as- if-serial 语义使单线程情况下，我们不需要担心重排序的问题，可见性的问题。
### [25.volatile 了解吗？](https://javabetter.cn/sidebar/sanfene/javathread.html#_25-volatile-%E4%BA%86%E8%A7%A3%E5%90%97)
推荐阅读：[volatile 关键字解析](https://javabetter.cn/thread/volatile.html)
volatile 关键字主要有两个作用，一个是保证变量的内存可见性，一个是禁止指令重排序。它确保一个线程对变量的修改对其他线程立即可见，同时防止代码执行顺序被编译器或 CPU 优化重排。
#### [volatile 怎么保证可见性的呢？](https://javabetter.cn/sidebar/sanfene/javathread.html#volatile-%E6%80%8E%E4%B9%88%E4%BF%9D%E8%AF%81%E5%8F%AF%E8%A7%81%E6%80%A7%E7%9A%84%E5%91%A2)
当一个变量被声明为 volatile 时，Java 内存模型会确保所有线程看到该变量时的值是一致的。
![image.jpg](./Java 内存模型 JMM.assert/1738590538639-cdba646b-279f-426a-b0b6-778802b68eb6.jpeg)

深入浅出 Java 多线程：Java内存模型
当线程对 volatile 变量进行写操作时，JMM 会在写入这个变量之后插入一个写屏障指令，这个指令会强制将本地内存中的变量值刷新到主内存中。
![image.jpg](./Java 内存模型 JMM.assert/1738590538653-b832fb65-20e8-48be-9357-6d832d7f57d7.png)

三分恶面渣逆袭：volatile写插入内存屏障后生成的指令序列示意图
在 x86 架构下，volatile 写操作会插入一个 lock 前缀指令，这个指令会将缓存行的数据写回到主内存中，确保内存可见性。

```java
mov [a], 2          ; 将值 2 写入内存地址 a
lock add [a], 0     ; lock 指令充当写屏障，确保内存可见性
```
当线程对 volatile 变量进行读操作时，JMM 会插入一个 读屏障指令，这个指令会强制让本地内存中的变量值失效，从而重新从主内存中读取最新的值。
![image.jpg](./Java 内存模型 JMM.assert/1738590538698-d2439c56-aee0-48f1-855d-b8aef9e2f4dd.png)

三分恶面渣逆袭：volatile写插入内存屏障后生成的指令序列示意图
例如，我们声明一个 volatile 变量 x：
线程 A 对 x 写入后会将其最新的值刷新到主内存中，线程 B 读取 x 时由于本地内存中的 x 失效了，就会从主内存中读取最新的值，内存可见性达成！
![image.jpg](./Java 内存模型 JMM.assert/1738590538697-180e5ffe-18ec-4646-8d62-1b253c245ed4.png)

三分恶面渣逆袭：volatile内存可见性
#### [volatile 怎么保证有序性的呢？](https://javabetter.cn/sidebar/sanfene/javathread.html#volatile-%E6%80%8E%E4%B9%88%E4%BF%9D%E8%AF%81%E6%9C%89%E5%BA%8F%E6%80%A7%E7%9A%84%E5%91%A2)
在程序执行期间，为了提高性能，编译器和处理器会对指令进行重排序。但涉及到 volatile 变量时，它们必须遵循一定的规则：

- 写 volatile 变量的操作之前的操作不会被编译器重排序到写操作之后。
- 读 volatile 变量的操作之后的操作不会被编译器重排序到读操作之前。这意味着 volatile 变量的写操作总是发生在任何后续读操作之前。
#### [volatile 和 synchronized 的区别](https://javabetter.cn/sidebar/sanfene/javathread.html#volatile-%E5%92%8C-synchronized-%E7%9A%84%E5%8C%BA%E5%88%AB)
volatile 关键字用于修饰变量，确保该变量的更新操作对所有线程是可见的，即一旦某个线程修改了 volatile 变量，其他线程会立即看到最新的值。
synchronized 关键字用于修饰方法或代码块，确保同一时刻只有一个线程能够执行该方法或代码块，从而实现互斥访问。
#### [volatile 加在基本类型和对象上的区别？](https://javabetter.cn/sidebar/sanfene/javathread.html#volatile-%E5%8A%A0%E5%9C%A8%E5%9F%BA%E6%9C%AC%E7%B1%BB%E5%9E%8B%E5%92%8C%E5%AF%B9%E8%B1%A1%E4%B8%8A%E7%9A%84%E5%8C%BA%E5%88%AB)
当 `volatile` 用于基本数据类型时，能确保该变量的读写操作是直接从主内存中读取或写入的。

```java
private volatile int count = 0;
```
当 `volatile` 用于引用类型时，它确保引用本身的可见性，即确保引用指向的对象地址是最新的。
但是，`volatile` 并不能保证引用对象内部状态的线程安全性。

```java
private volatile SomeObject obj = new SomeObject();
```
虽然 `volatile` 确保了 `obj` 引用的可见性，但对 `obj` 引用的具体对象的操作并不受 `volatile` 保护。如果需要保证引用对象内部状态的线程安全，需要使用其他同步机制（如 `synchronized` 或 `ReentrantLock`）。
