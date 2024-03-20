`docker-compose.yml` file with kafka ui :
- kafka ui [image: provectuslabs/kafka-ui]
- postgres connector debezium [image: debezium/example-postgres]
- elastic [image: elasticsearch:8.8.0]
- debezium connect [container_name: kafka_connect]
- postgresql admin [image: adminer]
- kafka [image: confluentinc/cp-kafka:7.0.1]
- zookeeper [image: confluentinc/cp-zookeeper:latest]

```
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 52181:2181

  kafka:
    image: confluentinc/cp-kafka:7.0.1
    ports:
      - "9092:9092"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092,PLAINTEXT_INTERNAL://kafka:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1

  kafka-admin:
    image: provectuslabs/kafka-ui
    container_name: kafka-admin
    ports:
      - "9091:8080"
    restart: always
    depends_on:
      - kafka
      - zookeeper
    environment:
      - KAFKA_CLUSTERS_0_NAME=local
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka:29092
      - KAFKA_CLUSTERS_0_ZOOKEEPER=zookeeper:2181

  db:
    platform: linux/x86_64
    image: debezium/example-postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: example
    ports:
      - 5432:5432
    extra_hosts:
      - "host.docker.internal:host-gateway"
    command:
      - "postgres"
      - "-c"
      - "wal_level=logical"
    volumes:
      - ./resource/init.sql:/docker-entrypoint-initdb.d/create-db-tables.sql

  elasticsearch:
    image: elasticsearch:8.8.0
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"

  adminer:
    image: adminer
    restart: always
    ports:
      - 8001:8080

  kafka_connect:
    container_name: kafka_connect
    image: debezium/connect
    links:
      - db
      - kafka
    ports:
      - '8083:8083'
    environment:
      - BOOTSTRAP_SERVERS=kafka:29092
      - GROUP_ID=medium_debezium
      - CONFIG_STORAGE_TOPIC=my_connect_configs
      - OFFSET_STORAGE_TOPIC=my_connect_offsets
      - STATUS_STORAGE_TOPIC=my_connect_statuses
```



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
