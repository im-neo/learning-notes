# 用于验证kafka server Demo

## maven 依赖

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.12</artifactId>
    <version>1.0.0</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.0.0</version>
</dependency>

<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>1.0.0</version>
</dependency>
```

## 生产者

```java
import java.util.Properties;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

public class KafkaProducerTest implements Runnable {

	private final KafkaProducer<String, String> producer;
	private final String topic;
	public KafkaProducerTest(String topicName) {
		Properties props = new Properties();
		props.put("bootstrap.servers", "localhost:9092");
		props.put("acks", "all");
		props.put("retries", 0);
		props.put("batch.size", 16384);
		props.put("key.serializer", StringSerializer.class.getName());
		props.put("value.serializer", StringSerializer.class.getName());
		this.producer = new KafkaProducer<String, String>(props);
		this.topic = topicName;
	}

	@Override
	public void run() {
		int messageNo = 1;
		try {
			for(;;) {
				String messageStr="你好，这是第"+messageNo+"条数据";
				producer.send(new ProducerRecord<String, String>(topic, "Message", messageStr));
				//生产了100条就打印
				if(messageNo%100==0){
					System.out.println("发送的信息:" + messageStr);
				}
				//生产1000条就退出
				if(messageNo%1000==0){
					System.out.println("成功发送了"+messageNo+"条");
					break;
				}
				messageNo++;
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			producer.close();
		}
	}
	
	public static void main(String args[]) {
		KafkaProducerTest test = new KafkaProducerTest("KAFKA_TEST");
		Thread thread = new Thread(test);
		thread.start();
	}
}
```

## 消费者

```java
import java.util.Arrays;
import java.util.Properties;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

public class KafkaConsumerTest implements Runnable {

    private final KafkaConsumer<String, String> consumer;
    private ConsumerRecords<String, String> msgList;
    private final String topic;
    private static final String GROUPID = "groupA";

    public KafkaConsumerTest(String topicName) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", GROUPID);
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("session.timeout.ms", "30000");
        props.put("auto.offset.reset", "earliest");
        props.put("key.deserializer", StringDeserializer.class.getName());
        props.put("value.deserializer", StringDeserializer.class.getName());
        this.consumer = new KafkaConsumer<String, String>(props);
        this.topic = topicName;
        this.consumer.subscribe(Arrays.asList(topic));
    }

    @Override
    public void run() {
        int messageNo = 1;
        System.out.println("---------开始消费---------");
        try {
            for (; ; ) {
                msgList = consumer.poll(1000);
                if (null != msgList && msgList.count() > 0) {
                    for (ConsumerRecord<String, String> record : msgList) {
                        //消费100条就打印 ,但打印的数据不一定是这个规律的
                        if (messageNo % 100 == 0) {
                            System.out.println(messageNo + "=======receive: key = " + record.key() + ", value = " + record.value() + " offset===" + record.offset());
                        }
                        //当消费了1000条就退出
                        if (messageNo % 1000 == 0) {
                            break;
                        }
                        messageNo++;
                    }
                } else {
                    Thread.sleep(1000);
                }
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            consumer.close();
        }
    }

    public static void main(String args[]) {
        KafkaConsumerTest test1 = new KafkaConsumerTest("KAFKA_TEST");
        Thread thread1 = new Thread(test1);
        thread1.start();
    }
}
```

