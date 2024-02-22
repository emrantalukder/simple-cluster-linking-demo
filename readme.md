
west-link.config
```conf
bootstrap.servers=east:29092
link.mode=BIDIRECTIONAL
```

```sh
kafka-cluster-links --create --link bidirectional-link \
--config-file west-link.config --bootstrap-server localhost:9093
```


east-link.config
```conf
bootstrap.servers=west:29092
link.mode=BIDIRECTIONAL
```

```sh
kafka-cluster-links --create --link bidirectional-link \
--config-file east-link.config --bootstrap-server localhost:9092
```


# start mirrors

```sh
# create topic
kafka-topics --bootstrap-server localhost:9092 --create --topic from-east --partitions 1 --replication-factor 1

# create mirror
kafka-mirrors --create --mirror-topic from-east --link bidirectional-link \
--replication-factor 1 --bootstrap-server localhost:9093
```

```sh
# create topic
kafka-topics --bootstrap-server localhost:9093 --create --topic from-west --partitions 1 --replication-factor 1

# create mirror
kafka-mirrors --create --mirror-topic from-west --link bidirectional-link \
--replication-factor 1 --bootstrap-server localhost:9092
```



# [Active/Active](https://docs.confluent.io/cloud/current/multi-cloud/cluster-linking/dr-failover.html#active-active-tutorial)

# create west link
kafka-cluster-links --create --link bidirectional-link \
--config-file west-link.config --bootstrap-server localhost:9093 \
--consumer-group-filters-json='{"groupFilters":[{"name":"*","patternType":"LITERAL","filterType":"INCLUDE"}]}' \
--topic-filters-json='{"topicFilters":[{"name": "*","patternType": "LITERAL","filterType": "INCLUDE"}]}'

# create east link
kafka-cluster-links --create --link bidirectional-link \
--config-file east-link.config --bootstrap-server localhost:9092 \
--consumer-group-filters-json='{"groupFilters":[{"name":"*","patternType":"LITERAL","filterType":"INCLUDE"}]}' \
--topic-filters-json='{"topicFilters":[{"name": "*","patternType": "LITERAL","filterType": "INCLUDE"}]}'

# produce and consume between environments with resuming consumer from synced offsets
```sh
kcat -b localhost:9092 -P -t clicks

a
b
c

kcat -b localhost:9093 -P -t clicks

d
e
f

# consume messages from east and stop
kcat -b localhost:9092 -G mygrp clicks west_clicks -o beginning

# produce messages to east and west
kcat -b localhost:9092 -P -t clicks

q
r
s

kcat -b localhost:9093 -P -t clicks

t
u
v

# resume consumption in west

kcat -b localhost:9093 -G mygrp clicks east_clicks
```