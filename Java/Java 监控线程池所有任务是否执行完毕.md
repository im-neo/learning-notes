# Java 监控线程池所有任务是否执行完毕

## 场景引入
> 在最近的工作中遇到一个需要批量生产百万数据并写入数据库的需求，先通过单线程的方式去实现，但是感觉效率一般，然后通过多线程的方式去改进，但是遇到下面的问题：
> 
> 生成数据的时候，每生成一条数据，就将数据丢入到集合中进行缓存，当集合中的数据量达到 500 的时候，就进行一次数据库写入操作，并清空集合。
> 
> 那么当我生成的数据量为 1200 的时候，就会出现最后 200 条没有写入到数据库的问题
> 
> 那么要解决这个问题，就要知道线程池的任务什么时候执行完毕，网上找了很多方法如：
> - 方案一 - 直接调用是否终止的方法
> ```java
> executor.isTerminated()
> ```
> - 方案二 - 判断活跃线程数
> ```java
> executor.getActiveCount() == 0
> ```
> - 方案三 - 设置一个较长的等待时间
> ```java
> executor.awaitTermination(10000, TimeUnit.SECONDS);
> ```
> 
> 但是都没有达到我的目的（也有可能是我使用的姿势不对）
> 
> 最后尝试了下面的方法

## 功能实现
```java
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.collections.CollectionUtils;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;

@Slf4j
public class ThreadPoolTest {

    static List<Integer> TEMP_LIST = new ArrayList<>(10);

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        Random random = new Random();
        int count = random.nextInt(100);
        for (int i = 0; i < count; i++) {
            int finalI = i;
            executorService.execute(() -> {
                addToTemp(finalI * 2, false);
                try {
                    Thread.sleep(random.nextInt(1000 * 2));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }

        // 监控线程执行情况
        monitor((ThreadPoolExecutor) executorService);
        // 最后一次数据插入
        addToTemp(null, true);
        System.out.println("结束");
    }
    
    /**
     * 添加数据到缓存
     *
     * @Author: Neo
     * @Date: 2019/12/19 21:46
     * @Version: 1.0
     */
    public static synchronized void addToTemp(Integer data, boolean focus) {
        if (null != data) {
            TEMP_LIST.add(data);
        }

        if (CollectionUtils.isNotEmpty(TEMP_LIST) && (CollectionUtils.size(TEMP_LIST) >= 10 || focus)) {
            // 模拟数据库写入
            log.info("执行一次插入，数据量：{}", CollectionUtils.size(TEMP_LIST));
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            TEMP_LIST.clear();
        }
    }
    
    /**
     * 监控线程执行情况
     * 一直阻塞到线程任务执行完毕
     *
     * @Author: Neo
     * @Date: 2019/12/19 21:09
     * @Version: 1.0
     */
    public static void monitor(ThreadPoolExecutor executor) {
        
        while (!isTerminated(executor)) {
            try {
                int queueSize = executor.getQueue().size();
                int activeCount = executor.getActiveCount();
                long completedTaskCount = executor.getCompletedTaskCount();
                long taskCount = executor.getTaskCount();

                log.info("当前排队线程数：{}，当前活动线程数：{}，执行完成线程数：{}，总线程数：{}", queueSize, activeCount, completedTaskCount, taskCount);

                Thread.sleep(1000 * 5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 线程池任务是否执行完毕
     *
     * @Author: Neo
     * @Date: 2019/12/19 21:11
     * @Version: 1.0
     */
    public static boolean isTerminated(ThreadPoolExecutor executor) {
        return executor.getQueue().size() == 0 && executor.getActiveCount() == 0;
    }
}
```