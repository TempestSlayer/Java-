# 终止模式-两阶段终止
在一个线程t1中如何“优雅”终止线程t2？这里的“优雅”指的是给t2一个料理后事的机会

## 错误思路
- 使用线程对象的stop()方法停止线程
    - stop方法会真正杀死线程，如果这时线程锁住了共享资源，那么当它被杀死后就再也没有机会释放锁，其他线程将永远无法获取锁
- 使用System.exit(int)方法停止线程
    - 目的仅是停止一个线程，但这种做法会让整个程序都停止

## 利用interrupt
```java
@Slf4j
public class TwoPhaseTermination {
    // 监控线程
    private Thread monitorThread;

    // 启动监控线程
    public void start() {
        monitorThread = new Thread(()->{
            while (true) {
                Thread current = Thread.currentThread();
                // 是否被打断
                if (current.isInterrupted()) {
                    log.info("料理后事");
                    break;
                }
                try {
                  Thread.sleep(1000);
                  log.info("指向监控记录");
                }  catch (InterruptedException ex) {
                    // 因为sleep出现异常后，会清除打断标记
                    // 需要重置打断标记
                    current.interrupt();
                }
            }
        },"monitor");
        monitorThread.start();
    }

    // 停止监控线程
    public void stop() {
        monitorThread.interrupt();
    }
}
```

## 利用volatile优化
```java
@Slf4j(topic = "Test")
public class Test {
    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination twoPhaseTermination = new TwoPhaseTermination();
        twoPhaseTermination.start();
        Thread.sleep(3500);
        log.info("停止监控");
        twoPhaseTermination.stop();
    }
}

@Slf4j
class TwoPhaseTermination {
    // 监控线程
    private Thread monitorThread;
    //
    private volatile boolean stop;

    // 启动监控线程
    public void start() {
        monitorThread = new Thread(() -> {
            while (true) {
                Thread current = Thread.currentThread();
                // 是否被打断
                if (stop) {
                    log.info("料理后事");
                    break;
                }
                try {
                    Thread.sleep(1000);
                    log.info("执行监控记录");
                } catch (InterruptedException ex) {
                    log.info("打断");
                }
            }
        }, "monitor");
        monitorThread.start();
    }

    // 停止监控线程
    public void stop() {
        stop = true;
        monitorThread.interrupt();
    }
}
```