# 犹豫模式
Balking（犹豫）模式用在一个线程发现另一个线程或本线程已经做了某一件相同的事，那么本线程就无需再做了，直接结束返回  
例如：  
```java
@Slf4j(topic = "Test")
public class Test {
    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination tpt = new TwoPhaseTermination();
        tpt.start();
        tpt.start();
        tpt.start();
        Thread.sleep(3500);
        log.info("停止监控");
        tpt.stop();
    }
}

@Slf4j
class TwoPhaseTermination {
    // 监控线程
    private Thread monitorThread;
    // 停止标记
    private volatile boolean stop;

    // 判断是否执行过start方法
    private boolean starting = false;

    // 启动监控线程
    public void start() {
        synchronized (this) {
            if (starting) return;
            starting = true;
        }
        monitorThread = new Thread(() -> {
            while (true) {
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
当前端页面多次点击按钮调用start时