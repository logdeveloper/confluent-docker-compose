# zookeeper&kafka docker-compose with confluent docker image

### 1. Create user defined bridge network
```bash
docker network create --gateway 172.18.0.1 --subnet 172.18.0.0/16 broker
```


### 2. Check docker-compose.yml

If you set up one zookeeper and multi kafka, you use docker-compose.yml in zoo-one-kafka-multi folder.
Or If you set up multi zookeeper and multi kafka, you use docker-compose.yml in zoo-multi-kafka-multi folder.

Below is an example of multi zookeeper and multi kafka.

```yaml
version: "3.8"
services:
  zoo1:
    image: zookeeper:3.4.9
    restart: always
    hostname: zoo1    
    networks:
      broker-bridge:
        ipv4_address: 172.18.0.11
    ports:
      - "2181:2181"
    environment:
        ZOO_MY_ID: 1
        ZOO_PORT: 2181
        ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
    volumes:
      - /home/broker/zookeeper-kafka-data/zoo1/data:/data
      - /home/broker/zookeeper-kafka-data/zoo1/datalog:/datalog

  zoo2:
    image: zookeeper:3.4.9
    hostname: zoo2
    restart: always
    networks:
      broker-bridge:
        ipv4_address: 172.18.0.12
    ports:
      - "2182:2182"
    environment:
        ZOO_MY_ID: 2
        ZOO_PORT: 2182
        ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
    volumes:
      - /home/broker/zookeeper-kafka-data/zoo2/data:/data
      - /home/broker/zookeeper-kafka-data/zoo2/datalog:/datalog

  zoo3:
    image: zookeeper:3.4.9
    hostname: zoo3
    restart: always
    networks:
      broker-bridge:
        ipv4_address: 172.18.0.13
    ports:
      - "2183:2183"
    environment:
        ZOO_MY_ID: 3
        ZOO_PORT: 2183
        ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
    volumes:
      - /home/broker/zookeeper-kafka-data/zoo3/data:/data
      - /home/broker/zookeeper-kafka-data/zoo3/datalog:/datalog


  
  kafka1:
    image: confluentinc/cp-kafka:5.5.0
    hostname: kafka1
    restart: always
    networks:
      broker-bridge:
        ipv4_address: 172.18.0.21
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: LISTENER_DOCKER_INTERNAL://kafka1:19092,LISTENER_DOCKER_EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_DOCKER_INTERNAL:PLAINTEXT,LISTENER_DOCKER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_DOCKER_INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181,zoo2:2182,zoo3:2183"
      KAFKA_BROKER_ID: 1
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      # replication factor
      KAFKA_DEFAULT_REPLICATION_FACTOR : 2
      # partition
      KAFKA_NUM_PARTITIONS: 3
    volumes:
      - /home/broker/zookeeper-kafka-data/kafka1/data:/var/lib/kafka/data
      # you have to create server.properties file before starting docker compose.
      - /home/broker/conf/server1.properties:/etc/kafka/server.properties      
    depends_on:
      - zoo1
      - zoo2
      - zoo3

  kafka2:
    image: confluentinc/cp-kafka:5.5.0
    hostname: kafka2
    restart: always
    networks:
      broker-bridge:
        ipv4_address: 172.18.0.32
    ports:
      - "9093:9093"
    environment:
      KAFKA_ADVERTISED_LISTENERS: LISTENER_DOCKER_INTERNAL://kafka2:19093,LISTENER_DOCKER_EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_DOCKER_INTERNAL:PLAINTEXT,LISTENER_DOCKER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_DOCKER_INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181,zoo2:2182,zoo3:2183"
      KAFKA_BROKER_ID: 2
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      # replication factor
      KAFKA_DEFAULT_REPLICATION_FACTOR : 2
      # partition
      KAFKA_NUM_PARTITIONS: 3
    volumes:
      - /home/broker/zookeeper-kafka-data/kafka2/data:/var/lib/kafka/data
      # you have to create server.properties file before starting docker compose.
      - /home/broker/conf/server2.properties:/etc/kafka/server.properties      
    depends_on:
      - zoo1
      - zoo2
      - zoo3

  kafka3:
    image: confluentinc/cp-kafka:5.5.0
    hostname: kafka3
    restart: always
    networks:
      broker-bridge:
        ipv4_address: 172.18.0.33
    ports:
      - "9094:9094"
    environment:
      KAFKA_ADVERTISED_LISTENERS: LISTENER_DOCKER_INTERNAL://kafka3:19094,LISTENER_DOCKER_EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9094
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_DOCKER_INTERNAL:PLAINTEXT,LISTENER_DOCKER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_DOCKER_INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181,zoo2:2182,zoo3:2183"
      KAFKA_BROKER_ID: 3
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      # replication factor
      KAFKA_DEFAULT_REPLICATION_FACTOR : 2
      # partition
      KAFKA_NUM_PARTITIONS: 3
    volumes:
      - /home/broker/zookeeper-kafka-data/kafka3/data:/var/lib/kafka/data
      # you have to create server.properties file before starting docker compose.
      - /home/broker/conf/server3.properties:/etc/kafka/server.properties      
    depends_on:
      - zoo1
      - zoo2
      - zoo3

networks:
  broker-bridge:
    external:
      name: broker



```



### 3. Run docker-compose.yml

```
$ docker-compose up -d
```


```
$ log@doople:~/docker-compose$ docker-compose ps
         Name                        Command               State                          Ports
-----------------------------------------------------------------------------------------------------------------------
docker-compose_kafka1_1   /etc/confluent/docker/run        Up      0.0.0.0:9092->9092/tcp
docker-compose_kafka2_1   /etc/confluent/docker/run        Up      9092/tcp, 0.0.0.0:9093->9093/tcp
docker-compose_kafka3_1   /etc/confluent/docker/run        Up      9092/tcp, 0.0.0.0:9094->9094/tcp
docker-compose_zoo1_1     /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2181->2181/tcp, 2888/tcp, 3888/tcp
docker-compose_zoo2_1     /docker-entrypoint.sh zkSe ...   Up      2181/tcp, 0.0.0.0:2182->2182/tcp, 2888/tcp, 3888/tcp
docker-compose_zoo3_1     /docker-entrypoint.sh zkSe ...   Up      2181/tcp, 0.0.0.0:2183->2183/tcp, 2888/tcp, 3888/tcp
```


