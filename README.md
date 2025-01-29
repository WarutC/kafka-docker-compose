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

1. Create JAAS config (config/kafka_server_jaas.conf on all nodes):

    ```conf
    KafkaServer {
        org.apache.kafka.common.security.scram.ScramLoginModule required
        username="admin"
        password="admin-secret";
    };
    ```

1. Start kafka1 first:

    ```bash
    docker compose up -d
    ```

1. Get Cluster ID from kafka1:

    ```bash
    docker logs kafka1 | grep "Cluster ID"
    ```

1. Update kafka2 and kafka3:

    Replace random-uuid with get-uuid command in their command sections

1. Start kafka2 and kafka3:

    ```bash
    docker compose up -d
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

## Important Notes

1. Ensure firewall allows traffic between nodes on ports 9092 and 9093

1. All machines must be able to resolve each other's IP addresses

1. For production:

    - Use SSL encryption (SASL_SSL)

    - Set up ACLs for proper authorization

    - Use proper hostnames with DNS resolution

1. The admin credentials in kafka_server_jaas.conf should be changed from defaults
