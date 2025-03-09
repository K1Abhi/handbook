## Kafka Configuration 



Use this to setup Kafka stadalone 
```ruby
docker run -d --name broker -p 9092:9092   -e KAFKA_NODE_ID=1   -e KAFKA_PROCESS_ROLES=broker,controller   -e KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093   -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://host.docker.internal:9092   -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER   -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT   -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@localhost:9093   -e KAFKA_LOG_DIRS=/tmp/kraft-combined-logs   -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1   apache/kafka:latest

```
Since our Kafka is running in a container we must change 

```java
properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
```
we must reference the docker host 

```java
properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "host.docker.internal:9092");
```
