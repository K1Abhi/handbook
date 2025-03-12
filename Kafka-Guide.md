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
