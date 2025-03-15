## Kafka Configuration 

#### 1. Basic Kafka Configuration without UI 

Use this to setup Kafka stadalone 
```ruby
docker run -d --name broker -p 9092:9092   -e KAFKA_NODE_ID=1   -e KAFKA_PROCESS_ROLES=broker,controller   -e KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093   -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://host.docker.internal:9092   -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER   -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT   -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@localhost:9093   -e KAFKA_LOG_DIRS=/tmp/kraft-combined-logs   -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1   apache/kafka:latest

```
To access the kafka we must log into the container 

```shell
docker exec --workdir /opt/kafka/bin/ -it broker sh
```
To run Kafka commands 
```shell
./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test-topic
```

## JAVA Programming 101 

### 1. Java Producer
A simple java producer that sends data to java producer 
```java
package first;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;
import org.slf4j.LoggerFactory;
import org.slf4j.Logger;

import java.util.Properties;

public class ProducerDemo {

    private static final Logger log = LoggerFactory.getLogger(ProducerDemo.class.getSimpleName());

    public static void main(String[] args) {
        log.info("Hello world !!");

        //Properties Producer Properties
        Properties properties = new Properties();
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "host.docker.internal:9092");
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        //Create producer
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

        //create a procuerRecord =
        ProducerRecord<String, String> producerRecord =
                new ProducerRecord<>("demo_java","Hello world Nice weekend");

        // send the data - asynchronus
        producer.send(producerRecord);

        // flush data
        producer.flush();

        // flush and close producer
        producer.close();
    }
}
```

Since our Kafka is running in a container we must change 

```java
properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
```
we must reference the docker host 

```java
properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "host.docker.internal:9092");
```

---
### 2. JAVA Producer API Callback 
```java
package first;

import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Properties;

public class ProducerDemoWithCallback {

    private static final Logger log = LoggerFactory.getLogger(ProducerDemoWithCallback.class.getSimpleName());

    public static void main(String[] args) {
        log.info("I am a Kafka Producer !!");

        //Properties Producer Properties
        Properties properties = new Properties();
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "host.docker.internal:9092");
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        //Create producer
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

        //create a ProduerRecord =
        ProducerRecord<String, String> producerRecord =
                new ProducerRecord<>("demo_java","I'm i getting anything out of this : (");

        // send the data - asynchronus
        producer.send(producerRecord, new Callback() {
            @Override
            public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                if (e == null){
                    log.info("Received new metadata \n" +
                            "Topic: " + recordMetadata.topic() + "\n" +
                            "Partition: " + recordMetadata.partition() + "\n" +
                            "Offset: " + recordMetadata.offset() + "\n" +
                            "Timestamp: " + recordMetadata.timestamp()
                    );
                }
                else {
                    log.error("Error while producing: ",e);
                }
            }
        });

        // flush data
        producer.flush();

        // flush and close producer
        producer.close();
    }
}
```

This part below is the is an asynchronous Kafka producer callback in Java that handles the result of sending a message to a Kafka topic.
```java
producer.send(producerRecord, new Callback() {
            @Override
            public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                if (e == null){
                    log.info("Received new metadata \n" +
                            "Topic: " + recordMetadata.topic() + "\n" +
                            "Partition: " + recordMetadata.partition() + "\n" +
                            "Offset: " + recordMetadata.offset() + "\n" +
                            "Timestamp: " + recordMetadata.timestamp()
                    );
                }
                else {
                    log.error("Error while producing: ",e);
                }
            }
        });
```
#### Breaking It Down
**1. Asynchronous Sending `(producer.send(producerRecord, callback))`**
- The `send(` method sends a Kafka message `(producerRecord)` asynchronously.
- The second parameter is a Callback, which executes once Kafka processes the message.

**2. Handling Success and Failure (onCompletion)**
- The `onCompletion()` method is called when Kafka acknowledges the message (whether successfully or with an error).
- It has two parameters:
    - `recordMetadata`: Contains metadata about the sent record if it was successful.
    - `e`: Contains an exception if an error occurred.

**3. Success Case `(e == null)`**
- If e is null, the message was successfully sent.
- It logs metadata of the record:
    - `recordMetadata.topic()`: The topic where the message was published.
    - `recordMetadata.partition()`: The partition where the message landed.
    - `recordMetadata.offset()`: The offset of the message in the partition.
    - `recordMetadata.timestamp()`: The timestamp when Kafka stored the message.

**4. Failure Case `(e != null)`**
    - If e is not null, there was an error.
    - It logs the error using `log.error("Error while producing: ", e);`

#### Why Use a Callback?
- Kafka's `send()` method is asynchronous, meaning it does not block execution.
- The callback ensures that we get notified when the message is successfully sent or encounters an error.
- This is useful for error handling, monitoring, and debugging Kafka producers.
- 

### 3. JAVA Producer: JAPA API - with Keys

```java
for (int i=0; i<10; i++){

            String topic = "demo_java";
            String value = "Abhishek you are slow " + i;
            String key = "id_"+i;

            //create a ProduerRecord
            ProducerRecord<String, String> producerRecord =
                    new ProducerRecord<>("demo_java", key, value);

            // send the data - asynchronus
            producer.send(producerRecord, new Callback() {
                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                    if (e == null){
                        log.info("Received new metadata \n" +
                                "Key: " + producerRecord.key() + "\n" +
                                "Partition: " + recordMetadata.partition() + "\n"
                        );
                    }
                    else {
                        log.error("Error while producing: ",e);
                    }
                }
            });
        }
```
We are sending the data to Kafka with keys 
```java
ProducerRecord<String, String> producerRecord =
                    new ProducerRecord<>("demo_java", key, value);
```
---

