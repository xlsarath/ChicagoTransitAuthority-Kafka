# Public Transit Status with Apache Kafka

In this project, you will construct a streaming event pipeline around Apache Kafka and its ecosystem. Using public data from the [Chicago Transit Authority](https://www.transitchicago.com/data/) we will construct an event pipeline around Kafka that allows us to simulate and display the status of train lines in real time.

When the project is complete, you will be able to monitor a website to watch trains move from station to station.

![Final User Interface](images/ui.png)


## Prerequisites

The following are required to complete this project:

* Docker
* Python 3.7
* Access to a computer with a minimum of 16gb+ RAM and a 4-core CPU to execute the simulation

## Description

The Chicago Transit Authority (CTA) has asked us to develop a dashboard displaying system status for its commuters. We have decided to use Kafka and ecosystem tools like REST Proxy and Kafka Connect to accomplish this task.

Our architecture will look like so:

![Project Architecture](images/diagram.png)

### Step 1: Create Kafka Producers
The first step in our plan is to configure the train stations to emit some of the events that we need. The CTA has placed a sensor on each side of every train station that can be programmed to take an action whenever a train arrives at the station.

To accomplish this, you must complete the following tasks:

1. Complete the code in `producers/models/producer.py`
1. Define a `value` schema for the arrival event in `producers/models/schemas/arrival_value.json` with the following attributes
	* `station_id`
	* `train_id`
	* `direction`
	* `line`
	* `train_status`
	* `prev_station_id`
	* `prev_direction`
1. Complete the code in `producers/models/station.py` so that:
	* A topic is created for each station in Kafka to track the arrival events
	* The station emits an `arrival` event to Kafka whenever the `Station.run()` function is called.
	* Ensure that events emitted to kafka are paired with the Avro `key` and `value` schemas
1. Define a `value` schema for the turnstile event in `producers/models/schemas/turnstile_value.json` with the following attributes
	* `station_id`
	* `station_name`
	* `line`
1. Complete the code in `producers/models/turnstile.py` so that:
	* A topic is created for each turnstile for each station in Kafka to track the turnstile events
	* The station emits a `turnstile` event to Kafka whenever the `Turnstile.run()` function is called.
	* Ensure that events emitted to kafka are paired with the Avro `key` and `value` schemas

### Step 2: Configure Kafka REST Proxy Producer
Our partners at the CTA have asked that we also send weather readings into Kafka from their weather hardware. Unfortunately, this hardware is old and we cannot use the Python Client Library due to hardware restrictions. Instead, we are going to use HTTP REST to send the data to Kafka from the hardware using Kafka's REST Proxy.

To accomplish this, you must complete the following tasks:

1. Define a `value` schema for the weather event in `producers/models/schemas/weather_value.json` with the following attributes
	* `temperature`
	* `status`
1. Complete the code in `producers/models/weather.py` so that:
	* A topic is created for weather events
	* The weather model emits `weather` event to Kafka REST Proxy whenever the `Weather.run()` function is called.
		* **NOTE**: When sending HTTP requests to Kafka REST Proxy, be careful to include the correct `Content-Type`. Pay close attention to the [examples in the documentation](https://docs.confluent.io/current/kafka-rest/api.html#post--topics-(string-topic_name)) for more information.
	* Ensure that events emitted to REST Proxy are paired with the Avro `key` and `value` schemas

### Step 3: Configure Kafka Connect
Finally, we need to extract station information from our PostgreSQL database into Kafka. We've decided to use the [Kafka JDBC Source Connector](https://docs.confluent.io/current/connect/kafka-connect-jdbc/source-connector/index.html).

To accomplish this, you must complete the following tasks:

1. Complete the code and configuration in `producers/connectors.py`
	* Please refer to the [Kafka Connect JDBC Source Connector Configuration Options](https://docs.confluent.io/current/connect/kafka-connect-jdbc/source-connector/source_config_options.html) for documentation on the options you must complete.
	* You can run this file directly to test your connector, rather than running the entire simulation.
	* Make sure to use the [Landoop Kafka Connect UI](http://localhost:8084) and [Landoop Kafka Topics UI](http://localhost:8085) to check the status and output of the Connector.
	* To delete a misconfigured connector: `CURL -X DELETE localhost:8083/connectors/stations`

### Step 4: Configure the Faust Stream Processor
We will leverage Faust Stream Processing to transform the raw Stations table that we ingested from Kafka Connect. The raw format from the database has more data than we need, and the line color information is not conveniently configured. To remediate this, we're going to ingest data from our Kafka Connect topic, and transform the data.

To accomplish this, you must complete the following tasks:

1. Complete the code and configuration in `consumers/faust_stream.py

#### Watch Out!

You must run this Faust processing application with the following command:

`faust -A faust_stream worker -l info`

### Step 5: Configure the KSQL Table
Next, we will use KSQL to aggregate turnstile data for each of our stations. Recall that when we produced turnstile data, we simply emitted an event, not a count. What would make this data more useful would be to summarize it by station so that downstream applications always have an up-to-date count

To accomplish this, you must complete the following tasks:

1. Complete the queries in `consumers/ksql.py`

#### Tips

* The KSQL CLI is the best place to build your queries. Try `ksql` in your workspace to enter the CLI.
* You can run this file on its own simply by running `python ksql.py`
* Made a mistake in table creation? `DROP TABLE <your_table>`. If the CLI asks you to terminate a running query, you can `TERMINATE <query_name>`


### Step 6: Create Kafka Consumers
With all of the data in Kafka, our final task is to consume the data in the web server that is going to serve the transit status pages to our commuters.

To accomplish this, you must complete the following tasks:

1. Complete the code in `consumers/consumer.py`
1. Complete the code in `consumers/models/line.py`
1. Complete the code in `consumers/models/weather.py`
1. Complete the code in `consumers/models/station.py`

### Documentation
In addition to the course content you have already reviewed, you may find the following examples and documentation helpful in completing this assignment:

* [Confluent Python Client Documentation](https://docs.confluent.io/current/clients/confluent-kafka-python/#)
* [Confluent Python Client Usage and Examples](https://github.com/confluentinc/confluent-kafka-python#usage)
* [REST Proxy API Reference](https://docs.confluent.io/current/kafka-rest/api.html#post--topics-(string-topic_name))
* [Kafka Connect JDBC Source Connector Configuration Options](https://docs.confluent.io/current/connect/kafka-connect-jdbc/source-connector/source_config_options.html)

## Directory Layout
The project consists of two main directories, `producers` and `consumers`.

The following directory layout indicates the files that the student is responsible for modifying by adding a `*` indicator. Instructions for what is required are present as comments in each file.

```
* - Indicates that the student must complete the code in this file

├── consumers
│   ├── consumer.py *
│   ├── faust_stream.py *
│   ├── ksql.py *
│   ├── models
│   │   ├── lines.py
│   │   ├── line.py *
│   │   ├── station.py *
│   │   └── weather.py *
│   ├── requirements.txt
│   ├── server.py
│   ├── topic_check.py
│   └── templates
│       └── status.html
└── producers
    ├── connector.py *
    ├── models
    │   ├── line.py
    │   ├── producer.py *
    │   ├── schemas
    │   │   ├── arrival_key.json
    │   │   ├── arrival_value.json *
    │   │   ├── turnstile_key.json
    │   │   ├── turnstile_value.json *
    │   │   ├── weather_key.json
    │   │   └── weather_value.json *
    │   ├── station.py *
    │   ├── train.py
    │   ├── turnstile.py *
    │   ├── turnstile_hardware.py
    │   └── weather.py *
    ├── requirements.txt
    └── simulation.py
```

## Running and Testing

To run the simulation, you must first start up the Kafka ecosystem on their machine utilizing Docker Compose.

```%> docker-compose up```

Docker compose will take a 3-5 minutes to start, depending on your hardware. Please be patient and wait for the docker-compose logs to slow down or stop before beginning the simulation.

Once docker-compose is ready, the following services will be available:

| Service | Host URL | Docker URL | Username | Password |
| --- | --- | --- | --- | --- |
| Public Transit Status | [http://localhost:8888](http://localhost:8888) | n/a | ||
| Landoop Kafka Connect UI | [http://localhost:8084](http://localhost:8084) | http://connect-ui:8084 |
| Landoop Kafka Topics UI | [http://localhost:8085](http://localhost:8085) | http://topics-ui:8085 |
| Landoop Schema Registry UI | [http://localhost:8086](http://localhost:8086) | http://schema-registry-ui:8086 |
| Kafka | PLAINTEXT://localhost:9092,PLAINTEXT://localhost:9093,PLAINTEXT://localhost:9094 | PLAINTEXT://kafka0:9092,PLAINTEXT://kafka1:9093,PLAINTEXT://kafka2:9094 |
| REST Proxy | [http://localhost:8082](http://localhost:8082/) | http://rest-proxy:8082/ |
| Schema Registry | [http://localhost:8081](http://localhost:8081/ ) | http://schema-registry:8081/ |
| Kafka Connect | [http://localhost:8083](http://localhost:8083) | http://kafka-connect:8083 |
| KSQL | [http://localhost:8088](http://localhost:8088) | http://ksql:8088 |
| PostgreSQL | `jdbc:postgresql://localhost:5432/cta` | `jdbc:postgresql://postgres:5432/cta` | `cta_admin` | `chicago` |

Note that to access these services from your own machine, you will always use the `Host URL` column.

When configuring services that run within Docker Compose, like **Kafka Connect you must use the Docker URL**. When you configure the JDBC Source Kafka Connector, for example, you will want to use the value from the `Docker URL` column.

### Running the Simulation

There are two pieces to the simulation, the `producer` and `consumer`. As you develop each piece of the code, it is recommended that you only run one piece of the project at a time.

However, when you are ready to verify the end-to-end system prior to submission, it is critical that you open a terminal window for each piece and run them at the same time. **If you do not run both the producer and consumer at the same time you will not be able to successfully complete the project**.

#### To run the `producer`:

1. `cd producers`
2. `virtualenv venv`
3. `. venv/bin/activate`
4. `pip install -r requirements.txt`
5. `python simulation.py`

Once the simulation is running, you may hit `Ctrl+C` at any time to exit.

#### To run the Faust Stream Processing Application:
1. `cd consumers`
2. `virtualenv venv`
3. `. venv/bin/activate`
4. `pip install -r requirements.txt`
5. `faust -A faust_stream worker -l info`


#### To run the KSQL Creation Script:
1. `cd consumers`
2. `virtualenv venv`
3. `. venv/bin/activate`
4. `pip install -r requirements.txt`
5. `python ksql.py`

#### To run the `consumer`:

** NOTE **: Do not run the consumer until you have reached Step 6!
1. `cd consumers`
2. `virtualenv venv`
3. `. venv/bin/activate`
4. `pip install -r requirements.txt`
5. `python server.py`

Once the server is running, you may hit `Ctrl+C` at any time to exit.

OverView:
1. producers/simulation.py

root@0c62e10e6a8c:/home/workspace/producers# python simulation.py
2021-05-13 00:11:48,571 __main__     INFO     Beginning simulation, press Ctrl+C to exit at any time
2021-05-13 00:11:48,573 __main__     INFO     loading kafka connect jdbc source connector
2021-05-13 00:11:50,332 __main__     INFO     beginning cta train simulation
2021-05-13 00:11:52,387 models.producer INFO     org.chicago.cta.weather.v1 is created

2. consumers/faust -A faust_stream worker -l info

root@0c62e10e6a8c:/home/workspace/consumers# faust -A faust_stream worker -l info
┌ƒaµS† v1.7.4─┬───────────────────────────────────────────────────┐
│ id          │ stations-stream                                   │
│ transport   │ [URL('kafka://localhost:9092')]                   │
│ store       │ memory:                                           │
│ web         │ http://localhost:6066/                            │
│ log         │ -stderr- (info)                                   │
│ pid         │ 8148                                              │
│ hostname    │ 0c62e10e6a8c                                      │
│ platform    │ CPython 3.7.3 (Linux x86_64)                      │
│ drivers     │                                                   │
│   transport │ aiokafka=1.0.6                                    │
│   web       │ aiohttp=3.6.2                                     │
│ datadir     │ /home/workspace/consumers/stations-stream-data    │
│ appdir      │ /home/workspace/consumers/stations-stream-data/v1 │
└─────────────┴───────────────────────────────────────────────────┘
[2021-05-13 00:12:52,751: INFO]: [^Worker]: Starting... 
[2021-05-13 00:12:52,759: INFO]: [^-App]: Starting... 
[2021-05-13 00:12:52,760: INFO]: [^--Monitor]: Starting... 
[2021-05-13 00:12:52,760: INFO]: [^--Producer]: Starting... 
[2021-05-13 00:12:52,760: INFO]: [^---ProducerBuffer]: Starting... 
[2021-05-13 00:12:52,788: INFO]: [^--CacheBackend]: Starting... 
[2021-05-13 00:12:52,789: INFO]: [^--Web]: Starting... 
[2021-05-13 00:12:52,789: INFO]: [^---Server]: Starting... 
[2021-05-13 00:12:52,790: INFO]: [^--Consumer]: Starting... 
[2021-05-13 00:12:52,792: INFO]: [^---AIOKafkaConsumerThread]: Starting... 
[2021-05-13 00:12:53,763: INFO]: [^--LeaderAssignor]: Starting... 
[2021-05-13 00:12:53,764: INFO]: [^--Producer]: Creating topic 'stations-stream-__assignor-__leader' 
[2021-05-13 00:12:53,803: INFO]: [^--Producer]: Topic 'stations-stream-__assignor-__leader' created. 
[2021-05-13 00:12:53,804: INFO]: [^--ReplyConsumer]: Starting... 
[2021-05-13 00:12:53,804: INFO]: [^--AgentManager]: Starting... 
[2021-05-13 00:12:53,804: INFO]: [^--Agent: faust_stream.station_event]: Starting... 
[2021-05-13 00:12:53,809: INFO]: [^---OneForOneSupervisor]: Starting... 
[2021-05-13 00:12:53,810: INFO]: [^--Conductor]: Starting... 
[2021-05-13 00:12:53,810: INFO]: [^--TableManager]: Starting... 
[2021-05-13 00:12:54,811: INFO]: [^--Table: org.chicago.cta.stations.table.v1]: Starting... 
[2021-05-13 00:12:54,814: INFO]: [^---Store: org.chicago.cta.stations.table.v1]: Starting... 
[2021-05-13 00:12:54,816: INFO]: [^--Producer]: Creating topic 'org.chicago.cta.stations.table.v1' 
[2021-05-13 00:12:54,868: INFO]: [^--Producer]: Topic 'org.chicago.cta.stations.table.v1' created. 
[2021-05-13 00:12:54,869: INFO]: [^---Recovery]: Starting... 
[2021-05-13 00:12:55,811: INFO]: [^--Producer]: Creating topic 'stations-stream-__assignor-__leader' 
[2021-05-13 00:12:56,814: INFO]: Updating subscribed topics to: frozenset({'org.chicago.cta.stations', 'org.chicago.cta.stations.table.v1', 'stations-stream-__assignor-__leader'}) 
[2021-05-13 00:12:56,815: INFO]: Subscribed to topic(s): {'org.chicago.cta.stations', 'org.chicago.cta.stations.table.v1', 'stations-stream-__assignor-__leader'} 
[2021-05-13 00:12:56,825: INFO]: Discovered coordinator 1 for group stations-stream 
[2021-05-13 00:12:56,827: INFO]: Revoking previously assigned partitions set() for group stations-stream 
[2021-05-13 00:12:57,814: INFO]: (Re-)joining group stations-stream 
[2021-05-13 00:12:57,819: INFO]: Joined group 'stations-stream' (generation 1) with member_id faust-1.7.4-5c65e161-bda3-491a-8011-541e1f43633a 
[2021-05-13 00:12:57,819: INFO]: Elected group leader -- performing partition assignments using faust 
[2021-05-13 00:12:57,823: INFO]: Successfully synced group stations-stream with generation 1 
[2021-05-13 00:12:57,824: INFO]: Setting newly assigned partitions {TopicPartition(topic='stations-stream-__assignor-__leader', partition=0), TopicPartition(topic='org.chicago.cta.stations', partition=0), TopicPartition(topic='org.chicago.cta.stations.table.v1', partition=0)} for group stations-stream 
[2021-05-13 00:12:59,769: INFO]: [^---Recovery]: Highwater for active changelog partitions:
┌Highwater - Active─────────────────┬───────────┬───────────┐
│ topic                             │ partition │ highwater │
├───────────────────────────────────┼───────────┼───────────┤
│ org.chicago.cta.stations.table.v1 │ 0         │ -1        │
└───────────────────────────────────┴───────────┴───────────┘ 
[2021-05-13 00:13:01,195: INFO]: [^---Recovery]: active offsets at start of reading:
┌Reading Starts At - Active─────────┬───────────┬────────┐
│ topic                             │ partition │ offset │
├───────────────────────────────────┼───────────┼────────┤
│ org.chicago.cta.stations.table.v1 │ 0         │ -1     │
└───────────────────────────────────┴───────────┴────────┘ 
[2021-05-13 00:13:01,772: INFO]: [^---Recovery]: standby offsets at start of reading:
┌Reading Starts At - Standby─┐
│ topic │ partition │ offset │
└───────┴───────────┴────────┘ 
[2021-05-13 00:13:02,770: INFO]: [^---Recovery]: Resuming flow... 
[2021-05-13 00:13:02,771: INFO]: [^---Recovery]: Recovery complete 
[2021-05-13 00:13:02,872: INFO]: [^---Recovery]: Restore complete! 
[2021-05-13 00:13:02,873: INFO]: [^---Recovery]: Seek stream partitions to committed offsets. 
[2021-05-13 00:13:03,775: INFO]: [^--Fetcher]: Starting... 
[2021-05-13 00:13:03,775: INFO]: [^---Recovery]: Worker ready 
[2021-05-13 00:13:03,776: INFO]: [^Worker]: Ready 
[2021-05-13 00:13:07,779: INFO]: Timer commit woke up too late, with a drift of +0.9306395850000628 

3. python consumers/ksql.py  --tables get created
>ksql
ksql> select * from turnstile;
1620865100258 | Ƨ���^ | 41330 | Montrose | blue
1620865100260 | ȧ���^ | 41330 | Montrose | blue
1620865100260 | ʧ���^ | 41330 | Montrose | blue
1620865100323 | ¨���^ | 40370 | Washington | blue
1620865100323 | ƨ���^ | 40370 | Washington | blue
1620865100324 | Ȩ���^ | 40370 | Washington | blue
1620865100340 | 訵��^ | 41340 | LaSalle | blue
1620865100455 | Ϊ���^ | 41380 | Bryn Mawr | red
ksql> select * from TURNSTILE_SUMMARY;
1620865207512 | 40230 | 40230 | 11
1620865207329 | 40350 | 40350 | 12
1620865207265 | 41280 | 41280 | 11

4. python consumers/server.py

root@0c62e10e6a8c:/home/workspace# python consumers/server.py 
2021-05-13 00:22:30,510 __main__     INFO     Open a web browser to http://localhost:8888 to see the Transit Status Page
2021-05-13 00:22:33,818 consumer     INFO     partitions assigned for org.chicago.cta.weather.v1
2021-05-13 00:22:34,150 consumer     INFO     partitions assigned for TURNSTILE_SUMMARY
2021-05-13 00:22:35,151 consumer     INFO     partitions assigned for ^org.chicago.cta.station.arrivals.
2021-05-13 00:22:36,153 consumer     INFO     partitions assigned for org.chicago.cta.stations.table.v1

5. Kafka Topics CLI useful commands:
### Kafka Topics CLI, topics appear for arrivals on each train line in addition to the turnstiles for each of those stations.
command kafka-topics --list --zookeeper localhost:2181 

### Kafka Topics CLI, messages continuously appear for each station on the train line, for both arrivals and turnstile actions.
kafka-avro-console-consumer --topic org.chicago.cta.turnstile.v1 --from-beginning --bootstrap-server localhost:9092 --max-messages 10

### kafka-console-consumer, (REST proxy)weather messages are visible in the weather topic and are regularly produced as the simulation runs.
kafka-avro-console-consumer --topic org.chicago.cta.weather.v1 --from-beginning --bootstrap-server localhost:9092 --max-messages 10

### kafka-console-consumer, all stations defined in Postgres are visible in the stations topic.
kafka-console-consumer --topic org.chicago.cta.stations --from-beginning --bootstrap-server localhost:9092 --max-messages 10

### A consumer group for Faust is created on the Kafka Connect Stations topic
python consumers/faust_stream.py tables

### Data is ingested in the Station format and is then transformed into the TransformedStation format.
python consumers/faust_stream.py --json worker

### A topic is present in Kafka with the output topic name the student supplied. Inspecting messages in the topic, every station ID is represented.
kafka-console-consumer --topic org.chicago.cta.stations.table.v1 --from-beginning --bootstrap-server localhost:9092 --max-messages 10

### Using the KSQL CLI, turnstile data is visible in the table TURNSTILE.
select * from turnstile;

### Using the KSQL CLI, verify that station IDs have an associated count column.
select * from TURNSTILE_SUMMARY;