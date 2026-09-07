JMM 是一种规范，是解决由于多线程通过共享内存进行通信时，存在的本地内存数据不一致、编译器会对代码指令重排序、处理器会对代码乱序执行等带来的问题。目的是保证并发编程场景中的原子性、可见性和有序性。


1. **程序次序规则**：一个线程内，代码书写顺序，前面操作 happen-before 后面。
2. **volatile 变量规则**：对 volatile 变量的写，happen-before 后续任意线程对这个 volatile 变量的读。
3. **锁规则（monitor 锁）**：解锁操作 happen-before 后续对同一锁的加锁。
4. **线程 start 规则**：Thread.start () happen-before 这个线程里面所有操作。
5. **线程 join 规则**：线程内所有操作 happen-before 其他线程的 thread.join () 返回。
6. **线程中断规则**：调用 interrupt () happen-before 中断线程检测到中断信号。
7. **对象终结规则**：对象构造方法执行完成 happen-before finalize 开始执行。
8. **传递性**：A HB B，B HB C → A HB C。



> 一句话前置：


 - **原子性：一次不可分割，要么全部执行成功，要么失败回滚；**
-  **可见性：一个线程修改变量，其他线程立刻能看到最新值；**
- **有序性：禁止CPU/JVM指令重排。**
  

## 1. 原子性

**保证手段：**
1. `synchronized`：锁，保证代码块内操作原子性。
2. `java.util.concurrent.atomic` 原子类（AtomicInteger等，CAS）。

> ❗volatile **不能保证原子性**，i++这种复合操作不行。


原理：
- synchronized：同一时间只有一个线程执行同步块；
- Atomic系列：CAS（Compare And Swap）+ volatile，硬件指令保证。

  

## 2. 可见性

**保证手段：**
1. `volatile`：变量写后刷新到主存，读的时候从主存加载，绕过CPU缓存。
2. `synchronized`：解锁前把变量刷新回主存；加锁时从主存读取。
3. `final`：final变量初始化完成后，其他线程可见。
## 3. 有序性

**保证手段：**
1. `volatile`：禁止volatile变量前后指令重排（内存屏障）。
2. `synchronized`：锁保证同一时刻单线程执行，自然规避重排带来的问题。
3. JMM的happen-before规则，约束指令不能随意重排。


## 高频追问

### 追问1：synchronized 和 volatile 对比？

回答：
- volatile：保证可见性、有序性，**不保证原子性**；无锁，不会阻塞。
- synchronized：原子性、可见性、有序性三者全部保证；会阻塞，有锁开销。

  

### 追问2：CAS有什么问题？

回答：
1. ABA问题；
2. 自旋消耗CPU；
3. 只能保证一个变量原子操作。