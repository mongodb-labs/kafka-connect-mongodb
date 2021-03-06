# MongoDB Kafka Connector

## The MongoDB Kafka source connector guide

An [Apache Kafka](https://kafka.apache.org/) [connect source connector](https://kafka.apache.org/documentation/#connect) for 
[MongoDB](https://www.mongodb.com/).

For more information about configuring connectors in general see the 
[official Confluent documentation](https://docs.confluent.io/current/connect/managing/configuring.html).

### MongoDB Source

Kafka records are generated from [change stream event documents](https://docs.mongodb.com/manual/changeStreams/). Change streams can observe
changes at the _collection_, _database_ or _client_ level.

Data is read from MongoDB using the configuration connection as specified in the 
[connection string](http://mongodb.github.io/mongo-java-driver/3.10/javadoc/com/mongodb/ConnectionString.html).

**Note:** Change streams require a replicaSet or a sharded cluster using replicaSets.

Currently, only JSON strings are supported as the output.

#### Event Document Format

The following document represents all possible fields that a change stream response document can have:

```
{
   _id : { <BSON Object> },
   "operationType" : "<operation>",
   "fullDocument" : { <document> },
   "ns" : {
      "db" : "<database>",
      "coll" : "<collection"
   },
   "to" : {
      "db" : "<database>",
      "coll" : "<collection"
   },
   "documentKey" : { "_id" : <value> },
   "updateDescription" : {
      "updatedFields" : { <document> },
      "removedFields" : [ "<field>", ... ]
   }
   "clusterTime" : <Timestamp>,
   "txnNumber" : <NumberLong>,
   "lsid" : {
      "id" : <UUID>,
      "uid" : <BinData>
   }
}
```

### Source Connector Configuration Properties 

| Name                        | Description                                                                                                                                                                                                                                                                                                          | Type    | Default                                                   | Valid Values                                   | Importance |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|-----------------------------------------------------------|------------------------------------------------|------------|
| connection.uri              | The connection URI as supported by the official drivers. eg: ``mongodb://user@pass@locahost/``.                                                                                                                                                                                                                      | string  | mongodb://localhost:27017,localhost:27018,localhost:27019 | A valid connection string                      | high       |
| database                    | The database to watch. If not set then all databases will be watched.                                                                                                                                                                                                                                                | string  | ""                                                        |                                                | medium     |
| collection                  | The collection in the database to watch. If not set then all collections will be watched.                                                                                                                                                                                                                            | string  | ""                                                        |                                                | medium     |
| publish.full.document.only  | Only publish the actual changed document rather than the full change stream document. Automatically, sets `change.stream.full.document=updateLookup` so updated documents will be included.                                                                                                                          | boolean | false                                                     |                                                | high       |
| pipeline                    | An inline JSON array with objects describing the pipeline operations to run. Example: `[{"$match": {"operationType": "insert"}}, {"$addFields": {"Kafka": "Rules!"}}]`                                                                                                                                               | string  | []                                                        | A valid JSON array                             | medium     |
| collation                   | The json representation of the Collation options to use for the change stream. Use the `Collation.asDocument().toJson()` to create the specific json representation.                                                                                                                                                 | string  | ""                                                        | A valid JSON document representing a collation | high       |
| batch.size                  | The cursor batch size.                                                                                                                                                                                                                                                                                               | int     | 0                                                         | [0,...]                                        | medium     |
| change.stream.full.document | Determines what to return for update operations when using a Change Stream. When set to 'updateLookup', the change stream for partial updates will include both a delta describing the changes to the document as well as a copy of the entire document that was changed from *some time* after the change occurred. | string  | ""                                                        | An empty string OR [default, updatelookup]     | high       |
| poll.await.time.ms          | The amount of time to wait before checking for new results on the change stream                                                                                                                                                                                                                                      | long    | 5000                                                      | [1,...]                                        | low        |
| poll.max.batch.size         | Maximum number of change stream documents to include in a single batch when polling for new data. This setting can be used to limit the amount of data buffered internally in the connector.                                                                                                                         | int     | 1000                                                      | [1,...]                                        | low        |
| topic.prefix                | Prefix to prepend to database & collection names to generate the name of the Kafka topic to publish data to.                                                                                                                                                                                                         | string  | ""                                                        |                                                | low        |


### Custom pipelines

The following example can be used to only observe inserted files:

```properties
pipeline=[{"$match": {"operationType": "insert"}}]
```

_Note:_ MongoDB requires all change stream documents to include the resume token - the top level `_id` field.

### Publishing the changed documents only

Due to the restriction of requiring the resume token, a special configuration option has been added that allows users to only publish the 
actual documents after inserts, replaces or updates.

```properties
publish.changed.documents.only=true
```

This will automatically configure `change.stream.full.document=updateLookup` and will only publish events that contain a `fullDocument` field.

### Topic naming convention

The MongoDB Kafka Source connector will publish events to topics using the events namespace. For example, an insert into the 'data' collection 
in the 'test' database will publish to a topic called: 'test.data'.

The following example, will set the topic prefix to be 'mongo':

```properties
topic.prefix=mongo
```

In this case changes to the 'data' collection in the 'test' database will published to a topic called: 'mongo.test.data'.

---
### Next

- [Installation guide](./install.md)
- [The MongoDB Kafka sink connector guide](./sink.md)
- The MongoDB Kafka source connector guide
- [A docker end 2 end example](../docker/README.md)
- [Changelog](./changelog.md)
