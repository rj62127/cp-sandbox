Troubleshooting Red Color in Kafka Broker Panel

Problem Statement

    The customer notices a red color for one of the panels in the control-center. This indicates a problem with one of the Kafka brokers in the system. The task is to troubleshoot the issue, identify the cause, and provide a solution to resolve it.

Solution Overview

To troubleshoot and resolve the issue, we will:

    Set up a secure environment using SSL/TLS certificates for Kafka brokers.

    Modify the Docker Compose configuration to include Kafka Connect.

    Ensure proper configuration for the Kafka cluster, and validate that all services are up and running without errors.

By the end of this process, the red indicator for the Kafka broker in the control-center should turn green, indicating a healthy status.


Steps to Fix the Issue

1. Stop All Docker Containers

    First, stop all running Docker containers to ensure we can start with a fresh setup.

        docker-compose down -v


2. Bring Up Docker Containers

    Now, bring the containers back up to reproduce the issue:

        docker-compose up -d

    You should see a red indicator on the Kafka broker panel when you visit localhost:9021 in the browser.


3. Create a Custom CA and Secure Kafka Brokers

    To fix the issue, we'll set up SSL/TLS for the Kafka brokers by creating a custom Certificate Authority (CA) and configuring keystores and truststores.

    Step-by-Step Process:

        Create Your Own CA:

        Generate a new CA key and certificate using openssl.

            openssl req -new -x509 -days 365 -keyout ca-key -out ca-cert -subj "/C=DE/ST=NRW/L=MS/O=juplo/OU=kafka/CN=Root-CA" -passout pass:kafka-broker

        Create Truststore and Import Root CA:

        Import the CA certificate into a truststore for the Kafka brokers.

            keytool -keystore kafka.server.truststore.jks -storepass kafka-broker -import -alias ca-root -file ca-cert -noprompt

        Create Keystore:

        Generate a new keystore and private key for the Kafka broker.

            keytool -keystore kafka.server.keystore.jks -storepass kafka-broker -alias kafka1 -validity 365 -keyalg RSA -genkeypair -keypass kafka-broker -dname "CN=kafka1,OU=kafka,O=juplo,L=MS,ST=NRW,C=DE"

        Create a Certificate Signing Request (CSR):

        Generate a CSR for the Kafka broker.

            keytool -alias kafka1 -keystore kafka.server.keystore.jks -certreq -file cert-file -storepass kafka-broker -keypass kafka-broker

        Sign the Certificate Using the CA:

        Sign the CSR with the CA certificate.

            openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial -passin pass:kafka-broker -extensions SAN -extfile <(printf "\n[SAN]\nsubjectAltName=DNS:kafka1,DNS:localhost")

        Import the CA Certificate into the Keystore:

        Import the CA certificate into the Kafka broker's keystore.

            keytool -importcert -keystore kafka.server.keystore.jks -alias ca-root -file ca-cert -storepass kafka-broker -keypass kafka-broker -noprompt

        Import the Signed Certificate into the Keystore:

        Import the signed certificate into the Kafka broker's keystore.

            keytool -keystore kafka.server.keystore.jks -alias kafka1 -import -file cert-signed -storepass kafka-broker -keypass kafka-broker -noprompt


4. Modify Docker Compose Configuration

    Next, add Kafka Connect service to the docker-compose.yaml file.

    Add the following section to your docker-compose.yaml:

        connect:
            image: confluentinc/cp-server-connect:7.4.0
            hostname: connect
            container_name: connect
            depends_on:
            - kafka1
            - kafka2
            - kafka3
            - schema-registry
            command: connect-distributed /etc/kafka/connect-distributed.properties
            ports:
            - "8083:8083"
            volumes:
            - ./connect:/etc/kafka
            deploy:
            resources:
                limits:
                cpus: "2"
                memory: 2048M


5. Restart Docker Containers

    After making the necessary changes, ensure all containers are stopped and then bring them back up:

        docker-compose down -v
        docker-compose up -d


6. Verify Logs

    Check the logs for all services to ensure everything is working correctly:

        docker ps -a


7. Visit Control Center

    Once everything is up and running, visit localhost:9021 in your browser. The red indicator in the broker panel should now turn green, indicating a healthy Kafka broker setup.


Results 

![alt text](<images/Screenshot from 2024-10-07 15-25-39.png>)
![alt text](<images/Screenshot from 2024-10-07 15-25-59.png>)