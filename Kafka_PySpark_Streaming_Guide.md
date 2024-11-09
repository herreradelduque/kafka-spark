
# Kafka, PySpark, and Streaming Setup Guide

This guide covers the process of setting up Kafka, PySpark, and streaming data to the console, including the necessary Docker commands, Python setup, and PySpark code. The steps are compiled in a well-organized markdown format.

## 1. **Set up Docker with Kafka and Zookeeper**

### **1.1 Docker Compose Setup**

Create a `docker-compose.yml` file for Kafka and Zookeeper:

```yaml
version: "3.7"
services:
  zookeeper:
    restart: always
    image: docker.io/bitnami/zookeeper:3.8
    ports:
      - "2181:2181"
    volumes:
      - "zookeeper-volume:/bitnami"
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    restart: always
    image: docker.io/bitnami/kafka:3.3
    ports:
      - "9093:9093"
    volumes:
      - "kafka-volume:/bitnami"
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9093
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka:9092,EXTERNAL://localhost:9093
      - KAFKA_CFG_INTER_BROKER_LISTENER_NAME=CLIENT
    depends_on:
      - zookeeper
volumes:
  kafka-volume:
  zookeeper-volume:
```

### **1.2 Start Kafka and Zookeeper with Docker Compose**

In your terminal, navigate to the directory containing the `docker-compose.yml` file and run:

```bash
docker-compose up -d
```

This will start the Kafka and Zookeeper containers in the background.

## 2. **Create Kafka Topic**

### **2.1 Access the Kafka Container**

Identify the Kafka container using:

```bash
docker ps
```

To enter the Kafka container (we do have 3 containers...choose first..):

```bash
docker exec -it kafka-spark-kafka-1 /bin/bash
```

### **2.2 Create a Kafka Topic**

Once inside the container, create a Kafka topic by running:

```bash
/opt/bitnami/kafka/bin/kafka-topics.sh --create --topic your_topic_name --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

Replace `your_topic_name` with your desired topic name.

### **2.3 Verify the Topic**

List all topics to verify that your topic was created:

```bash
/opt/bitnami/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

---

## 3. **Send Events to Kafka**

You can send events to Kafka either from the Kafka CLI or using a Python script.

### **3.1 Send Events from Kafka CLI**

Enter the Kafka container again (if not already inside) and use the Kafka producer:

```bash
docker exec -it kafka-spark-kafka-1 /bin/bash
/opt/bitnami/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic your_topic_name
```

Once inside the producer, you can type messages to send them to Kafka.

---

## 4. **Consume Events from Kafka**

To consume the events from the Kafka topic and view them in the console, use the Kafka consumer:

```bash
/opt/bitnami/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic your_topic_name --from-beginning
```

---

## 5. **Set up PySpark for Streaming**

### **5.1 Install PySpark and kafka-python**

Install the necessary Python libraries to interact with Kafka:

```bash
pip install pyspark kafka-python
```

### **5.2 Set up PySpark Streaming**

Here’s an example of a PySpark script to consume data from Kafka and output it to the console.

```python
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder     .appName("KafkaStreamExample")     .getOrCreate()

# Define DataFrame that reads from Kafka
df = spark.readStream     .format("kafka")     .option("kafka.bootstrap.servers", "localhost:9093")     .option("subscribe", "your_topic_name")     .load()

# Start a streaming query that writes to the console
query = df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")     .writeStream     .outputMode("append")     .format("console")     .start()

# Wait for the termination signal to keep the stream open
query.awaitTermination()
```

### **5.3 Run the PySpark Script**

Run the script with:

```bash
python your_spark_script.py
```

This will start streaming data from your Kafka topic to the console.

---

## 6. **Verify the Streaming Process**

- **Kafka Producer:** You can produce messages to the topic as shown in Step 3.
- **Kafka Consumer (PySpark):** If you have the PySpark script running, it should show the incoming events in the console.

---

## Troubleshooting Tips

- Ensure that **Kafka** is running and accessible at the correct ports (`localhost:9093`).
- Ensure the **topic name** in both the producer and consumer matches.
- If using **Python 3.10+**, ensure you are using the latest version of `kafka-python` (>=2.0.2) to avoid compatibility issues.
- If Kafka is not starting, ensure that Docker resources (such as CPU and memory) are allocated properly, as Kafka can be resource-intensive.

---

## Conclusion

With this setup, you've configured Kafka in Docker, created topics, sent events, and consumed events using both the Kafka CLI and PySpark. This provides a basic real-time streaming pipeline for processing data in a Kafka environment.
