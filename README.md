# iu_data_engineering

This repository represents an example infrastructure for a data intensive real time backend application based on Python and Infrastructure as Code (IaC).  
The infrastructure contains a Kafka cluster with several additions like Kafka connect clusters and Faust streaming.  
There is an example data source integrated which simulates continous value streams of an IoT smartwatch sensor.  
After some aggregation and processing activities the data will be uploaded to an InfluxDB database which can be monitored through a Grafana dashboard.  
All components are realized in Docker containers and orchestrated by Docker-Compose.  

# Table of Contents
- [Prerequisites](#prerequisites)
- [Launch](#launch)
- [Project Description](#project-description)
- [Project Structure](#project-structure)
- [External Interfaces](#external-interfaces)
- [Limitations](#limitations)
- [Sources/Bibliography](#bibiography)

# Prerequisites

- [Linux OS](https://help.ubuntu.com/) (recommended distribution: Ubuntu 22.04.3 LTS) must be installed (e.g. in WSL2 or Virtual Machine environment)  
- [Git](https://git-scm.com/doc) (reommended verion: 2.34.1) must be installed in the Linux environment  
- [Docker Desktop](https://docs.docker.com/desktop/) (recommended version: 4.31.1) must be installed  
- Recommended Docker settings (depending on host system availability): >= 12GB RAM usage, <=800% CPU usage (8 cores)
- Internet connection is required for downloading docker containers  


# Launch

To get the backend application running on a local machine, the following steps are required:
1. Clone the repository
    ```
    git clone https://github.com/ChriZa1410/data_eng_backend.git
    ```
2. Launching the backend infrastructure running the Docker-Compose file inside the repository in detached mode
   ```
   docker-compose up -d --build
   ```
    Note: After installing the packages it can take up to 90 seconds until cluster components are finally bootet and synchronised.


# Project Description

- Goals:  
This project realizes an example backend infrastructure for handling data intensive real time stream processing applications. It was developed considering the main aspects reliability, scalability and maintainability.  
Reliability and scalability can be achieved through the implementation of replication via three individual Kafka brokers which can balance the load dynamically depending on the data load. The brokers can also substitute each other in case of failures.  
With the deployment of individual microservices which communicate via defined interfaces a manageable maintainability is achieved. With this restarting single microservices is also possible in case of disturbances.  

  As the infrastructure consists of more components the microservices are implemented as Docker containers which are orchestrated by Docker-Compose centrally to be able to manage and maintain the system.  

- Summary:  

  The backend application receives heart rate values by several IoT smartwatch sensors (simulated input data provided by a *.csv file source). After aggregating the input data some processing calculations are done over all known sensor values. After that the sensor information is separated by sensor ID and then uploaded to an InfluxDB database. Uploaded data streams can be monitored by a Grafana dashboard accompanied by some metrics which give an overview of the Kafka cluster performance.  

- Microservices (Kafka cluster components excluded):
  Main functionality:  
  - data_simulator: simulates IoT smartwatches by providing sensor values to *.csv file
  - data_aggregation: fetches raw sensor data and aggregates data to a proper format
  - data_processing: fetches aggregated sensor data and processes data with some calculations (e.g. max/min/mean values for each sensor)
  - data_upload: fetches processed data and uploads data to Influxdb data sink
  - grafana: monitoring processed IoT sensor data and metrics inside the kafka cluster
  - influxdb: InfluxDB database to store productive and metrics data

  Kafka connect components:
  - ingestion_connector: Source Connect cluster (FileSource) to append external data source (csv file) to the kafka cluster
  - db_connector: Sink Connect cluster (InfluxDB Sink) to append external data sink (InfluxDB) to the kafka cluster for storing productive data of the cluster
  - metrics_connector: Sink Connect cluster (InfluxDB Sink) to append external data sink (InfluxDB) to the kafka cluster for storing performance metrics of the cluster
  - IoT_sensor_data_simulation_smartwatch: simulates an IoT data source of type smartwatch (is a *.csv file source connector instance inside ingestion_connector cluster)
  - kafkaconnect_influxdb_sink_productive: stores productive data after being processed in the kafka cluster (is a sink connector instance inside db_connector cluster)
  - kafkaconnect_influxdb_sink_metrics: stores metrics data after being processed in the kafka cluster (is a sink connector instance inside metrics_connector cluster)

  Initialization and Kafka periphery:
  - init-kafka-topics: defines the topics in the kafka cluster 
  - init-schema-registry: initializes schemas of kafka topics
  - init-databases: initializes InfluxDB databases
  - init-kafkaconnect_influxdb_sink_productive: initializes sink connector instance inside db_connector cluster for productive data
  - init-kafkaconnect_influxdb_sink_metrics: initializes sink connector instance inside db_connector cluster for metrics data
  - control-center: Frontend hub to manage and check health and functionality of the kafka cluster


  More details about the microservices' functionality can be found in comments inside the code files.

![Backend_infrastructure_concept](documentation/concept.jpg)  
Source: Own illustration.

# Project Structure

```
├── README.md
├── docker-compose.yml
├── data
│   └── influxdb
├── data_aggregation
│   ├── Dockerfile
│   ├── aggregation.py
│   └── requirements.txt
├── data_processing
│   ├── Dockerfile
│   ├── processing.py
│   └── requirements.txt
├── data_simulator
│   ├── Dockerfile
│   ├── data_simulator.py
│   └── smartwatch_heartrate_source_data.csv
├── data_upload
│   ├── Dockerfile
│   ├── requirements.txt
│   └── uploading.py
├── databases
│   ├── Dockerfile
│   └── create_databases.sh
├── documentation
│   └── concept.jpg
├── provisioning
│   ├── dashboards
│   │   ├── dashboard.json
│   │   └── dashboard.yml
│   └── datasources
│       └── datasources.yml
└── schema-registry
    ├── Dockerfile
    └── create_schema.sh
```

Tree view created with [tree library](https://linux.die.net/man/1/tree).

# External Interfaces

- Grafana: `localhost:3000`  
  Initial login: user = admin, password = admin (credentials are already preset and do not need to be changed)
  Select one of the following dashboards for visualization:
  - Smartwatch Heartrate - Dashboard
    - the number of smartwatch sensors monitored by the backend infrastructure
    - max. heart rate value over all sensors
    - min. heart rate value over all sensors
    - mean heart rate value over all sensors
    - timestamp of last recorded data sample
    - all recorded values and meta data of each monitored smartwatch sensor

  - Smartwatch Metrics - Dashboard
    - number of uploaded sensor values to database per minute
    - latency of the sensor values from the data simulation to database upload

- Control-Center: `localhost:9021`  
  Confluent Platform's control center provides the possibility to check information and data inside the Kafka cluster, e.g.:
    - health of the brokers
    - existing topics
    - monitoring the data sent via the topics as a consumer's point of view

# Limitations

- Backend infrastructure project can be deployed on a local machine. The integration to a cloud is currently not possible.

- Implementation of the infrastructure focuses on:
  - modular setup with separate microservices which communicate via defined interfaces
  - microservices running in Docker containers and orchestration with Docker-Compose (Infrastructure as Code (IaC))
  - usage of existing Docker images which are configured/modified to achieve specific needs
  - creating own Docker containers with specific logical content to reach backend functionality
  - providing the possibility to check results of data processing via a Grafana dashboard
  - implementation of replication via multiple Kafka brokers

- Implementation does not fulfill all possible requirements regarding:
  - error handling along all microservices
  - security configurations regarding the container communication
  - deep analysis possibility via productive data and metrics dashboard

- Sensor simulation contains fix number of IoT sensors (10) and value ranges to test basic functionality. This does not represent a real smartwatch sensor range.
- Overall performance of the infrastructure can still be improved, e.g. duration of startup
- There may errors occure during container startup and synchronisation process. In this case some of the containers ned to be restarted.

- Licences: 
  - Control Center: Confluent Enterprise License
  - Kafka Connect, Schema registry: Confluent Community License
  - Others: Apache 2.0 License
  for details, see https://docs.confluent.io/platform/current/installation/license.h

- Copyright by respective image providers.  


# Bibliography

- Learned the basic handling of Docker containers with the demos of Github repository [docker/getting-started](https://github.com/docker/getting-started) 
- Made the first steps in the handling of Kafka components and Docker containers with [Steps to setup Kafka cluster on Local And Docker](https://medium.com/nerd-for-tech/steps-to-setup-kafka-cluster-on-local-and-docker-ae84324cde86) 
- Learned about Python based streaming library Faust which is based upon the ideas of Kafka Streaming with [Faust - Python Stream Processing](https://faust-streaming.github.io/faust/index.html) 
- Learned about Kafka Connect and InfluxDB sinks with [Using kafkaconnect](https://kafka-connect-manager.lsst.io/userguide.html) and [InfluxDB Sink Connector for Confluent](https://docs.confluent.io/kafka-connectors/influxdb/current/influx-db-sink-connector/overview.html)
- Project is inspired by conceptual ideas of [backend-infrastructure-demo](https://github.com/finkbefl/backend-infrastructure-demo) GitHub repository by [Florian Finkbeiner](https://github.com/finkbefl)
- Time series data simulation and storing in InfluxDB is inspired by [Time-Series with Kafka, Kafka Connect & InfluxDB](https://lenses.io/blog/2016/12/kafka-influxdb/) by [Christina Daskalaki](https://lenses.io/author/christina-daskalaki/)