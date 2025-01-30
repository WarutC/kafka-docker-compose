# Kafka cluster setup step using docker compose in multiple node

## Execution Workflow

1. On all machines:

    ```bash
    mkdir -p kraft-data kraft-logs config
    ```

1. Change folder owner

    ```bash
    sudo chown -R 1000:1000 kraft-data
    sudo chown -R 1000:1000 kraft-logs
    ```

1. Create JAAS config (config/kafka_server_jaas.conf on all machines):

    ```conf
    KafkaServer {
        org.apache.kafka.common.security.plain.PlainLoginModule required
        username="admin"
        password="admin-secret"
        user_admin="admin-secret"
        user_ccdc="user1-secret";
    };
    ```

1. Init kafka cluster (Node 1 only)

    ```bash
    docker compose -f docker-compose.init.yml up
    ```

1. Get cluster id and edit .env on all nodes

    Get cluster id

    ```bash
    cd kraft_data
    cat kraft-data/meta.properties | grep cluster.id
    ```

    example response

    ```bash
    cluster.id=asdfasfasfasdfsdfasdf
    ```

    edit .env change CLUSTER_ID

    ```bash
    CLUSTER_ID=asdfasfasfasdfsdfasdf
    ```

1. Start kafka1 first:

    ```bash
    docker compose up -d
    ```

1. Start kafka2 and kafka3:

    ```bash
    docker compose up -d
    ```

## Client Connection Example

client.properties:

```java
bootstrap.servers=192.168.182.58:9092,192.168.182.59:9092,192.168.182.60:9092
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAINTEXT
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required
username="user1"
password="user1-secret";
```

Produce message:

```bash
kafka-console-producer.sh \
  --topic test \
  --bootstrap-server 192.168.182.58:9092 \
  --producer.config client.properties
```

## Important Notes

1. Ensure firewall allows traffic between nodes on ports 9092 and 9093

1. All machines must be able to resolve each other's IP addresses

1. For production:

    - Use SSL encryption (SASL_SSL)

    - Set up ACLs for proper authorization

    - Use proper hostnames with DNS resolution

1. The admin credentials in kafka_server_jaas.conf should be changed from defaults
