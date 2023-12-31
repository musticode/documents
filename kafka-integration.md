
Docker image compose file

```
version: '3.7'
services:

  zookeeper:
    image: confluentinc/cp-zookeeper
    container_name: zookeeper
    restart: always
    ports:
      - 2181:2181
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    container_name: kafka
    image: confluentinc/cp-kafka
    restart: always
    depends_on:
      - zookeeper
    ports:
      - 9092:9092
    environment:
      KAFKA_ZOOKEEPER_CONNECT: "zookeeper:2181"
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: "1"
      KAFKA_DELETE_TOPIC_ENABLE: "true"
      KAFKA_ADVERTISED_HOST_NAME:

  kafdrop:
    image: obsidiandynamics/kafdrop
    container_name: kafdrop
    restart: always
    depends_on:
      - zookeeper
      - kafka
    ports:
      - 9000:9000
    environment:
      KAFKA_BROKERCONNECT: kafka:29092
```
