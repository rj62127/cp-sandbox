# task3_scenario1

Kafka Connect Cluster Super User Issue Resolution

Problem Statement:

After a recent security audit, certain users were removed from the super.users list in the Kafka broker's server.properties file. This resulted in the Kafka Connect cluster appearing down on Confluent Control Center (localhost:9021).


The goal is to troubleshoot and securely resolve the issue by updating the super.users list and restarting the Kafka services.

Prerequisites

    Access to Kafka brokers: SSH or direct access to the Kafka broker nodes.
    Administrative access to the Confluent Control Center.
    Kafka and Confluent Platform installed and running.

Steps to Resolve

Step 1: Check the Current Status

Before troubleshooting, confirm that the Kafka Connect cluster is down in the Confluent Control Center on localhost:9021.

    Open a browser and go to http://localhost:9021.
    Navigate to the Connect tab.
    Verify that the Connect cluster is shown as unavailable or down.

Before Troubleshooting Screenshot:

![alt text](<images/Screenshot from 2024-09-23 22-12-47.png>)

Step 2: SSH into Kafka Broker Nodes
SSH into all the Kafka brokers in your cluster (Kafka1, Kafka2, Kafka3). Use the following commands for each broker.

    ssh user@kafka1_ip_address
    ssh user@kafka2_ip_address
    ssh user@kafka3_ip_address

Step 3: Modify server.properties File

For each Kafka broker, update the server.properties file to include the correct super users.

    Open the server.properties file on each broker:

        sudo nano /path/to/kafka/config/server.properties

    Locate the super.users property and update it as follows:

        super.users=User:bob;User:kafka1;User:kafka2;User:kafka3;User:mds;User:schemaregistryUser;User:controlcenterAdmin;User:connectAdmin

    Save and exit the file after making changes (Ctrl + X, then Y to confirm changes).

Step 4: Restart Kafka Brokers

After updating the server.properties file on each broker, restart the Kafka services for the changes to take effect.

    Restart each Kafka broker:

        sudo systemctl restart confluent-kafka

    Check the status of each broker:

        sudo systemctl status confluent-kafka

Step 5: Restart Kafka Connect and Control Center

After updating the Kafka brokers, restart the Kafka Connect service and Confluent Control Center.

    Restart Kafka Connect:

        sudo systemctl restart confluent-kafka-connect

    Restart Control Center:

        sudo systemctl restart confluent-control-center

    Check the status of both services:

        sudo systemctl status confluent-kafka-connect
        sudo systemctl status confluent-control-center

Step 6: Verify the Changes
Once the services are restarted, verify that the Kafka Connect cluster is back online in Confluent Control Center.

    Open a browser and go to http://localhost:9021.
    Navigate to the Connect tab.
    Confirm that the Connect cluster is now available.

After Troubleshooting Screenshot:

![alt text](<images/Screenshot from 2024-09-23 22-18-19.png>)

Step 7: Validate the Security Configuration

After resolving the issue, check the Kafka logs to ensure there are no permission-related errors and that all necessary services are functioning with the updated super.users list.

    tail -f /path/to/kafka/logs/server.log



Before and After Troubleshooting


Before:

    The Kafka Connect cluster is shown as down or unavailable in Confluent Control Center (localhost:9021).
    Error logs in the Connect service may indicate permission issues due to missing super users.

Before Troubleshooting Screenshot:

![alt text](<images/Screenshot from 2024-09-23 22-12-47.png>)


After:
    The Kafka Connect cluster is up and running in Confluent Control Center (localhost:9021).
    No permission-related errors appear in the Kafka and Connect logs.

After Troubleshooting Screenshot:

![alt text](<images/Screenshot from 2024-09-23 22-18-19.png>)
![alt text](<images/Screenshot from 2024-09-23 22-18-25.png>)
![alt text](<images/Screenshot from 2024-09-23 23-12-07.png>)







