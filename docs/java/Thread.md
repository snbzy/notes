## Reentrantlock
1. 可重入
2. 可打断(interrupt)

## interrupt
1.可打断状态为sleep,wait,join状态的线程，被打断后以异常形式处理，打断标记置为false
2.打断正常运行的线程，并不会立马停止，interrupt方法只会改变打断标记，如何处置需要另行操作
应用：
两阶段终止模式:在线程1中优雅的终止线程2

执行interrupt(),可从isinterrupted()获取打断标记，而inerrupted()获取打断标记后会清除打断标记

线程休眠三种方式:

    1. Thread.sleep
    1. Object.wait()
    2. LockSupport.park(),区别：一个线程只能被打断一次，再次打断无效

不推荐使用的打断方法：

    stop ()
    suspend()
    resume()

## synchronized
    markword最后两位：  
    001-无锁
    101-偏向锁
    00-轻量级锁
    10-重量级锁

偏向锁加锁过程：

    只有第一次CAS将线程ID设置到对象头markword，之后发现这个线程id是自己就没有竞争，无需CAS，若下次有别的线程CAS设置对象头就会变成轻量级锁

轻量级锁加锁过程:

    1. 获取锁对象的markword,如果是轻量级锁的标记(01),则和线程所记录cas,此时对象头就会改变锁标记(00)并记录线程信息
    2. 当有其他线程访问同样会CAS尝试设置所标记，获取锁，如果对象头markword已经是00，则会进行锁膨胀
    1. 当该线程再次访问该锁，CAS会失败但是会进行锁重入(线程会再添加一个锁记录，并记录获取锁的次数)
    2. 线程解锁逆向过程，如果CAS还原对象头失败,则表示已经进行锁膨胀
    3. 轻量级锁膨胀之前会有自旋优化

轻量级锁升级重量级锁：

    1. 新的线程未获取到锁，未对象申请一个monitor，对象头记录monitor地址，markword改成10，新的线程进入entrylist并阻塞
    2. 之前的线程解锁过程：恢复对象头此时会失败(因为markword已经设置成10),这是会找到对象头中monitor地址，设置owner未null
    3. 解锁完毕，monitor从entrylist获取第一个对象，此时新的线程拥有锁
