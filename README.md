# Kafka Connect MongoDB Plugin

Dockerfile for [Mongo official Plugin For Kafka Connect](https://github.com/mongodb/mongo-kafka)

This Image is available in [Docker Hub](docker.io/elliminium/kafka-connect-mongodb)

# Example:

run `docker-compose up -d --build`

### Mongo Replica set :

    docker-compose exec mongo1 mongo

    rs.initiate({
          _id : "rs0",
          members: [
            { _id : 0, host : "mongo1:27017", priority: 1.0 },
            { _id : 1, host : "mongo2:27017", priority: 0.5 },
          ]
        })

your mongo string url would be :

> mongodb://127.0.0.1:27017,127.0.0.1:27018/?replicaSet=rs0

### Generate Some Data (kafka connect datagen):

    curl -X POST -H "Content-Type: application/json" --data '
      { "name": "datagen-pageviews",
        "config": {
          "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
          "kafka.topic": "pageviews",
          "quickstart": "pageviews",
          "key.converter": "org.apache.kafka.connect.json.JsonConverter",
          "value.converter": "org.apache.kafka.connect.json.JsonConverter",
          "value.converter.schemas.enable": "false",
          "producer.interceptor.classes": "io.confluent.monitoring.clients.interceptor.MonitoringProducerInterceptor",
          "max.interval": 200,
          "iterations": 10000000,
          "tasks.max": "1"
    }}' http://localhost:8083/connectors -w "\n"

### Add MongoDB Kafka Sink Connector:

    curl -X POST -H "Content-Type: application/json" --data '
      { "name": "mongo-sink",
        "config": {
          "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
          "tasks.max":"1",
          "topics":"pageviews",
          "connection.uri":"mongodb://mongo1:27017,mongo2:27017",
          "database":"test",
          "collection":"pageviews",
          "key.converter": "org.apache.kafka.connect.storage.StringConverter",
          "value.converter": "org.apache.kafka.connect.json.JsonConverter",
          "value.converter.schemas.enable": "false"
    }}' http://localhost:8083/connectors -w "\n"

## Add MongoDB Kafka Source Connector:

    curl -X POST -H "Content-Type: application/json" --data '
      {"name": "mongo-source",
       "config": {
         "tasks.max":"1",
         "connector.class":"com.mongodb.kafka.connect.MongoSourceConnector",
         "connection.uri":"mongodb://mongo1:27017,mongo2:27017",
         "topic.prefix":"mongo",
         "database":"test",
         "collection":"pageviews"
    }}' http://localhost:8083/connectors -w "\n"

## Result

navigate to `127.0.0.1:9021` to see connectors and topics.
