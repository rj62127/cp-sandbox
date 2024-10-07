# Kafka SSL Configuration Issue - Troubleshooting Guide

## Problem Statement

During a routine health check, the client has noticed that the Confluent Control Center is not accessible, and a few components of the Kafka cluster are down. This issue is caused by an SSL misconfiguration in the Kafka brokers.

## Prerequisites

Before beginning the troubleshooting process, ensure there are no previously running Kafka sandbox containers by using the following command:

```bash
docker-compose down -v

This command will stop and remove all running containers and volumes to ensure a clean start.


Steps to Troubleshoot

Follow the steps below to resolve the SSL configuration issue for all three Kafka brokers.

1. Start the Kafka Cluster

    Start the Kafka cluster by running:

    docker-compose up -d

    Wait for all services to come up by checking the status with:

    docker-compose ps -a

    If all services are marked as healthy, no further action is required. However, if you notice any service, particularly the Control Center or any broker, is down or inaccessible, proceed with the troubleshooting steps below.


2. SSL Configuration Fix (for Broker 1, Broker 2, and Broker 3)

    The SSL misconfiguration needs to be fixed for each of the three Kafka brokers. The steps below must be applied to Broker 1, Broker 2, and Broker 3.

    Step 1: Create Your Own CA

    Generate a new CA key and certificate that will be used to sign certificates for all brokers:

        openssl req -new -x509 -days 365 -keyout ca-key -out ca-cert -subj "/C=DE/ST=NRW/L=MS/O=juplo/OU=kafka/CN=Root-CA" -passout pass:kafka-broker

    Step 2: Create Truststore and Import Root CA

    Create a truststore for Kafka brokers and import the CA certificate into it. This needs to be done only once, as the same truststore will be used for all brokers:

        keytool -keystore kafka.server.truststore.jks -storepass kafka-broker -import -alias ca-root -file ca-cert -noprompt

    Step 3: Create Keystore for Broker 1

    Generate a new keystore and private key for Broker 1:

        keytool -keystore kafka1.server.keystore.jks -storepass kafka-broker -alias kafka1 -validity 365 -keyalg RSA -genkeypair -keypass kafka-broker -dname "CN=kafka1,OU=kafka,O=juplo,L=MS,ST=NRW,C=DE"

    Step 4: Create Certificate Signing Request (CSR) for Broker 1

    Generate a CSR (Certificate Signing Request) for Broker 1:

        keytool -alias kafka1 -keystore kafka1.server.keystore.jks -certreq -file kafka1-cert-file -storepass kafka-broker -keypass kafka-broker

    Step 5: Sign the CSR Using the CA for Broker 1

    Sign the CSR with the CA certificate for Broker 1:

        openssl x509 -req -CA ca-cert -CAkey ca-key -in kafka1-cert-file -out kafka1-cert-signed -days 365 -CAcreateserial -passin pass:kafka-broker -extensions SAN -extfile <(printf "\n[SAN]\nsubjectAltName=DNS:kafka1,DNS:localhost")

    Step 6: Import CA Certificate into Keystore for Broker 1

    Import the CA certificate into Broker 1's keystore:

        keytool -importcert -keystore kafka1.server.keystore.jks -alias ca-root -file ca-cert -storepass kafka-broker -keypass kafka-broker -noprompt

    Step 7: Import Signed Certificate into Keystore for Broker 1

    Finally, import the signed certificate into Broker 1's keystore:

        keytool -keystore kafka1.server.keystore.jks -alias kafka1 -import -file kafka1-cert-signed -storepass kafka-broker -keypass kafka-broker -noprompt

    Repeat Steps 3 to 7 for Broker 2 and Broker 3

    Follow the same procedure (Steps 3 to 7) for Broker 2 and Broker 3 using their respective CNs (CN=kafka2 and CN=kafka3) to generate their own keystores and certificates.


3. Restart Kafka Cluster

    After configuring SSL for all three brokers, restart the Kafka cluster to apply the changes:

        docker-compose down -v
        docker-compose up -d

    Again, wait for all services to be up by running:

        docker-compose ps -a

    If everything is healthy, you can proceed to the verification step.


4. Verify the Solution

Once the Kafka cluster is up and healthy, visit the Confluent Control Center at:

    http://localhost:9021
    
Ensure all brokers and services are up and running, and everything is healthy. You can also verify that the SSL certificates are correctly set by checking the logs of the brokers for any SSL-related errors.














