# This dockerfile adds to the strimzi kafka base image to support connectors - ref https://strimzi.io/docs/operators/latest/deploying#creating-new-image-using-kafka-connect-build-str
# Postgresl driver, Avro conector, JDBC connector and a custom JAR
# Postgresql Driver - https://jdbc.postgresql.org/download
# The connectors are from Confluent Hub, no way to download
# Avro connector - https://www.confluent.io/hub/confluentinc/kafka-connect-avro-converter 
# JDBC connector - https://www.confluent.io/hub/confluentinc/kafka-connect-jdbc
# Debezium Postgresql Connector - https://www.confluent.io/hub/debezium/debezium-connector-postgresql
# Debezium MySQL Connector - https://www.confluent.io/hub/debezium/debezium-connector-mysql
# Strip Unicode - Local file https://github.com/cyberjar09/StripUnicodeNullTransform

ARG STRIMZI_VERSION=0.50.0-kafka-4.1.1

FROM quay.io/strimzi/kafka:${STRIMZI_VERSION}

ARG AVRO_CONVERTER_VERSION=8.1.0
ARG JDBC_CONNECTOR_VERSION=10.9.1
ARG POSTGRESQL_DRIVER_VERSION=42.7.8
ARG STRIP_UNICODE_VERSION=0.0.2
ARG DEBEZIUM_CONNECTOR_VERSION=3.1.2
ARG MYSQL_DRIVER_VERSION=8.0.33
ARG CLICKHOUSE_CONNECTOR_VERSION=1.3.5

USER root:root
# Download StripUnicodeNullTransform to plugin folder
RUN curl -L -o /opt/kafka/plugins/StripUnicodeNullTransform-${STRIP_UNICODE_VERSION}.jar \
    https://github.com/cyberjar09/StripUnicodeNullTransform/releases/download/${STRIP_UNICODE_VERSION}/StripUnicodeNullTransform-${STRIP_UNICODE_VERSION}.jar 

# Download Confluent JDBC Connector and Avro Converter to plugin folder
COPY ./plugins/ /opt/kafka/plugins/
RUN unzip /opt/kafka/plugins/confluentinc-kafka-connect-jdbc-${JDBC_CONNECTOR_VERSION}.zip -d /opt/kafka/plugins \
    && rm /opt/kafka/plugins/confluentinc-kafka-connect-jdbc-${JDBC_CONNECTOR_VERSION}.zip

RUN unzip /opt/kafka/plugins/confluentinc-kafka-connect-avro-converter-${AVRO_CONVERTER_VERSION}.zip -d /opt/kafka/plugins \
    && rm /opt/kafka/plugins/confluentinc-kafka-connect-avro-converter-${AVRO_CONVERTER_VERSION}.zip

RUN unzip /opt/kafka/plugins/debezium-debezium-connector-postgresql-${DEBEZIUM_CONNECTOR_VERSION}.zip -d /opt/kafka/plugins \
    && rm /opt/kafka/plugins/debezium-debezium-connector-postgresql-${DEBEZIUM_CONNECTOR_VERSION}.zip

RUN unzip /opt/kafka/plugins/debezium-debezium-connector-mysql-${DEBEZIUM_CONNECTOR_VERSION}.zip -d /opt/kafka/plugins/debezium-debezium-connector-mysql \
    && rm /opt/kafka/plugins/debezium-debezium-connector-mysql-${DEBEZIUM_CONNECTOR_VERSION}.zip

RUN curl -L -o /opt/kafka/plugins/confluentinc-kafka-connect-jdbc-${JDBC_CONNECTOR_VERSION}/lib/postgresql-${POSTGRESQL_DRIVER_VERSION}.jar \
    https://repo1.maven.org/maven2/org/postgresql/postgresql/${POSTGRESQL_DRIVER_VERSION}/postgresql-${POSTGRESQL_DRIVER_VERSION}.jar
RUN curl -L -o /opt/kafka/plugins/confluentinc-kafka-connect-jdbc-${JDBC_CONNECTOR_VERSION}/lib/mysql-${MYSQL_DRIVER_VERSION}.jar \
    https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/${MYSQL_DRIVER_VERSION}/mysql-connector-j-${MYSQL_DRIVER_VERSION}.jar

# Download ClickHouse Connector
RUN curl -L -o /opt/kafka/plugins/clickhouse-kafka-connect-v${CLICKHOUSE_CONNECTOR_VERSION}.zip \
    https://github.com/ClickHouse/clickhouse-kafka-connect/releases/download/v${CLICKHOUSE_CONNECTOR_VERSION}/clickhouse-kafka-connect-v${CLICKHOUSE_CONNECTOR_VERSION}.zip \
    && unzip /opt/kafka/plugins/clickhouse-kafka-connect-v${CLICKHOUSE_CONNECTOR_VERSION}.zip -d /opt/kafka/plugins \
    && rm /opt/kafka/plugins/clickhouse-kafka-connect-v${CLICKHOUSE_CONNECTOR_VERSION}.zip

USER 1001