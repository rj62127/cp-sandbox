Kafka SASL/PLAIN and SSL Configuration Troubleshooting


This document outlines the steps to troubleshoot and resolve issues when trying to produce to the domestic_orders Kafka topic using SASL/PLAIN over PLAINTEXT with user authentication. The client encountered a connection issue with the domestic_orders topic, while the international_orders topic worked fine.


Prerequisites

    Ensure that there are no other versions of the Kafka sandbox running:

        docker-compose down -v

    Start the Kafka cluster using Docker Compose:

        docker-compose up -d

    Wait for all services to be healthy:

        docker-compose ps -a

    Check that the domestic_orders and international_orders topics are created by visiting the Control Center at localhost:9021.


Problem Overview

    The client was able to produce messages to the international_orders topic but received the following error while producing and consuming from the domestic_orders topic:

        WARN [Producer clientId=console-producer] Connection to node 2 (kafka2/172.20.0.6:19093) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)

    Commands used:

        Producer:

                kafka-console-producer --bootstrap-server kafka1:19092 --producer.config /opt/client/client.properties --topic domestic_orders

        Consumer:

                kafka-console-consumer --bootstrap-server kafka1:19092 --consumer.config /opt/client/client.properties --from-beginning --topic domestic_orders

    Cause:

        The error indicates that the connection to node 2 (kafka2) could not be established, suggesting a misconfiguration in the Kafka broker setup.


Solution

Step 1: SSL Configuration

    Create a Custom CA: Generate a new CA key and certificate to sign the broker certificates:

        openssl req -new -x509 -days 365 -keyout ca-key -out ca-cert -subj "/C=DE/ST=NRW/L=MS/O=juplo/OU=kafka/CN=Root-CA" -passout pass:kafka-broker;

    Create Truststore and Import CA: Create a truststore for the Kafka brokers and import the CA certificate:

        keytool -keystore kafka.server.truststore.jks -storepass kafka-broker -import -alias ca-root -file ca-cert -noprompt

    Create a Keystore: Generate a new keystore and private key for the broker (kafka1 in this case):

        keytool -keystore kafka.server.keystore.jks -storepass kafka-broker -alias kafka1 -validity 365 -keyalg RSA -genkeypair -keypass kafka-broker -dname "CN=kafka1,OU=kafka,O=juplo,L=MS,ST=NRW,C=DE"

    Create a Certificate Signing Request (CSR): Generate a CSR for kafka1:

        keytool -alias kafka1 -keystore kafka.server.keystore.jks -certreq -file cert-file -storepass kafka-broker -keypass kafka-broker

    Sign the CSR Using the CA: Sign the CSR using the CA certificate:

        openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial -passin pass:kafka-broker -extensions SAN -extfile <(printf "\n[SAN]\nsubjectAltName=DNS:kafka1,DNS:localhost");

    Import the CA Certificate into Keystore: Import the CA certificate into the broker's keystore:

        keytool -importcert -keystore kafka.server.keystore.jks -alias ca-root -file ca-cert -storepass kafka-broker -keypass kafka-broker -noprompt

    Import the Signed Certificate into Keystore: Finally, import the signed certificate into the broker's keystore:

        keytool -keystore kafka.server.keystore.jks -alias kafka1 -import -file cert-signed -storepass kafka-broker -keypass kafka-broker -noprompt


Step 2: Modify client.properties

In the client.properties file, ensure the following configuration:

    Remove the following lines from the client.properties file:

    password="bob-secret";
    acks=1

    This resolves any potential misconfiguration related to credentials and acknowledgment settings.


Step 3: Docker Compose Changes

    Modify Docker Compose File: In the docker-compose.yml file, there were issues with the entrypoint for the Kafka client. Modify the entrypoint and dependencies:

        Original Entrypoint:

                entrypoint: /opt/client/kafkaClient.sh /opt/client/start.sh

            Modified Entrypoint: Comment out the above line, as it's causing issues with client startup.

        Dependency Configuration: In the depends_on section, comment out unnecessary dependencies:

            depends_on:
                # kafka2
                # kafka3


Step 4: Kafka Server Properties Fix (Kafka 2)

    The server.properties file for kafka2 had an incorrect port configuration. It was set to 19093 instead of the correct port 29092.

        Incorrect:

            listeners=PLAINTEXT://:19093

        Correct:

            listeners=PLAINTEXT://:29092

        Update the server.properties file with the correct port and restart the kafka2 broker.


Step 5: Restart the Environment

After all the changes are applied, restart the environment to ensure everything is correctly set up:

    docker-compose down -v
    docker-compose up -d


Step 6: Verifying the Fix

    Start a Kafka Client:

        Open a terminal and execute the following:

            docker exec -it kfkclient bash

        Then run the producer command to send messages to the domestic_orders topic:

            kafka-console-producer --bootstrap-server kafka1:19092 --producer.config /opt/client/client.properties --topic domestic_orders

    Start Another Kafka Client for the Consumer:

        Open another terminal and execute the following:

            docker exec -it kfkclient bash

        Then run the consumer command to read messages from the domestic_orders topic:

            kafka-console-consumer --bootstrap-server kafka1:19092 --consumer.config /opt/client/client.properties --from-beginning --topic domestic_orders

    Test the Messaging:

        In the first terminal, type a message (e.g., Hello from domestic_orders) and press enter. This message should appear in the second terminal (consumer), confirming that the setup is working correctly.



Result
    After troubleshooting, the client was successfully able to:

        Produce messages to the domestic_orders topic.

        Consume messages from the domestic_orders topic.

    The connection issue with kafka2 (caused by incorrect port configuration) was resolved, and proper SSL setup ensured that the client could communicate securely over SASL/PLAIN with the correct authentication.


Screenshot of Successful Result:

![alt text](<images/Screenshot from 2024-09-30 12-57-07.png>)
![alt text](<images/Screenshot from 2024-09-30 12-58-53.png>)
![alt text](<images/Screenshot from 2024-09-30 12-59-10.png>)
![alt text](<images/Screenshot from 2024-09-30 13-09-22.png>)



