## Schema Management

#### Schema Registry
- Schema Registry is a distributed storage layer for schemas
- uses Kafka as its underlying storage mechanism.
- Some key design decisions:
  - Assigns globally unique ID to each registered schema. Allocated IDs are guaranteed to be monotonically increasing but not necessarily consecutive.
  - Kafka provides the durable backend, and functions as a write-ahead changelog for the state of Schema Registry and the schemas it contains.
  - Schema Registry is designed to be distributed, with single-primary architecture, and ZooKeeper/Kafka coordinates primary election (based on the configuration).
- Supported Formats
  - Avro, JSON, and Protobuf
- Schema ID Allocation  
  - always happens in the primary node and Schema IDs are always monotonically increasing
- Kafka is used as Schema Registry storage backend. The special Kafka topic <kafkastore.topic> (default _schemas), with a single partition, is used as a highly available write ahead log
- Single Primary Architecture
  - one Schema Registry instance is the primary at any given moment
  - primary  -> responsible for writes
  - all nodes are capable of directly serving read requests

- Primary election
  - Kafka Coordinator Primary Election
    - Kafka based primary election is chosen when <kafkastore.connection.url> is not configured and has the Kafka bootstrap brokers <kafkastore.bootstrap.servers> specified. The kafka group protocol, chooses one amongst the primary eligible nodes leader.eligibility=true as the primary
  - ZooKeeper Primary Election - deprecated

- Schema Compatibility Types
  - **BACKWARD** : consumer using schema X can process data produced with schema X or X-1
  - **BACKWARD_TRANSITIVE** : consumer using schema X can process data produced with schema X, X-1, or X-2
  - **FORWARD** : data produced using schema X can be read by consumers with schema X or X-1
  - **FORWARD_TRANSITIVE** : data produced using schema X can be read by consumers with schema X, X-1, or X-2
  - **FULL** : backward and forward compatibile between schemas X and X-1
  - **FULL_TRANSITIVE** : backward and forward compatibile between schemas X, X-1, and X-2
  - **NONE** : compatibility type means schema compatibility checks are disabled.


#### AVRO

- Data types supported
  - Primitive Types
    - null , boolean, int, long, float, double, string, bytes
  - Complex
    - Records, Enums, Arrays, Maps, Unions, Fixed
- Avro Schema Definition

```
namespace (required)
type (required) => record, enum, array, map, union, fixed
name (required)
doc (optional)
aliases (optional)
fields (required) {
    name (required)
    type (required)
    doc (optional)
    default (optional)
    order (optional)
    aliases (optional)
}
```

#### Notes
- Schema Registry stores all schemas in a Kafka topic **_schemas** defined by kafkastore.config = _schemas (default) which is a single partition topic with log compacted.
- HTTP and HTTPS client protocol are supported for schema registry.
- Default port for listener is 8081
- The default response media type application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json are used in response header.


#### Resources

 - https://docs.confluent.io/current/schema-registry/
