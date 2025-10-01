# Pre-req
1. Install Clickhouse http sink connector drivers for kafka-connect:


Can reference the guide [here](https://clickhouse.com/docs/integrations/kafka/clickhouse-kafka-connect-sink)

# Clickhouse K8s Installation

1. Installing the operator:

For connected setup:
```bash
curl -s https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/operator-web-installer/clickhouse-operator-install.sh | OPERATOR_NAMESPACE=clickhouse-operator bash
```

For disconnected setup:
```bash
oc apply --namespace="starlake-clickhouse" -f <( cat "./clickhouse-operator-install-template.yaml" | \
    OPERATOR_IMAGE="altinity/clickhouse-operator:latest" OPERATOR_NAMESPACE="starlake-clickhouse" METRICS_EXPORTER_IMAGE="altinity/metrics-exporter:latest" METRICS_EXPORTER_NAMESPACE="starlake-clickhouse" envsubst \
)
```

2. Installing a single-instance ClickHouse:

```bash
kubectl apply -f clickhouse.yaml -n clickhouse-operator
```

3. Install Cloudbeaver as a client for Admin:

```bash
helm install cloudbeaver ./cloudbeaver-minimal -n clickhouse-operator
```

4. Setup Clickhouse on Cloudbeaver with the following creds:

`admin/plschangeme`

5. Ensure Kafka Connect has the [clickhouse JDBC Connector](https://github.com/ClickHouse/clickhouse-java/releases). Copy the `.jar` into:
`/share/confluent-hub-components/confluentinc-kafka-connect-jdbc/lib/`.

5. Run `setup.ipynb` cell-by-cell to ingest and sink data into Clickhouse and PostgreSQL DB.

6. Run `benchmark.ipynb` cell-by-cell to run a few queries.

# FAQ

## ClickHouse Cluster Setup & Management
Q: What is the minimal viable configuration for a production-ready ClickHouse cluster?
A: For a production-ready MVP, a single shard with two or more replicas is the minimum. This provides high availability and data redundancy. You must also include a persistent storage class for both ClickHouse and the coordination service to ensure data durability.

Q: Do I need a coordination service like ZooKeeper or ClickHouse Keeper?
A: Yes, a coordination service is mandatory for a replicated setup. It ensures data is synchronized between replicas and manages distributed DDL operations. It is recommended to use ClickHouse Keeper with the Altinity Operator, as it is a native, C++-based, and more resource-efficient alternative to ZooKeeper.

Q: How do I configure ClickHouse Keeper for a production environment?
A: You should configure an odd number of replicas (e.g., 3 or 5) for your Keeper to maintain a quorum. You must also use persistent storage for the Keeper's data to prevent state loss on pod restarts. The following YAML snippet shows a corrected configuration:

```YAML

spec:
  configuration:
    keeper:
      sentinel: true
      replicasCount: 3
      storage:
        type: persistentVolumeClaim
        name: keeper-storage-template
```
## User and Password Management

Q: How do I correctly set up users and passwords for a cluster managed by the Altinity Operator?
A: You must define all users and their passwords directly in your ClickHouseInstallation manifest. This is the only reliable way to manage credentials. The operator will then securely generate and apply the configuration to the pods.

Q: How do I ensure my passwords are not stored in plain text in the manifest?
A: You can use a Kubernetes Secret to store the hashed password. The ClickHouseInstallation manifest can then reference this secret using secretKeyRef to pull the password dynamically. This is a best practice for production environments.

```YAML
users:
  admin:
    password_sha256_hex:
      valueFrom:
        secretKeyRef:
          name: clickhouse-credentials
          key: admin
```
Q: Why do my logins fail even when the password in my manifest is correct?
A: This is most often due to network access restrictions or a failed operator reconciliation. Ensure that the user's networks configuration allows connections from your IP address or use a CIDR block like `0.0.0.0/0` for broad access in a testing environment. If you've just updated the password, a small change to the manifest (like adding a force-reconciliation annotation) can force the operator to re-apply the changes.

## Data Handling and Performance

Engine Family|Primary Purpose|Key Examples|Core Concept
|------------|-----------|-----------|---------|
MergeTree|Primary storage for high-performance OLAP.|MergeTree, ReplicatedMergeTree, SummingMergeTree|A columnar, compressed, and indexed data-storage engine that is scalable and fault-tolerant.
Log|Simple, non-indexed tables for small datasets.|TinyLog, Log, StripeLog|Lightweight engines that sequentially store data without complex features like merging or replication.
Distributed|Virtual table for sharding and query routing.|Distributed|A virtual engine that doesn't store data. It acts as a single view over tables on multiple servers.
Integration|Connecting to external data sources.|Kafka, JDBC, MySQL, PostgreSQL|Engines that read data from external systems as if they were native ClickHouse tables.
Special|Unique, in-memory, or temporary storage.|Memory, Join, File|Engines optimized for specific tasks like in-memory lookups or reading data from files.


Q: Is ClickHouse good at table joins?
A: No, ClickHouse is not optimized for complex, multi-table joins like a traditional OLTP database. It is best suited for workloads with a denormalized data model. The recommended practice is to perform joins during the data ingestion (ETL) process and store the result in a single, wide table.

Q: What are the best ways to get data from Kafka to ClickHouse?
A: The most robust method is to use the official ClickHouse Kafka Connect Sink connector. This connector is purpose-built to handle data types like Avro and is a more reliable, low-code solution than using the generic JDBC connector.


