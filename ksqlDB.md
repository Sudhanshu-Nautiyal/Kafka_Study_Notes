


- KSQL is not ANSI SQL compliant, for now there are no defined standards on streaming SQL languages

- Pull query
  -  pull data at a point in time
  - like traditional db

```
  SELECT ride_id, current_latitude, current_longitude
  FROM ride_locations
  WHERE ROWKEY = ‘6fd0fcdb’;

  +-----------+-----------------------+-----------------------+
  |RIDE_ID    |CURRENT_LATITUDE	    |CURRENT_LONGITUDE	    |
  +-----------+-----------------------+-----------------------+
  |45334	    |37.7749		    |122.4194		    |
  +-----------+-----------------------+-----------------------+

```

- Push query
  - for continunous changes
  - EMIT changes

```
SELECT ride_id, current_latitude, current_longitude
FROM ride_locations
WHERE ROWKEY = ‘6fd0fcdb’
EMIT CHANGES;

+-----------+-----------------------+-----------------------+
|RIDE_ID    |CURRENT_LATITUDE	    |CURRENT_LONGITUDE	    |
+-----------+-----------------------+-----------------------+
|45334      |37.7749		    |122.4194		    |
|45334      |37.7749    	    |122.4192		    |
|45334      |37.7747                |122.4190		    |
|45334      |37.7748    	    |122.4188	            |
```

#### Key Requirements
- Message keys
  - The CREATE STREAM and CREATE TABLE statements define streams and tables over data in Kafka topics. They allow you to specify which columns should be read from the Kafka message key, as opposed to the value, by using the KEY and PRIMARY KEY keywords, for streams and tables, respectively.

- Example
  ```
  CREATE TABLE users (
    userId INT PRIMARY KEY, -- userId will be read from the Kafka message key
    registertime BIGINT,    -- all other columns from the value
    gender VARCHAR,
    regionid VARCHAR
  ) WITH (
    KAFKA_TOPIC='users',
    VALUE_FORMAT='JSON'
  );
  ```
- While tables require a PRIMARY KEY, the KEY column of a stream is optional.
- Joins involving tables can be joined to the table on the PRIMARY KEY column. Joins involving streams have no such limitation. Stream joins on any expression other than the stream's KEY column require an internal repartition, but joins on the stream's KEY column do not.


#### Examples

```
-- Example timestamp format: yyyy-MM-dd'T'HH:mm:ssX
CREATE STREAM TEST (id BIGINT KEY, event_timestamp VARCHAR)
  WITH (
    kafka_topic='test_topic',
    value_format='JSON',
    timestamp='event_timestamp',
    timestamp_format='yyyy-MM-dd''T''HH:mm:ssX'
  );

  -- Create a stream on the original topic without a KEY columns:
  CREATE STREAM users_with_wrong_key (
       userid STRING,
       username VARCHAR,
       email VARCHAR
    ) WITH (
       KAFKA_TOPIC='users',
       VALUE_FORMAT='JSON'
    );

  -- Derive a new stream with the required key changes.
  -- 1) The CAST statement converts userId to the required SQL type.
  -- 2) The PARTITION BY clause re-partitions the stream based on the new, converted key.
  -- 3) The SELECT clause selects the required value columns, all in this case.
  -- The resulting schema is: USERID INT KEY, USERNAME STRING, EMAIL STRING.
  CREATE STREAM users_with_proper_key
    WITH (KAFKA_TOPIC='users-with-proper-key') AS
    SELECT
      CAST(userid AS BIGINT) AS userId,
      userName,
      email
    FROM users_with_wrong_key
    PARTITION BY CAST(userid AS BIGINT)
    EMIT CHANGES;

  -- Now you can create the table on the properly keyed stream.
  CREATE TABLE users_table (
      userId INT PRIMARY KEY,
      username VARCHAR,
      email VARCHAR
    ) WITH (
      KAFKA_TOPIC='users-with-proper-key',
      VALUE_FORMAT='JSON'
    );

  -- Or, if you prefer, you can keep userId in the value of the repartitioned data
  -- by using the AS_VALUE() function.
  -- The resulting schema is: USERID INT KEY, USERNAME STRING, EMAIL STRING, VUSERID INT
  CREATE STREAM users_with_proper_key_and_user_id
    WITH(KAFKA_TOPIC='users_with_proper_key_and_user_id') AS
    SELECT
      CAST(userid AS BIGINT) as USERID,
      username,
      email,
      AS_VALUE(CAST(userid AS BIGINT)) as vUserId,
    FROM users_with_wrong_key
    PARTITION BY CAST(userid AS BIGINT)
    EMIT CHANGES;

  -- Now you can create the table on the properly keyed stream.
  CREATE TABLE users_table_2 (
       userId INT PRIMARY KEY,
       username VARCHAR,
       email VARCHAR,
       vUserId INT
    ) WITH (
       KAFKA_TOPIC='users_with_proper_key_and_user_id',
       VALUE_FORMAT='JSON'
    );

CREATE TABLE users (
    userId INT PRIMARY KEY, -- userId will be read from the Kafka message key
    registertime BIGINT,    -- all other columns from the value
    gender VARCHAR,
    regionid VARCHAR
  ) WITH (
    KAFKA_TOPIC='users',
    VALUE_FORMAT='JSON'
  );
```


#### Resources

https://docs.ksqldb.io/en/latest/concepts/ksqldb-architecture/
https://docs.ksqldb.io/en/latest/developer-guide/syntax-reference/
https://www.confluent.io/blog/intro-to-ksqldb-sql-database-streaming/
