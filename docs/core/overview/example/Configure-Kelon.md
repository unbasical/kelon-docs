# Configure Kelon

With all the previous written policies in mind, we now have to tell Kelon how to find its way from incoming requests all the way down to  . The configuration process of kelon has three basic parts:

1. datastore.yml
    - Configures datastore connections and their entity-schemas
2. api.yml
    - Mappings from incomin requests to a query fired against all policies inside a package
3. opa.yml
    - Configuration for the internally running Open Policy Agent

## datastore.yml

In order to enable Kelon to connect to your datastores, you need to configure the connections as well as the internal entity-structure of all your datastores inside the `datastore.yml` config.

```yaml
# Datastores to connect to
datastores:
  mysql:
    type: mysql
    connection:
      host: localhost # (mysql in case of the docker-compose example)
      port: 3306
      database: appstore
      user: You
      password: SuperSecure

  pg:
    type: postgres
    connection:
      host: localhost # (postgres in case of the docker-compose example)
      port: 5432
      database: appstore
      user: You
      password: SuperSecure
      sslmode: disable

  mongo:
    type: mongo
    connection:
      host: localhost # (mongo in case of the docker-compose example)
      port: 27017
      database: appstore
      user: You
      password: SuperSecure

# Entity-Schemas define the structure of the entities of one schema inside a datastore
entity_schemas:

  mysql:
    appstore:
      entities:
        - name: users
        - name: app_rights
        - name: apps
        - name: app_tags
        - name: tags

  pg:
    appstore:
      entities:
        - name: users
        - name: app_rights
        - name: apps
        - name: app_tags
        - name: tags

  mongo:
    appstore:
      entities:
        - name: users
        - name: apps
          entities:
            - name: rights
              entities:
                - name: user
                  alias: users
```

While the definition of an entity_schema for a SQL-Datastore is straight forward (Just list all entities of each datastore), creating a entity_schema for a No-SQL-Datastore (like MongoDB) ist a bit more difficult. Here you need to map the Structure of your nested entities so that Kelon can use this nested structure to find the right query-path inside your No-SQL-Datastore when joining entities inside your policies.

There is also the possibility to add more connection options for each datastore or use environment variables as yaml values. Please see the [Operations Guide](/operations/Configuration/#datastoreyml) for more information.

## api.yml

In the end all your services, which you want to secure with Kelon, receive client requests to a lot of different endpoints. To tell Kelon which policy it should use for which endpoint, you need to create the `api.yml` config. Inside this configuration you can define all your different APIs which will ultimately route a incoming request (Method, Path, Body) to a OPA-Package and Datastore.

```yaml
apis:
  # Route all requests starting with /api/mysql to MySQL
  - path-prefix: /api/mysql
    datastore: mysql
    mappings:
      - path: /apps/.*
        package: applications.mysql
  # Route all requests starting with /api/mongo to MongoDB
  - path-prefix: /api/mongo
    datastore: mongo
    mappings:
      - path: /apps/.*
        package: applications.mongo
  # All other requests are routed to PostgreSQL
  - path-prefix: /api/postgres
    datastore: pg
    mappings:
      - path: /apps/.*
        package: applications.pg
```

## call-operands

A core feature of Kelon is to map the result of OPA's partial evaluated policies to native datastore queries. To keep the adoption to different datastore-platforms as flexible as possible, all [builtin call operands](https://www.openpolicyagent.org/docs/latest/policy-language/#operators) from the OPA's Policy-Language are mapped via mappings-files which are located in the `call-operands` directory. Each mapping file is used for one datastory type and is therefore named with the pattern `call-operands/{datastore-type}.yml`.

### call-operands/mysql.yml & call-operands/postgres.yml

Each internal function essentially consists of a operator (op) and its arguments. To map this function to a datastore-native query function, you need to define the original functions-operator and the amount of expected arguments as well as the resulting mapping inside the native query.

```yaml
call-operands:
  # Mathematical operands
  #  ...
  # Relational operands
  - op: eq
    args: 2
    mapping: "$0 = $1"
  - op: equal
    args: 2
    mapping: "$0 = $1"
  - op: neq
    args: 2
    mapping: "$0 != $1"
  - op: lt
    args: 2
    mapping: "$0 < $1"
  - op: gt
    args: 2
    mapping: "$0 > $1"
  - op: lte
    args: 2
    mapping: "$0 <= $1"
  - op: gte
    args: 2
    mapping: "$0 >= $1"
  # Mathematical Functions
  # ...
```

Let's take the mapping `gte` from above. The builtin call `gte(a, b)` will be mapped to `a >= b` inside your native SQL-Query.

### call-operands/mongo.yml

```yaml
call-operands:
  # Relational operands
  - op: eq
    args: 2
    mapping: "$0: $1"
  - op: equal
    args: 2
    mapping: "$0: $1"
  - op: neq
    args: 2
    mapping: "$0: { \"$ne\": $1 }"
  - op: lt
    args: 2
    mapping: "$0: { \"$lt\": $1 }"
  - op: gt
    args: 2
    mapping: "$0: { \"$gt\": $1 }"
  - op: lte
    args: 2
    mapping: "$0: { \"$lte\": $1 }"
  - op: gte
    args: 2
    mapping: "$0: { \"$gte\": $1 }"
```

## opa.yml

Becaues Kelon is just a wrapper around the Open Policy Agent, you can also [configure](https://www.openpolicyagent.org/docs/latest/configuration/) the internally used OPA itself.

```yaml
labels:
  app: Kelon
  region: europe-west3
  environment: development

decision_logs:
  console: true
  reporting:
    min_delay_seconds: 300
    max_delay_seconds: 600

default_decision: /http/authz/allow
```