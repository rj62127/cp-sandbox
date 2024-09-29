# task_no3_part_01

# Kafka SSL Authentication Setup Guide


This guide provides a step-by-step process to configure SSL for a Kafka cluster with three nodes. The goal is to enable secure communication between Kafka brokers and clients using SSL encryption. If you're new to Kafka or SSL, don't worry—this guide explains everything in detail.

## Prerequisites

1. **Install Java JDK**: Kafka requires Java to run, and you need the `keytool` command (which is part of JDK) for SSL setup.
   - [Download JDK](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html)

2. **Install OpenSSL**: You'll use OpenSSL to generate certificates.
   - On Linux: 
     ```bash
     sudo apt-get install openssl
     ```
   - On macOS (using Homebrew): 
     ```bash
     brew install openssl
     ```

3. **Kafka & Zookeeper**: Make sure you have a working Kafka and Zookeeper setup. Kafka needs Zookeeper to coordinate its brokers.
   - [Kafka Quickstart](https://kafka.apache.org/quickstart)

## Step 1: Understand SSL and Why We Need It

Kafka uses SSL (Secure Sockets Layer) to:
- Encrypt data between brokers and clients.
- Ensure that only authorized clients can communicate with the brokers.
- Prevent data from being altered or intercepted during transmission.

SSL works by using certificates to verify the identity of brokers and clients. These certificates are stored in files called **keystores** and **truststores**.

## Step 2: Create Your Own Certificate Authority (CA)

The first step is to create your own CA, which will be used to sign certificates for your Kafka brokers. A CA is like a digital notary that verifies the identity of brokers and clients.

### Command to Create CA:
```bash
openssl req -new -x509 -days 365 -keyout ca-key -out ca-cert -subj "/C=DE/ST=NRW/L=MS/O=juplo/OU=kafka/CN=Root-CA" -passout pass:kafka-broker

Explanation:
    -new -x509: This tells OpenSSL to create a new self-signed certificate.
    -days 365: The certificate is valid for 1 year.
    -keyout ca-key: This is the private key file for the CA.
    -out ca-cert: This is the public certificate file for the CA.
    -subj: Specifies the details of the certificate (Country, State, City, Organization, etc.).
    -passout pass:kafka-broker: Sets a password for the CA’s private key.

This command generates two files:

    ca-key: The private key of the CA (used to sign certificates).
    ca-cert: The public certificate of the CA (to be distributed).

Step 3: Create Truststore and Import Root CA

    A truststore contains the public certificates of trusted entities (in this case, our CA). Kafka brokers and clients use this truststore to trust the CA that signed the certificates.

Command to Create Truststore:

    keytool -keystore kafka.server.truststore.jks -storepass kafka-broker -import -alias ca-root -file ca-cert -noprompt

Explanation:

    keytool: A command-line tool to manage Java keystores and certificates.
    -keystore kafka.server.truststore.jks: Creates a truststore file.
    -storepass kafka-broker: Password for the truststore.
    -import -alias ca-root: Imports the CA certificate into the truststore.
    -file ca-cert: The CA certificate created in Step 2.
    -noprompt: Automatically confirms the import.

Step 4: Create a Keystore for Kafka Brokers

    A keystore contains the private key and certificate for each Kafka broker. Each broker needs its own keystore with a unique certificate.

Command to Generate Keystore:

    keytool -keystore kafka.server.keystore.jks -storepass kafka-broker -alias kafka1 -validity 365 -keyalg RSA -genkeypair -keypass kafka-broker -dname "CN=kafka1,OU=kafka,O=juplo,L=MS,ST=NRW,C=DE"


Explanation:

    -keystore kafka.server.keystore.jks: Creates a keystore file for the broker.
    -storepass kafka-broker: Password for the keystore.
    -alias kafka1: The alias (name) of the broker in the keystore.
    -keyalg RSA: The algorithm used to generate the key.
    -genkeypair: Generates a public-private key pair.
    -dname: Specifies the details for the broker's certificate.

Step 5: Create a Certificate Signing Request (CSR)

    The CSR is sent to the CA to sign the broker's certificate. This ensures that the broker's identity can be verified.

Command to Generate CSR:

keytool -alias kafka1 -keystore kafka.server.keystore.jks -certreq -file cert-file -storepass kafka-broker -keypass kafka-broker

Explanation:

    -certreq: Creates a certificate request.
    -file cert-file: The CSR file to be signed by the CA.

Step 6: Sign the Broker’s Certificate Using the CA

    Now, use the CA to sign the broker’s certificate. This verifies that the broker’s identity is trusted by the CA.

Command to Sign the CSR:

openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial -passin pass:kafka-broker -extensions SAN -extfile <(printf "\n[SAN]\nsubjectAltName=DNS:kafka1,DNS:localhost")


Explanation:

    -CA ca-cert: The CA certificate.
    -CAkey ca-key: The CA private key.
    -in cert-file: The CSR file to be signed.
    -out cert-signed: The signed certificate.
    -subjectAltName: Specifies alternative names (e.g., broker’s DNS name).

Step 7: Import the CA Certificate and Signed Certificate into Keystore

    First, import the CA certificate into the broker’s keystore.

Command to Import CA Certificate:

    keytool -importcert -keystore kafka.server.keystore.jks -alias ca-root -file ca-cert -storepass kafka-broker -keypass kafka-broker -noprompt


Next, import the signed certificate into the broker’s keystore.

Command to Import Signed Certificate:

    keytool -keystore kafka.server.keystore.jks -alias kafka1 -import -file cert-signed -storepass kafka-broker -keypass kafka-broker -noprompt


Step 8: Configure Kafka Brokers for SSL

Update the server.properties file for each Kafka broker to enable SSL.

Example server.properties:

    listeners=SSL://0.0.0.0:9093
    ssl.keystore.location=/path/to/kafka.server.keystore.jks
    ssl.keystore.password=kafka-broker
    ssl.key.password=kafka-broker
    ssl.truststore.location=/path/to/kafka.server.truststore.jks
    ssl.truststore.password=kafka-broker
    ssl.client.auth=required
    ssl.endpoint.identification.algorithm=


Explanation:

    listeners=SSL://0.0.0.0:9093: Kafka will listen for SSL connections on port 9093.
    Set the correct paths to your keystore and truststore files.

Restart each Kafka broker after modifying this configuration.


Step 9: Verify the Setup with Docker and Grafana

    After setting up SSL, you can verify that everything is working by checking the Docker logs and accessing Grafana.

Start Docker Containers: Bring up all Docker containers for your Kafka cluster.

    docker-compose up -d

Check Docker Logs: Ensure that the Kafka brokers and other services are up and running without        SSL-related errors. Use this command to check the logs:

    docker logs <container_name>


Access Grafana:

    Open your browser and go to http://localhost:3000.
    Log in with the default credentials (admin / admin).
    Once logged in, go to the home screen and browse the dashboard to verify that the Kafka metrics are being displayed correctly.




NOTE:-  Document SSL setup and Grafana monitoring instructions in README for Kafka cluster. If you want to check the results, go to the "Images" section where I've added screenshots for reference.
