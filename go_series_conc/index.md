# Go系列：并发


## 系列目录

[《Go系列：内存管理》]()

[《Go系列：调度器》]()

[《Go系列：并发》]()

***

## 1. 介绍





## 2. 面向并发的内存模型

> Go内存模型指定了一种条件，在这种条件下，可以保证在一个goroutine中读取变量可以观察到在不同goroutine中写入同一变量所产生的值。

程序如果修改被多个协程同时访问的数据，那么必须串行化这些访问操作。

为了保证串行化访问，可以使用golang的channel操作或者使用sync和sync/atomic包中的同步原语来保护数据。

### 2.1 Happens Before

在单个goroutine中，读取和写入的行为必须像它们按照程序指定的顺序执行一样。也就是说，仅当重新排序不会改变语言规范所定义的该goroutine中的行为时，编译器和处理器才可以对单个goroutine中执行的读取和写入进行重新排序。由于此重新排序，一个goroutine观察到的执行顺序可能不同于另一个goroutine所察觉到的执行顺序。例如，如果一个goroutine执行a = 1； b = 2；另一个可能会在b的更新值之前观察b的更新值。

为了指定读取和写入的要求，我们定义`happens Before`，Go程序中执行内存操作的部分顺序。如果事件e1`happens before`事件e2，那么我们说e2`happens after`e1。同样的，如果e1不`happens before`e2并且e1也不`happens after`e2，那么我们说e1和e2`happens concurrently`。

在单个goroutine中，事前发生顺序是程序表示的顺序。

如果同时满足以下两个条件，则允许对变量v的读r观察对v的写w：

1. *r* does not happen before *w*.
2. There is no other write *w'* to `v` that happens after *w* but before *r*.

为了保证变量v的读取r观察到对v的特定写入w，请确保w是唯一允许r观察的写入。也就是说，如果同时满足以下两个条件，则保证r遵守w：

1. *w* happens before *r*.
2. Any other write to the shared variable `v` either happens before *w* or after *r*.

这对条件比第一个条件强。它要求没有其他写入与w或r同时发生。

在单个goroutine中，没有并发性，因此这两个定义是等效的：read r观察最近写入w到v的值。当多个goroutine访问共享变量v时，它们必须使用同步事件来建立事件发生-在确保读取遵守期望的写入的条件之前。

用v的类型的零值初始化变量v的行为与在内存模型中的写操作相同。

大于单个机器字的值的读取和写入将以未指定的顺序充当多个机器字大小的操作。

***

### 2.2 同步

#### 2.2.1 Initialization

程序初始化在单个goroutine中运行，但是该goroutine可能会创建其他同时运行的goroutine。

如果包p导入了包q，则完成q的init函数`happen before`p的任何一个的开始。

函数main.main的启动 `happen after`所有init函数均已完成。

#### 2.2.2 Goroutine creation

开始新goroutine的go语句 `happen before`该goroutine的执行开始。

#### 2.2.3 Goroutine destruction

无法保证goroutine的退出 `happen before`	程序中的任何事件

#### 2.2.4 Channel communication

channel通信是goroutine之间同步的主要方法。通常在不同的goroutine中，将特定通道上的每个发送与该通道上的相应接收进行匹配。

一个在channel上的发送`happen before`他从该频道收到的相应信息完成。

channel关闭`happen before`由于通道关闭，接收返回零值的接收

来自无缓冲channel的接收`happen before`他在该channel上的发送完成了。

第k次从初始化空间为C的channel的接收`happens before`第k+C次往channel完成发送。

#### 2.2.5 Locks

sync包实现了两种锁数据类型，sync.Mutex和sync.RWMutex。

对于任何sync.Mutex或sync.RWMutex的锁变量l和两个描述次数的n和m（`n < m`），调用第n次l.Unlock()`happens before`调用第m次l.Lock()的返回。

对于sync.RWMutex变量l的任意调用`l.RLock`，`l.RLock`阻塞直到n次调用`l.UnLock`，并且n次`l.RUnlock` `happens before` 第n+1次调用`l.Lock`。

#### 2.2.6 Once

同步包为使用一次类型的多个goroutines进行初始化提供了一种安全的机制。多个线程可以执行一次。对于一个特定的f执行Do（f），但是只有一个将运行f（），而另一个调用将阻塞直到f（）返回。

单个从once.Do(f)调用f() `happen before`任何调用once.Do(f)的返回

***






