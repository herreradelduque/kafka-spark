# Kafka, PySpark, and Streaming Setup Guide

This guide outlines the setup process for Kafka, Zookeeper, and PySpark to enable real-time data streaming, including Docker commands, Python setup, and sample PySpark code.

## 1. **Docker Setup with Kafka and Zookeeper**

### **1.1 Docker Compose File**

Create a `docker-compose.yml` file to define services for Kafka and Zookeeper:
```
version: "3.7"
services:
  zookeeper:
    image: docker.io/bitnami/zookeeper:3.8
    restart: always
    ports:
      - "2181:2181"
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
    volumes:
      - "zookeeper-volume:/bitnami"

  kafka:
    image: docker.io/bitnami/kafka:3.3
    restart: always
    ports:
      - "9092:9092"
      - "9093:9093"
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_LISTENERS=CLIENT://0.0.0.0:9092,EXTERNAL://0.0.0.0:9093
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka:9092,EXTERNAL://localhost:9093
      - KAFKA_CFG_INTER_BROKER_LISTENER_NAME=CLIENT
    depends_on:
      - zookeeper
    volumes:
      - "kafka-volume:/bitnami"

  kafdrop:
    image: obsidiandynamics/kafdrop:latest
    restart: always
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: "kafka:9092"
      JVM_OPTS: "-Xms32M -Xmx64M"
    depends_on:
      - kafka

volumes:
  kafka-volume:
  zookeeper-volume:

```


### **1.2 Start Kafka and Zookeeper Services**

In the terminal, navigate to the directory containing `docker-compose.yml` and run:

```
docker-compose up -d
```

This command will start the Kafka, Zookeeper, and Kafdrop services in the background.

---

## 2. **Create Kafka Topics**

### **2.1 Access the Kafka Container**

To enter the Kafka container, find the container ID and open a bash session inside it:
```
docker ps
```

```
docker exec -it -u 0 kafka-spark-kafka-1 /bin/bash
```

### **2.2 Create a Kafka Topic**

Inside the Kafka container, use the following command to create a topic:

```
/opt/bitnami/kafka/bin/kafka-topics.sh --create --topic your_topic_name --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

Replace `your_topic_name` with your desired topic name.

### **2.3 Verify the Topic**

List all topics to confirm successful creation:

```
/opt/bitnami/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

---

## 3. **Send Events to Kafka**

You can send events to Kafka either from the Kafka CLI or using a Python script.

### **3.1 Send Events from the Kafka CLI**

If you are already in the Kafka container, use the following command to start a Kafka producer:

```
/opt/bitnami/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic your_topic_name
```

Inside the producer, type messages to send them to Kafka.

---

## 4. **Consume Events from Kafka**

To consume events from a Kafka topic and view them in the console, use the following command:

```
/opt/bitnami/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic your_topic_name --from-beginning
```

---

## 5. **Set up PySpark for Streaming**

### **5.1 Install PySpark and kafka-python**

Install the necessary libraries for Python to interact with Kafka:

```
pip install pyspark kafka-python
```

### **5.2 PySpark Streaming Example**

Here’s an example PySpark script to consume data from Kafka and output it to the console:

```python
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("KafkaStreamExample").getOrCreate()

# Define DataFrame that reads from Kafka
df = spark.readStream     .format("kafka")     .option("kafka.bootstrap.servers", "localhost:9093")     .option("subscribe", "your_topic_name")     .load()

# Start streaming query that writes to console
query = df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")     .writeStream     .outputMode("append")     .format("console")     .start()

# Wait for the termination signal to keep stream open
query.awaitTermination()
```

Replace `"your_topic_name"` with your topic name.

### **5.3 Run the PySpark Script**

Execute the script:

```
python your_spark_script.py
```

---

## 6. **Verify the Streaming Process**

- **Kafka Producer:** Produce messages as shown in Step 3.
- **Kafka Consumer (PySpark):** If your PySpark script is running, it should display incoming events in the console.

---

## Troubleshooting Tips

- Ensure **Kafka** is running and accessible at the correct ports (`localhost:9093`).
- Ensure the **topic name** in both the producer and consumer matches.
- If using **Python 3.10+**, ensure compatibility with the latest version of `kafka-python` (>=2.0.2).
- If Kafka fails to start, verify Docker resources (CPU/memory) as Kafka is resource-intensive.

---

## Conclusion

With this setup, you can create and manage Kafka topics, produce and consume events, and process data in real-time with PySpark. This provides a basic streaming data pipeline for Kafka in a Docker environment.
```

This is how your document would appear in plain text format. Let me know if you need any further modifications!