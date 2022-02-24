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

This notebook summarizes the concepts described in the following [article](https://www.confluent.io/blog/crossing-streams-joins-apache-kafka/). The Kafka 

## Code

### Creating the streams
TEXT
#### Views stream
```
CREATE STREAM AdViews_STREAM (id VARCHAR, ts VARCHAR)
  WITH (
  kafka_topic='AdView_topic', 
  value_format='json', 
  partitions=1,
  timestamp='ts',
  timestamp_format='yyyy-MM-dd''T''HH:mm:ssZ');
```
#### Clicks stream
```
CREATE STREAM AdClicks_STREAM (id VARCHAR, ts VARCHAR)
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
#### Generation of stream events
![](https://cdn.confluent.io/wp-content/uploads/input-streams-1-768x300.jpg)
```
INSERT INTO AdViews_STREAM (id, ts) VALUES ('A', '2021-10-23T06:00:00+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('B', '2021-10-23T06:00:01+0200');
INSERT INTO AdClicks_STREAM (id, ts) VALUES ('A', '2021-10-23T06:00:01+0200');
INSERT INTO AdClicks_STREAM (id, ts) VALUES ('C', '2021-10-23T06:00:02+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('C', '2021-10-23T06:00:03+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('D', '2021-10-23T06:00:04+0200');
INSERT INTO AdClicks_STREAM (id, ts) VALUES ('E', '2021-10-23T06:00:05+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('F', '2021-10-23T06:00:06+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('F', '2021-10-23T06:00:06+0200');
INSERT INTO AdClicks_STREAM (id, ts) VALUES ('F', '2021-10-23T06:00:07+0200');
INSERT INTO AdViews_STREAM (id, ts) VALUES ('G', '2021-10-23T06:00:08+0200');
INSERT INTO AdClicks_STREAM (id, ts) VALUES ('G', '2021-10-23T06:00:09+0200');
INSERT INTO AdClicks_STREAM (id, ts) VALUES ('G', '2021-10-23T06:00:09+0200');
INSERT INTO AdClicks_STREAM (id, ts) VALUES ('B', '2021-10-23T06:00:11+0200');
```
### Kstream to Kstream joins
TEXT
#### Inner join
![](https://cdn.confluent.io/wp-content/uploads/inner_stream-stream_join-768x475.jpg)
```
SELECT *
FROM AdViews_STREAM V
  JOIN AdClicks_STREAM C WITHIN 10 SECONDS
  ON V.id = C.id
EMIT CHANGES;
```
```
+------------------------+------------------------+------------------------+------------------------+
|V_ID                    |V_TS                    |C_ID                    |C_TS                    |
+------------------------+------------------------+------------------------+------------------------+
|A                       |2021-10-23T06:00:00+0200|A                       |2021-10-23T06:00:01+0200|
|B                       |2021-10-23T06:00:01+0200|B                       |2021-10-23T06:00:11+0200|
|C                       |2021-10-23T06:00:03+0200|C                       |2021-10-23T06:00:02+0200|
|F                       |2021-10-23T06:00:06+0200|F                       |2021-10-23T06:00:07+0200|
|F                       |2021-10-23T06:00:06+0200|F                       |2021-10-23T06:00:07+0200|
|G                       |2021-10-23T06:00:08+0200|G                       |2021-10-23T06:00:09+0200|
|G                       |2021-10-23T06:00:08+0200|G                       |2021-10-23T06:00:09+0200|
```
#### Left join
![](https://cdn.confluent.io/wp-content/uploads/left-stream-stream-join-768x459.jpg)
```
SELECT *
FROM AdViews_STREAM V
  LEFT JOIN AdClicks_STREAM C WITHIN 10 SECONDS
  ON V.id = C.id
EMIT CHANGES;
```
```
+------------------------+------------------------+------------------------+------------------------+
|V_ID                    |V_TS                    |C_ID                    |C_TS                    |
+------------------------+------------------------+------------------------+------------------------+
|A                       |2021-10-23T06:00:00+0200|null                    |null                    |
|A                       |2021-10-23T06:00:00+0200|A                       |2021-10-23T06:00:01+0200|
|B                       |2021-10-23T06:00:01+0200|null                    |null                    |
|C                       |2021-10-23T06:00:03+0200|C                       |2021-10-23T06:00:02+0200|
|D                       |2021-10-23T06:00:04+0200|null                    |null                    |
|F                       |2021-10-23T06:00:06+0200|null                    |null                    |
|F                       |2021-10-23T06:00:06+0200|null                    |null                    |
|F                       |2021-10-23T06:00:06+0200|F                       |2021-10-23T06:00:07+0200|
|F                       |2021-10-23T06:00:06+0200|F                       |2021-10-23T06:00:07+0200|
|G                       |2021-10-23T06:00:08+0200|null                    |null                    |
|G                       |2021-10-23T06:00:08+0200|G                       |2021-10-23T06:00:09+0200|
|G                       |2021-10-23T06:00:08+0200|G                       |2021-10-23T06:00:09+0200|
|B                       |2021-10-23T06:00:01+0200|B                       |2021-10-23T06:00:11+0200|
```
#### Outer join
![](https://cdn.confluent.io/wp-content/uploads/outer-stream-stream-join-768x464.jpg)
```
SELECT *
FROM AdViews_STREAM V
  FULL OUTER JOIN AdClicks_STREAM C WITHIN 10 SECONDS
  ON V.id = C.id
EMIT CHANGES;
```
```
+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+
|ROWKEY                   |V_ID                     |V_TS                     |C_ID                     |C_TS                     |
+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+
|null                     |A                        |2021-10-23T06:00:00+0200 |null                     |null                     |
|null                     |A                        |2021-10-23T06:00:00+0200 |A                        |2021-10-23T06:00:01+0200 |
|null                     |B                        |2021-10-23T06:00:01+0200 |null                     |null                     |
|null                     |null                     |null                     |C                        |2021-10-23T06:00:02+0200 |
|null                     |C                        |2021-10-23T06:00:03+0200 |C                        |2021-10-23T06:00:02+0200 |
|null                     |D                        |2021-10-23T06:00:04+0200 |null                     |null                     |
|null                     |null                     |null                     |E                        |2021-10-23T06:00:05+0200 |
|null                     |F                        |2021-10-23T06:00:06+0200 |null                     |null                     |
|null                     |F                        |2021-10-23T06:00:06+0200 |null                     |null                     |
|null                     |F                        |2021-10-23T06:00:06+0200 |F                        |2021-10-23T06:00:07+0200 |
|null                     |F                        |2021-10-23T06:00:06+0200 |F                        |2021-10-23T06:00:07+0200 |
|null                     |G                        |2021-10-23T06:00:08+0200 |null                     |null                     |
|null                     |G                        |2021-10-23T06:00:08+0200 |G                        |2021-10-23T06:00:09+0200 |
|null                     |G                        |2021-10-23T06:00:08+0200 |G                        |2021-10-23T06:00:09+0200 |
|null                     |B                        |2021-10-23T06:00:01+0200 |B                        |2021-10-23T06:00:11+0200 |
```
### Ktable to Ktable joins
#### Creating a materialized view for AdViews
```
CREATE TABLE AdViews_view AS
  SELECT id AS ID, LATEST_BY_OFFSET(ts) AS TS
  FROM AdViews_STREAM
  GROUP BY id
  EMIT CHANGES;
```
#### Creating a materialized view for AdClicks
```
CREATE TABLE AdClicks_view AS
  SELECT id AS ID, LATEST_BY_OFFSET(ts) AS TS
  FROM AdClicks_STREAM
  GROUP BY id
  EMIT CHANGES;
```
```

```
#### Inner join
![](https://cdn.confluent.io/wp-content/uploads/Inner-Table-Table-Join-768x727.jpg)
```
SELECT V.id AS ID, V.ts AS TS_VIEW, C.ts AS TS_CLICK
FROM AdViews_view V
  INNER JOIN AdClicks_view C
  ON V.id = C.id
  EMIT CHANGES;
```
```
+----------------------------+----------------------------+----------------------------+
|ID                          |TS_VIEW                     |TS_CLICK                    |
+----------------------------+----------------------------+----------------------------+
|A                           |2021-10-23T06:00:00+0200    |2021-10-23T06:00:01+0200    |
|B                           |2021-10-23T06:00:01+0200    |2021-10-23T06:00:11+0200    |
|C                           |2021-10-23T06:00:03+0200    |2021-10-23T06:00:02+0200    |
|F                           |2021-10-23T06:00:06+0200    |2021-10-23T06:00:07+0200    |
|G                           |2021-10-23T06:00:08+0200    |2021-10-23T06:00:09+0200    |
|A                           |2021-10-23T06:00:00+0200    |2021-10-23T06:00:01+0200    |
|C                           |2021-10-23T06:00:03+0200    |2021-10-23T06:00:02+0200    |
|F                           |2021-10-23T06:00:06+0200    |2021-10-23T06:00:07+0200    |
|G                           |2021-10-23T06:00:08+0200    |2021-10-23T06:00:09+0200    |
|B                           |2021-10-23T06:00:01+0200    |2021-10-23T06:00:11+0200    |
```

#### Left join
![](https://cdn.confluent.io/wp-content/uploads/Left-Table-Table-Join-768x727.jpg)
```
SELECT V.id AS ID_VIEW, V.ts AS TS_VIEW, C.id AS ID_CLICK, C.ts AS TS_CLICK
FROM AdViews_view V
  LEFT JOIN AdClicks_view C
  ON V.id = C.id
  EMIT CHANGES;
```
```
+-----------------------------+-----------------------------+-----------------------------+-----------------------------+
|ID_VIEW                      |TS_VIEW                      |ID_CLICK                     |TS_CLICK                     |
+-----------------------------+-----------------------------+-----------------------------+-----------------------------+
|A                            |2021-10-23T06:00:00+0200     |A                            |2021-10-23T06:00:01+0200     |
|B                            |2021-10-23T06:00:01+0200     |B                            |2021-10-23T06:00:11+0200     |
|C                            |2021-10-23T06:00:03+0200     |C                            |2021-10-23T06:00:02+0200     |
|D                            |2021-10-23T06:00:04+0200     |null                         |null                         |
|F                            |2021-10-23T06:00:06+0200     |F                            |2021-10-23T06:00:07+0200     |
|G                            |2021-10-23T06:00:08+0200     |G                            |2021-10-23T06:00:09+0200     |
|A                            |2021-10-23T06:00:00+0200     |A                            |2021-10-23T06:00:01+0200     |
|C                            |2021-10-23T06:00:03+0200     |C                            |2021-10-23T06:00:02+0200     |
|F                            |2021-10-23T06:00:06+0200     |F                            |2021-10-23T06:00:07+0200     |
|G                            |2021-10-23T06:00:08+0200     |G                            |2021-10-23T06:00:09+0200     |
|B                            |2021-10-23T06:00:01+0200     |B                            |2021-10-23T06:00:11+0200     |
```
#### Outer join
![](https://cdn.confluent.io/wp-content/uploads/Outer-Table-Table-Join-768x727.jpg)
```
SELECT V.id AS ID_VIEW, V.ts AS TS_VIEW, C.id AS ID_CLICK, C.ts AS TS_CLICK
FROM AdViews_view V
  FULL OUTER JOIN AdClicks_view C
  ON V.id = C.id
  EMIT CHANGES;
```
```
+-----------------------------+-----------------------------+-----------------------------+-----------------------------+
|ID_VIEW                      |TS_VIEW                      |ID_CLICK                     |TS_CLICK                     |
+-----------------------------+-----------------------------+-----------------------------+-----------------------------+
|A                            |2021-10-23T06:00:00+0200     |A                            |2021-10-23T06:00:01+0200     |
|B                            |2021-10-23T06:00:01+0200     |B                            |2021-10-23T06:00:11+0200     |
|C                            |2021-10-23T06:00:03+0200     |C                            |2021-10-23T06:00:02+0200     |
|D                            |2021-10-23T06:00:04+0200     |null                         |null                         |
|F                            |2021-10-23T06:00:06+0200     |F                            |2021-10-23T06:00:07+0200     |
|G                            |2021-10-23T06:00:08+0200     |G                            |2021-10-23T06:00:09+0200     |
|A                            |2021-10-23T06:00:00+0200     |A                            |2021-10-23T06:00:01+0200     |
|C                            |2021-10-23T06:00:03+0200     |C                            |2021-10-23T06:00:02+0200     |
|null                         |null                         |E                            |2021-10-23T06:00:05+0200     |
|F                            |2021-10-23T06:00:06+0200     |F                            |2021-10-23T06:00:07+0200     |
|G                            |2021-10-23T06:00:08+0200     |G                            |2021-10-23T06:00:09+0200     |
|B                            |2021-10-23T06:00:01+0200     |B                            |2021-10-23T06:00:11+0200     |
```
### Kstream to Ktable joins
TEXT
#### Inner join
![](https://cdn.confluent.io/wp-content/uploads/Inner-Stream-Table-Join-768x561.jpg)
```
SELECT V.id AS ID, V.ts AS TS_VIEW, C.ts AS TS_CLICK
FROM AdViews_STREAM V
  INNER JOIN AdClicks_view C
  ON V.id = C.id
EMIT CHANGES;
```
```

```
#### Left join
![](https://cdn.confluent.io/wp-content/uploads/Left-Stream-Table-Join-768x561.jpg)
```
SELECT V.id AS ID_VIEW, V.ts AS TS_VIEW, C.id AS ID_CLICK, C.ts AS TS_CLICK
FROM AdViews_STREAM V
  LEFT JOIN AdClicks_view C
  ON V.id = C.id
EMIT CHANGES;
```
```

```
### Kstream to Global Ktable joins
GlobalKTable is not supported by KsqlDB yet. In contrast to a KTable that is partitioned over all KafkaStreams instances, a GlobalKTable is fully replicated per KafkaStreams instance. Every partition of the underlying topic is consumed by each GlobalKTable, such that the full set of data is available in every KafkaStreams instance. This provides the ability to perform joins with KStream without having to repartition the input stream. All joins with the GlobalKTable require that a KeyValueMapper is provided that can map from the KeyValue of the left hand side KStream to the key of the right hand side GlobalKTable.
#### Creating a GlobalKTable through StreamBuilder
```
builder.globalTable("topic-name", "queryable-store-name");
```
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
