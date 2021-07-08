# 카프카 클러스터 구성하기

3개의 호스트에 각각 주키퍼 카브카를 도커컴포즈를 사용해서 실행하고 카프카 클러스터를 구축해보자.



## 사전 준비

**kafkacat 설치**

* kafkacat: 브로커와 통신할 때 쓸 수 있는 도구
* kafkacat을 사용해 브로커에게 Metadata를 얻을 것이다.

```bash
# for mac
brew install kafkacat

# using docker
docker run -it confluentinc/cp-kafkacat bash

# Metadata 조회
$ kafkacat -L -b 139.150.75.238:9092
# Metadata 조회(Pretty-printed JSON)
$ kafkacat -b 139.150.75.238:9092 -L -J | jq .
```

참고: https://github.com/edenhill/kafkacat/



## docker-compose 작성

각각 서버마다 docker-compose.yml을 작성한다.

**서버정보**

* 서버1: 139.150.75.238(퍼블릭), 10.7.27.17(프라이빗)
* 서버2: 139.150.75.239(퍼블릭), 10.7.27.18(프라이빗)
* 서버3: 139.150.75.240(퍼블릭), 10.7.27.19(프라이빗)

**카프카 주요 옵션**

`listeners`

> `listeners`는 수신을 위해 Kafka가 바인딩되는 호스트/IP 및 포트의 쉼표로 구분된 목록입니다. 보다 복잡한 네트워킹의 경우 시스템의 지정된 네트워크 인터페이스와 연결된 IP 주소일 수 있습니다. 기본값은 0.0.0.0으로, 모든 인터페이스에서 수신을 의미합니다.

`advertised.listeners`

> `advertised.listeners`는 호스트/IP 및 포트가 있는 listeners의 쉼표로 구분된 목록입니다. 클라이언트에 다시 전달되는 메타데이터입니다.

`inter.broker.listener.name`

> 카프카 브로커들은 보통 내부 네트워크를 통해 서로 통신합니다. `inter.broker.listener.name`은 카프카 브로커가 내부 통신을 위해 사용할 listener를 지정합니다.

**server1.yml**

```yml
version: "3.5"
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:5.5.0
    container_name: zookeeper
    ports:
      - 2181:2181
      - 2888:2888
      - 3888:3888
    networks:
      - kafka-network
    environment:
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 10
      ZOOKEEPER_SYNC_LIMIT: 5
      ZOOKEEPER_SERVERS: "0.0.0.0:2888:3888;52.79.195.122:2888:3888;3.35.141.179:2888:3888"
  kafka:
    image: confluentinc/cp-kafka:5.5.0
    container_name: kafka
    ports:
      - 9092:9092
      - 19092:19092
    networks:
      - kafka-network
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181,52.79.195.122:2181,3.35.141.179:2182
      KAFKA_LISTENERS: INTERNAL://:19092,EXTERNAL://:9092
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://172.31.9.241:19092,EXTERNAL://3.36.125.236:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
networks:
  kafka-network:
    name: kafka-network
```

**server2.yml**

```yml
version: "3.5"
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:5.5.0
    container_name: zookeeper
    ports:
      - 2181:2181
      - 2888:2888
      - 3888:3888
    networks:
      - kafka-network
    environment:
      ZOOKEEPER_SERVER_ID: 2
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 10
      ZOOKEEPER_SYNC_LIMIT: 5
      ZOOKEEPER_SERVERS: "3.36.125.236:2888:3888;0.0.0.0:2888:3888;3.35.141.179:2888:3888"
  kafka:
    image: confluentinc/cp-kafka:5.5.0
    container_name: kafka
    ports:
      - 9092:9092
      - 19092:19092
    networks:
      - kafka-network
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181,3.36.125.236:2181,3.35.141.179:2182
      KAFKA_LISTENERS: INTERNAL://:19092,EXTERNAL://:9092
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://172.31.15.156:19092,EXTERNAL://52.79.195.122:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
networks:
  kafka-network:
    name: kafka-network
```

**server3.yml**

