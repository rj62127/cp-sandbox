# kafka-jmx-grafana-docker

Docker-compose file for Confluent Kafka with configuration mounted as properties files. Brings up Kafka and components with JMX metrics exposed and visualized using Prometheus and Grafana. The environment simulates running Confluent Platform on VMs/Bare metal servers using properties files but using docker containers. The various branches in the repository contains troubleshooting scenarios for Kafka adminstrators to practice production-like issues.

## Start

```
docker-compose up -d
```

## Usage

The docker-compose file brings up 3 node kafka cluster with security enabled. Each service in the compose file has its properties/configurations mounted as a volume from a directory with the same name as the service.

Check the kafka server.properties for more details about the Kafka setup.

### Health

Check if all components are up and running using

```bash
docker-compose ps -a
# Ensure there are no Exited services and all containers have the status `Up`
```


### Client

To use a kafka client, exec into the `kfkclient` container which contains the Kafka CLI and other tools necessary for troubleshooting Kafka. THe `kfkclient` container also contains a properties file mounted to `/opt/client`, which can be used to define the client properties for communicating with Kafka.

```
docker exec -it kfkclient bash
```

### Logs

Check the logs of the respective service by its container name.

```bash
docker logs <container_name> # docker logs kafka1
```

### Restarting services

To restart a particular service - 

```bash
docker-compose restart <service_name> # docker-compose restart kafka1
# OR
docker-compose up -d --force-recreate <service_name> # docker-compose up -d --force-recreate kafka1
```

# Scenario 18

> Before starting ensure that there are no other versions of the sandbox running Run `docker-compose down -v` before starting

1. Start the scenario with docker-compose up -d
2. Wait for all services to be up and healthy `docker-compose ps -a`
3. **Wait for the `init` container to exit. It should exit with code 0. This should take a while, depending on your system.** (You can run `watch docker-compose ps -a`)
4. Verify that the following topics are created through the CLI or control center
- debit_transactions
- credit_transactions
- loan_payments
- new_accounts
- fraud_detection
- business_requests
- account_audit
- clickstream
- international_transactions
- app_telemetry

## Problem Statement

The customer notices a red color for one of the panels in the control-center. Troubleshoot the cause for the red color in the control-center and propose a solution to fix the issue.

![img](control-center.png)