# Kafka cluster setup step using docker compose in multiple node

## Execution Workflow

1. On all machines:

    ```bash
    mkdir -p kraft-data kraft-logs config
    ```

2. Create JAAS config (config/kafka_server_jaas.conf on all nodes):

    ```conf
    KafkaServer {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="admin"
    password="admin-secret";
    };
    ```

3. Start kafka1 first:

    ```bash
    docker-compose up -d
    ```

4. Get Cluster ID from kafka1:

    ```bash
    docker logs kafka1 | grep "Cluster ID"
    ```

5. Update kafka2 and kafka3:

    Replace random-uuid with get-uuid command in their command sections

6. Start kafka2 and kafka3:

    ```bash
    docker-compose up -d
    ```

## Client Connection Example

client.properties:

```js
bootstrap.servers=192.168.182.58:9092,192.168.182.59:9092,192.168.182.60:9092
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="alice" password="alice-secret";
```

Produce message:

```bash
kafka-console-producer.sh \
  --topic test \
  --bootstrap-server 192.168.182.58:9092 \
  --producer.config client.properties
```

## Docker compose file

### Machine 1 (192.168.182.58) - docker-compose.yml

```bash
services:
  kafka:
    image: apache/kafka:3.6.1
    container_name: kafka1
    hostname: kafka1
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: controller,broker
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@192.168.182.58:9093,2@192.168.182.59:9093,3@192.168.182.60:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,SASL_PLAINTEXT:SASL_PLAINTEXT
      KAFKA_LISTENERS: CONTROLLER://:9093,SASL_PLAINTEXT://:9092
      KAFKA_ADVERTISED_LISTENERS: SASL_PLAINTEXT://192.168.182.58:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_SASL_ENABLED_MECHANISMS: SCRAM-SHA-256
      KAFKA_OPTS: -Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf
    volumes:
      - ./kraft-data:/tmp/kraft-data
      - ./kraft-logs:/tmp/kraft-logs
      - ./config:/etc/kafka
    ports:
      - "9092:9092"
      - "9093:9093"
    command: [
      "sh", "-c",
      "kafka-storage format --ignore-formatted --cluster-id $(kafka-storage random-uuid) --config /etc/kafka/kafka.properties && exec kafka-server-start /etc/kafka/kafka.properties"
    ]
```

### Machine 2 (192.168.182.59) - docker-compose.yml

```bash
services:
  kafka:
    image: apache/kafka:3.6.1
    container_name: kafka2
    hostname: kafka2
    environment:
      KAFKA_NODE_ID: 2
      KAFKA_PROCESS_ROLES: controller,broker
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@192.168.182.58:9093,2@192.168.182.59:9093,3@192.168.182.60:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,SASL_PLAINTEXT:SASL_PLAINTEXT
      KAFKA_LISTENERS: CONTROLLER://:9093,SASL_PLAINTEXT://:9092
      KAFKA_ADVERTISED_LISTENERS: SASL_PLAINTEXT://192.168.182.59:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_SASL_ENABLED_MECHANISMS: SCRAM-SHA-256
      KAFKA_OPTS: -Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf
    volumes:
      - ./kraft-data:/tmp/kraft-data
      - ./kraft-logs:/tmp/kraft-logs
      - ./config:/etc/kafka
    ports:
      - "9092:9092"
      - "9093:9093"
    command: [
      "sh", "-c",
      "kafka-storage format --ignore-formatted --cluster-id $(kafka-storage get-uuid --config /etc/kafka/kafka.properties) --config /etc/kafka/kafka.properties && exec kafka-server-start /etc/kafka/kafka.properties"
    ]
```

### Machine 3 (192.168.182.60) - docker-compose.yml

```bash
services:
  kafka:
    image: apache/kafka:3.6.1
    container_name: kafka3
    hostname: kafka3
    environment:
      KAFKA_NODE_ID: 3
      KAFKA_PROCESS_ROLES: controller,broker
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@192.168.182.58:9093,2@192.168.182.59:9093,3@192.168.182.60:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,SASL_PLAINTEXT:SASL_PLAINTEXT
      KAFKA_LISTENERS: CONTROLLER://:9093,SASL_PLAINTEXT://:9092
      KAFKA_ADVERTISED_LISTENERS: SASL_PLAINTEXT://192.168.182.60:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_SASL_ENABLED_MECHANISMS: SCRAM-SHA-256
      KAFKA_OPTS: -Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf
    volumes:
      - ./kraft-data:/tmp/kraft-data
      - ./kraft-logs:/tmp/kraft-logs
      - ./config:/etc/kafka
    ports:
      - "9092:9092"
      - "9093:9093"
    command: [
      "sh", "-c",
      "kafka-storage format --ignore-formatted --cluster-id $(kafka-storage get-uuid --config /etc/kafka/kafka.properties) --config /etc/kafka/kafka.properties && exec kafka-server-start /etc/kafka/kafka.properties"
    ]
```

## Important Notes

1. Ensure firewall allows traffic between nodes on ports 9092 and 9093

1. All machines must be able to resolve each other's IP addresses

1. For production:

    - Use SSL encryption (SASL_SSL)

    - Set up ACLs for proper authorization

    - Use proper hostnames with DNS resolution

1. The admin credentials in kafka_server_jaas.conf should be changed from defaults