```yml
version: "3.5"
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:5.5.0
    container_name: zookeeper
    ports:
      - 2181:2181
      - 2888:2888
      - 3888:3888
    networks:
      - kafka-network
    environment:
      ZOOKEEPER_SERVER_ID: 3
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 10
      ZOOKEEPER_SYNC_LIMIT: 5
      ZOOKEEPER_SERVERS: "3.36.125.236:2888:3888;52.79.195.122:2888:3888;0.0.0.0:2888:3888"
  kafka:
    image: confluentinc/cp-kafka:5.5.0
    container_name: kafka
    ports:
      - 9092:9092
      - 19092:19092
    networks:
      - kafka-network
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181,3.36.125.236:2181,52.79.195.122:2181
      KAFKA_LISTENERS: INTERNAL://:19092,EXTERNAL://:9092
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://172.31.0.32:19092,EXTERNAL://3.35.141.179:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
networks:
  kafka-network:
    name: kafka-network
```

**카프카 클러스터 실행**

```bash
# 서버1에서 실행
docker-compose -f server1.yml up -d
# 서버2에서 실행
docker-compose -f server2.yml up -d
# 서버3에서 실행
docker-compose -f server3.yml up -d
```

**모든 브로커에게 metadata 받아보기 **

* 아무 브로커에게 metadata를 요구해도 동일한 endpoint를 얻을 수 있다

```bash
docker run -it confluentinc/cp-kafkacat bash

$ kafkacat -L -b 3.36.125.236:9092
Metadata for all topics (from broker 1: 3.36.125.236:9092/1):
 3 brokers:
  broker 2 at 52.79.195.122:9092 (controller)
  broker 3 at 3.35.141.179:9092
  broker 1 at 3.36.125.236:9092
 1 topics:
  topic "__confluent.support.metrics" with 1 partitions:
    partition 0, leader 2, replicas: 2, isrs: 2
    
$ kafkacat -L -b 3.36.125.236:19092
Metadata for all topics (from broker -1: 3.36.125.236:19092/bootstrap):
 3 brokers:
  broker 2 at 172.31.15.156:19092 (controller)
  broker 3 at 172.31.0.32:19092
  broker 1 at 172.31.9.241:19092
 1 topics:
  topic "__confluent.support.metrics" with 1 partitions:
    partition 0, leader 2, replicas: 2, isrs: 2

$ kafkacat -L -b 52.79.195.122:9092
Metadata for all topics (from broker 2: 52.79.195.122:9092/2):
 3 brokers:
  broker 2 at 52.79.195.122:9092 (controller)
  broker 3 at 3.35.141.179:9092
  broker 1 at 3.36.125.236:9092
 1 topics:
  topic "__confluent.support.metrics" with 1 partitions:
    partition 0, leader 2, replicas: 2, isrs: 2

$ kafkacat -L -b 52.79.195.122:19092
Metadata for all topics (from broker -1: 52.79.195.122:19092/bootstrap):
 3 brokers:
  broker 2 at 172.31.15.156:19092 (controller)
  broker 3 at 172.31.0.32:19092
  broker 1 at 172.31.9.241:19092
 1 topics:
  topic "__confluent.support.metrics" with 1 partitions:
    partition 0, leader 2, replicas: 2, isrs: 2

$ kafkacat -L -b 3.35.141.179:9092
Metadata for all topics (from broker 3: 3.35.141.179:9092/3):
 3 brokers:
  broker 2 at 52.79.195.122:9092 (controller)
  broker 3 at 3.35.141.179:9092
  broker 1 at 3.36.125.236:9092
 1 topics:
  topic "__confluent.support.metrics" with 1 partitions:
    partition 0, leader 2, replicas: 2, isrs: 2

$ kafkacat -L -b 3.35.141.179:19092
Metadata for all topics (from broker -1: 3.35.141.179:19092/bootstrap):
 3 brokers:
  broker 2 at 172.31.15.156:19092 (controller)
  broker 3 at 172.31.0.32:19092
  broker 1 at 172.31.9.241:19092
 1 topics:
  topic "__confluent.support.metrics" with 1 partitions:
    partition 0, leader 2, replicas: 2, isrs: 2
```

