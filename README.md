# kafka-spark

https://developer.confluent.io/confluent-tutorials/kafka-on-docker/

https://hackernoon.com/setting-upkcat -b localhost:9093 -L  # list all topics currently in kafka-kafka-on-docker-for-local-development



##### 1 ######


Running Kafka on Docker
To run Kafka on Docker, first confirm your Docker Desktop is running. Then execute the following command from the kafka-on-docker directory:

Copy
 docker compose up -d
The -d flag runs the docker container in detached mode which is similar to running Unix commands in the background by appending &. To confirm the container is running, run this command:

Copy
docker logs broker
And if everything is running ok you'll see something like this at this at the end of screen output:

Copy
[2024-05-21 17:30:58,752] INFO Awaiting socket connections on broker:29092. (kafka.network.DataPlaneAcceptor)
[2024-05-21 17:30:58,754] INFO Awaiting socket connections on 0.0.0.0:9092. (kafka.network.DataPlaneAcceptor)
[2024-05-21 17:30:58,756] INFO [BrokerServer id=1] Waiting for all of the authorizer futures to be completed (kafka.server.BrokerServer)
[2024-05-21 17:30:58,756] INFO [BrokerServer id=1] Finished waiting for all of the authorizer futures to be completed (kafka.server.BrokerServer)
[2024-05-21 17:30:58,756] INFO [BrokerServer id=1] Waiting for all of the SocketServer Acceptors to be started (kafka.server.BrokerServer)
[2024-05-21 17:30:58,756] INFO [BrokerServer id=1] Finished waiting for all of the SocketServer Acceptors to be started (kafka.server.BrokerServer)
[2024-05-21 17:30:58,756] INFO [BrokerServer id=1] Transition from STARTING to STARTED (kafka.server.BrokerServer)
[2024-05-21 17:30:58,757] INFO Kafka version: 3.7.0 (org.apache.kafka.common.utils.AppInfoParser)
[2024-05-21 17:30:58,757] INFO Kafka commitId: 2ae524ed625438c5 (org.apache.kafka.common.utils.AppInfoParser)
[2024-05-21 17:30:58,757] INFO Kafka startTimeMs: 1716312658757 (org.apache.kafka.common.utils.AppInfoParser)
[2024-05-21 17:30:58,758] INFO [KafkaRaftServer nodeId=1] Kafka Server started (kafka.server.KafkaRaftServer)
Now let's produce and consume a message! To produce a message, let's open a command terminal on the Kafka container:

Copy
docker exec -it -w /opt/kafka/bin broker sh
Then create a topic:

Copy
./kafka-topics.sh --create --topic my-topic --bootstrap-server broker:29092
The result of this command should be

Copy
Created topic my-topic.
Important
Take note of the --bootstrap-server flag. Because you're connecting to Kafka inside the container, you use broker:29092 for the host:port. If you were to use a client outside the container to connect to Kafka, a producer application running on your laptop for example, you'd use localhost:9092 instead.

Next, start a console producer with this command:

Copy
./kafka-console-producer.sh  --topic my-topic --bootstrap-server broker:29092
At the prompt copy each line one at time and paste into the terminal hitting enter key after each one:

Copy
All streams
lead to Kafka
Then enter a CTRL-C to close the producer.

Now let's consume the messages with this command:

Copy
./kafka-console-consumer.sh --topic my-topic --from-beginning --bootstrap-server broker:29092
And you should see the following:

Copy
All streams
lead to Kafka
Enter a CTRL-C to close the consumer and then type exit to close the docker shell.

To shut down the container, run

Copy
docker compose down -v