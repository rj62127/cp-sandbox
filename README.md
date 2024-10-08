Kafka Broker Metrics Troubleshooting

    This guide provides steps to troubleshoot an issue where broker metrics are not captured in Confluent Control Center, despite the setup of the Confluent Metrics Reporter. It also includes steps to generate custom SSL certificates for Kafka brokers and configure them accordingly.

Prerequisites

Ensure no other versions of the sandbox are running.

Run the following command to stop and remove any existing services:

    docker-compose down -v

Starting the Scenario

    Start the required services:

        docker-compose up -d

    Wait for all services to be up and healthy:

        docker-compose ps


Problem Statement

You might encounter an issue in Confluent Control Center that states:

    Please set up Confluent Metrics Reporter to view broker metrics.

Although the Confluent Metrics Reporter is set up, broker metrics are not visible. Follow the troubleshooting steps below to resolve the issue.


Troubleshooting Steps

Step 1: Create SSL Certificates for Kafka Brokers

    1.1 Create Your Own CA

    Generate a new CA key and certificate:

        openssl req -new -x509 -days 365 -keyout ca-key -out ca-cert -subj "/C=DE/ST=NRW/L=MS/O=juplo/OU=kafka/CN=Root-CA" -passout pass:kafka-broker

    1.2 Create Truststore and Import Root CA

    Create a truststore for Kafka brokers and import the CA certificate:

        keytool -keystore kafka.server.truststore.jks -storepass kafka-broker -import -alias ca-root -file ca-cert -noprompt

    1.3 Create Keystore for Kafka Broker

    Generate a new keystore and private key for a Kafka broker:

        keytool -keystore kafka.server.keystore.jks -storepass kafka-broker -alias kafka1 -validity 365 -keyalg RSA -genkeypair -keypass kafka-broker -dname "CN=kafka1,OU=kafka,O=juplo,L=MS,ST=NRW,C=DE"

    1.4 Create Certificate Signing Request (CSR)

    Generate a CSR for the broker:

        keytool -alias kafka1 -keystore kafka.server.keystore.jks -certreq -file cert-file -storepass kafka-broker -keypass kafka-broker

    1.5 Sign the CSR Using the CA

    Sign the CSR with the CA certificate:

        openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial -passin pass:kafka-broker -extensions SAN -extfile <(printf "\n[SAN]\nsubjectAltName=DNS:kafka1,DNS:localhost")

    1.6 Import CA Certificate into Keystore

    Import the CA certificate into the broker’s keystore:

        keytool -importcert -keystore kafka.server.keystore.jks -alias ca-root -file ca-cert -storepass kafka-broker -keypass kafka-broker -noprompt

    1.7 Import Signed Certificate into Keystore

    Import the signed certificate into the broker’s keystore:

        keytool -keystore kafka.server.keystore.jks -alias kafka1 -import -file cert-signed -storepass kafka-broker -keypass kafka-broker -noprompt


Step 2: Update Kafka Broker Configuration

Add the following properties to the server.properties file for each Kafka broker:

    listeners=CLIENT://:19092,BROKER://:19093,TOKEN://:19094
    advertised.listeners=CLIENT://kafka1:19092,BROKER://kafka1:19093,TOKEN://kafka1:19094
    ssl.keystore.password=kafka-broker
    listener.security.protocol.map=CLIENT:SASL_PLAINTEXT,BROKER:SSL,TOKEN:SASL_PLAINTEXT
    confluent.metrics.reporter.bootstrap.servers=kafka1:19093,kafka2:29093,kafka3:39093
    confluent.metrics.reporter.ssl.keystore.password=kafka-broker


Step 3: Update Schema Registry Configuration

In the Schema Registry configuration file (schema-registry.properties), add the following line:

    kafkastore.bootstrap.servers=kafka1:19094,kafka2:29094,kafka3:39094


Step 4: Restart Docker Services

    Bring down all services:

        docker-compose down -v

    Start the services again:

        docker-compose up -d

    Wait for all services to be healthy:

        docker-compose ps -a


Verifying the Solution

    After following the above steps, visit http://localhost:9021 in your browser to check if all services are healthy and broker metrics are visible in Confluent Control Center.

![alt text](<images/Screenshot from 2024-10-01 15-01-43.png>)
![alt text](<images/Screenshot from 2024-10-01 15-06-34.png>)
![alt text](<images/Screenshot from 2024-10-01 15-06-44.png>)
![alt text](<images/Screenshot from 2024-10-01 15-06-54.png>)
![alt text](<images/Screenshot from 2024-10-01 15-07-01.png>)
![alt text](<images/Screenshot from 2024-10-01 15-07-07.png>)
![alt text](<images/Screenshot from 2024-10-01 15-07-29.png>)
![alt text](<images/Screenshot from 2024-10-01 15-07-40.png>)



