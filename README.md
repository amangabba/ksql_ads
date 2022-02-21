# ksqlDB - Ads

## Set up

### start up the infrastructure

Bring up the entire stack by running:

```
docker-compose up -d
```

### Open ksqlDB cli

```
docker exec -it ksqldb-cli ksql http://ksqldb-server:8088
```

### Configure it for this demo

run the following commands that allows to rerun the queries always from the beginning of the data streams. This is **not to use in production**! It is for debugging/educational purpose only.

```
SET 'auto.offset.reset' = 'earliest';
```

## Introduction

Please refer to [continuous-analytics-examples/epl_robotic-arm/readme.md](https://github.com/quantiaconsulting/continuous-analytics-examples/tree/master/epl_robotic-arm).

## Code

### Creating the streams
TEXT
#### Views stream
```
CREATE STREAM AdViews_STREAM (id VARCHAR, stressLevel INT, ts VARCHAR)
  WITH (
  kafka_topic='AdView_topic', 
  value_format='json', 
  partitions=1,
  timestamp='ts',
  timestamp_format='yyyy-MM-dd''T''HH:mm:ssZ');
```
#### Clicks stream
```
CREATE STREAM AdClicks_STREAM (id VARCHAR, stressLevel INT, ts VARCHAR)
  WITH (
  kafka_topic='AdClicks_topic', 
  value_format='json', 
  partitions=1,
  timestamp='ts',
  timestamp_format='yyyy-MM-dd''T''HH:mm:ssZ');
```
### Data generation
The events are generated through an [external data generator](https://github.com/amangabba/ksql_ads/edit/main/data_generator)
>Data stream used in the examples
>* a click event arrives 1 sec after the view
>* a click event arrives 11 sec after the view
>* a view event arrives 1 sec after the click
>* there is a view event but no click
>* there is a click event but no view
>* there are two consecutive view events and one click event 1 sec after the first view
>* there is a view event followed by two click events shortly after each other

![](https://cdn.confluent.io/wp-content/uploads/input-streams-1-768x300.jpg)
#### Generation of Views stream
```
INSERT INTO AdViews_STREAM (id, ts) VALUES ('A', '2021-10-23T06:00:00+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('B', '2021-10-23T06:00:01+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('C', '2021-10-23T06:00:03+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('D', '2021-10-23T06:00:04+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('F', '2021-10-23T06:00:06+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('F', '2021-10-23T06:00:06+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('G', '2021-10-23T06:00:08+0200');
```
#### Generation of Clicks stream
```
INSERT INTO ClickViews_STREAM (id, ts) VALUES ('A', '2021-10-23T06:00:01+0200');
INSERT INTO ClickViews_STREAM (id, ts) VALUES ('C', '2021-10-23T06:00:02+0200');
INSERT INTO ClickViews_STREAM (id, ts) VALUES ('E', '2021-10-23T06:00:05+0200');
INSERT INTO ClickViews_STREAM (id, ts) VALUES ('F', '2021-10-23T06:00:07+0200');
INSERT INTO ClickViews_STREAM (id, ts) VALUES ('G', '2021-10-23T06:00:09+0200');
INSERT INTO ClickViews_STREAM (id, ts) VALUES ('G', '2021-10-23T06:00:09+0200');
INSERT INTO ClickViews_STREAM (id, ts) VALUES ('B', '2021-10-23T06:00:11+0200');
```
### Kstream to Kstream joins
TEXT
#### Inner join
![](https://cdn.confluent.io/wp-content/uploads/inner_stream-stream_join-768x475.jpg)
```
code
```
#### Left join
![](https://cdn.confluent.io/wp-content/uploads/left-stream-stream-join-768x459.jpg)
```
code
```
#### Outer join
![](https://cdn.confluent.io/wp-content/uploads/outer-stream-stream-join-768x464.jpg)
```
code
```
### Ktable to Ktable joins
TEXT
#### Inner join
![](https://cdn.confluent.io/wp-content/uploads/Inner-Table-Table-Join-768x727.jpg)
```
code
```
#### Left join
![](https://cdn.confluent.io/wp-content/uploads/Left-Table-Table-Join-768x727.jpg)
```
code
```
#### Outer join
![](https://cdn.confluent.io/wp-content/uploads/Outer-Table-Table-Join-768x727.jpg)
```
code
```
### Kstream to Ktable joins
TEXT
#### Inner join
![](https://cdn.confluent.io/wp-content/uploads/Inner-Stream-Table-Join-768x561.jpg)
```
code
```
#### Left join
![](https://cdn.confluent.io/wp-content/uploads/Left-Stream-Table-Join-768x561.jpg)
```
code
```
### Kstream to Global Ktable joins
TEXT
#### Inner join
![](https://cdn.confluent.io/wp-content/uploads/Inner-Stream-GlobalTable-Join-768x561.jpg)
```
code
```
#### Left join
![](https://cdn.confluent.io/wp-content/uploads/Left-Steam-GlobalTable-Join-768x561.jpg)
```
code
```
