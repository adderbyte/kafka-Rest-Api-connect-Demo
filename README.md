
## Rest-API Demo for 
Based on the work of [kafka-connect-demo](https://github.com/l0n3star/kafka-connect-demo). The only changes made was to use [kafka Rest connector](https://github.com/llofberg/kafka-connect-rest.git) instead of couchbase connector which was used in the original demo. The source configuration was changed to relflect connection to a url (and not a couchbase source).

## Aim
This helps to connect to a Rest-Api and collect data continuously in kafka. Further analysis can be done on the collected data.


## Prerequisites

* Docker
* bash
* curl 
* Java JDK 1.8

## Set up Rest Api connector and Network

1. Start the rest Api connector Docker container and network:

       ./setup-kafka-restApi.sh
       ./setup-network.sh

## Set up Kafka

1. Start the Confluent Kafka containers:

       ./setup-kafka.sh

It takes a long time for the containers to actually finish starting up.
Eventually you will be able to visit http://localhost:9021/ to see the Confluent Control Center.

While you're waiting for Kafka start, move on to the next step.

## Set up connect-cli tool

0. Install gradle 5 : follow instruction in the url below:
   
    https://linuxize.com/post/how-to-install-gradle-on-ubuntu-18-04/

1. Download and build the `connect-cli` tool:

       ./setup-connect-cli.sh
       
2. Add the tool to your PATH environment variable:

       export PATH=$(pwd)/tmp/kafka-connect-tools/bin:${PATH}

3. Try listing installed connectors:

       connect-cli ps

If the "connect" Kafka container has finished starting up, the tool should report "No running connectors".

## Start a Source connector

Kafka connectors are created and controlled by POSTing JSON to an HTTP server.
The `connect-cli` tool handles that for us, and also handles converting property files to JSON. 
 
    connect-cli run cb-source < config/source-2-raw-json.properties
To watch the connector service log file:

    docker logs -f connect

If all goes well, the connector will populate the `connect-demo` topic with documents from the `travel-sample` bucket.



## Experiment with other configurations

The config directory has a few other source and sink configurations to play with.
To re-stream the Couchbase documents, first delete the source connector:

    connect-cli rm cb-source
    

Then recreate it using :  ` connect-cli run cb-source < config/source-2-default-schema.properties `

## Modifying the custom source handler

After modifying the source code for `CustomSourceHandler` (located under the `custom-extensions` directory) you'll need to restart the Kafka containers in order for the new version to be active:
  
    docker-compose -f docker/docker-compose.yml down -v
    ./setup-kafka.sh

## Clean up

When you're done with the demo, remove the Docker containers:

    ./cleanup.sh
    
## For data scientist (additional info after installation/setup)

To log on to the ksql server after setting up kafka, couchdb and command line tool

`docker exec -it ksql-cli ksql http://ksql-server:8088 `

Or use the python wrapper

` pip install ksql `

* for more info on ksql:

[python wrapper for the KSQL REST API](https://github.com/bryanyang0528/ksql-python)

for info on why you need the python wrapper for ksql, see link below:

[why you need the python wrapper for ksql_REST_API](https://www.confluent.io/blog/machine-learning-with-python-jupyter-ksql-tensorflow/#:~:text=The%20result%20of%20such%20a,and%20other%20widespread%20Python%20libraries.)

* Connector-ksql integration

[connector-ksql integration](https://docs.ksqldb.io/en/latest/concepts/connectors/)

### KSQL-Connect Integration [To do]

[ksql Connect integration](https://docs.confluent.io/5.4.2/ksql/docs/developer-guide/ksql-connect.html)