### 4. JAVA Consumer with Polling

```java
package first;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;

public class ConssumerDemo {

    private static final Logger log = LoggerFactory.getLogger(ConssumerDemo.class.getSimpleName());

    public static void main(String[] args) {

        String bootstrapServer = "host.docker.internal:9092";
        String topic = "demo_java";
        String groupID = "my_group_id";


        // create consumer config
        Properties properties = new Properties();
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServer);
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, groupID);
        properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        // Kafka Consumer
        KafkaConsumer<String, String>  consumer = new KafkaConsumer<>(properties);

        // Subscribe Consumer to out topic(s)
        consumer.subscribe(Arrays.asList(topic));

        // Poll for new data

        while(true){

            log.info("Polling");

            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

            for(ConsumerRecord<String, String> record: records) {
                log.info("Key: " + record.key() + "Value" + record.value());
                log.info("partition: "+ record.partition() + "Offset: " + record.partition());

            }


        }

        }

    }
```


### 5. JAVA Consumer with Graceful Shutdown 

```java
package first;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.errors.WakeupException;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;

public class ConssumerDemoWithShutdown {

    private static final Logger log = LoggerFactory.getLogger(ConssumerDemoWithShutdown.class.getSimpleName());

    public static void main(String[] args) {

        String bootstrapServer = "host.docker.internal:9092";
        String topic = "demo_java";
        String groupID = "my_third_application";


        // create consumer config
        Properties properties = new Properties();
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServer);
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, groupID);
        properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        // Kafka Consumer
        KafkaConsumer<String, String>  consumer = new KafkaConsumer<>(properties);

        //get a reference to the current thread
        final Thread mainThread = Thread.currentThread();

        //adding the shutdown hook
        Runtime.getRuntime().addShutdownHook(new Thread(){
            public void run() {
                log.info("Detected a shutdown, let's exit by calling consumer.wakeup()....");
                consumer.wakeup();

                //join the main thread to allow the execution of the code in the main thread
                try {
                    mainThread.join();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        });

        try {

            //get a reference to the current thread as shutdown hook will run in different thread
            // Subscribe Consumer to out topic(s)
            consumer.subscribe(Arrays.asList(topic));

            // Poll for new data
            while(true){

                log.info("Polling");

                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

                for(ConsumerRecord<String, String> record: records) {
                    log.info("Key: " + record.key() + "Value" + record.value());
                    log.info("partition: "+ record.partition() + "Offset: " + record.partition());

                }
            }
        } catch (WakeupException e) {
            log.info("Wake up exception !");
            //We ignore this as this is an expected exception when closing a consumer
        } catch (Exception e){
            log.error("Unexpected exception");
        } finally {
            consumer.close();
            log.info("The consumer is now gracefully closed");
        }


    }
}
```
This implementation ensures that the Kafka consumer shuts down cleanly when the application is stopped, preventing resource leaks and handling consumer group rebalancing properly.

#### 1. Setup Kafka Consumer & Configuration
```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
```
Initializes a Kafka consumer with the given properties.

#### 2.Get a Reference to the Main Thread
```java
final Thread mainThread = Thread.currentThread();
```
Stores a reference to the main thread so we can safely handle shutdown logic.

#### 3. Add a Shutdown Hook
```java
Runtime.getRuntime().addShutdownHook(new Thread(){
    public void run() {
        log.info("Detected a shutdown, let's exit by calling consumer.wakeup()....");
        consumer.wakeup(); // Interrupts consumer.poll()

        try {
            mainThread.join(); // Ensures main thread finishes execution before exiting
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
});
```
**What is a Shutdown Hook?**
- It's a special thread that runs when the JVM is shutting down (e.g., when the app is stopped using CTRL+C or kill command).
- The shutdown hook calls `consumer.wakeup()`, which interrupts `consumer.poll()`.

**Why Call `consumer.wakeup()`?**
- `consumer.poll()` is a blocking call, so if we don’t wake it up, it might hang indefinitely.
- The WakeupException allows the program to exit gracefully.

#### 4. Start the Consumer Loop
```java
while (true) {
    log.info("Polling");
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

    for (ConsumerRecord<String, String> record : records) {
        log.info("Key: " + record.key() + " Value: " + record.value());
        log.info("Partition: " + record.partition() + " Offset: " + record.offset());
    }
}
```
- This keeps polling for new messages until the shutdown signal is received.
  
#### 5. Catch WakeupException to Stop the Consumer
```java
} catch (WakeupException e) {
    log.info("Wake up exception !");
    // Expected exception when closing consumer, so we ignore it.
} catch (Exception e){
    log.error("Unexpected exception", e);
}
```
- `WakeupException` is thrown when `consumer.wakeup()` is called.
- This is expected behavior, so we catch and ignore it.
- Any other exceptions are logged as errors.

#### 6. Close the Consumer Properly
```java
} finally {
    consumer.close();
    log.info("The consumer is now gracefully closed");
}
```
- `consumer.close()` ensures:
    - The consumer leaves the consumer group properly.
    - Commits offsets (if using auto-commit).
    - Releases resources.



