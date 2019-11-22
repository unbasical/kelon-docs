# Configuration

## Kelon (CLI)

Kelon already comes with a CLI. Just type `kelon --help` after you installed it.
```
usage: kelon [<flags>] <command> [<args> ...]

Kelon policy enforcer.

Flags:
  -h, --help                 Show context-sensitive help (also try --help-long and --help-man).
  -o, --opa-conf=./opa.yml   Path to the OPA configuration yaml.
  -r, --rego-dir=./policies  Dir containing .rego files which will be loaded into OPA.
      --path-prefix="/v1"    Prefix which is used to proxy OPA's Data-API.
  -p, --port=8181            Port on which the proxy endpoint is served.
      --envoy-port=9191      Also start Envoy GRPC-Proxy on specified port so integrate kelon with Istio.
      --envoy-dry-run        Enable/Disable the dry run feature of the envoy-proxy.
      --envoy-reflection     Enable/Disable the reflection feature of the envoy-proxy.
  -d, --datastore-conf=./datastore.yml  
                             Path to the datastore configuration yaml.
  -a, --api-conf=./api.yml   Path to the api configuration yaml.
      --config-watcher-path=./policies  
                             Path where the config watcher should listen for changes.
      --version              Show application version.

Commands:
  help [<command>...]
    Show help.

  start
    Start kelon in production mode.

  debug
    Enable debug mode.
```

In addition to that Kelon provides the possibility to be configured via environment variables. This may be handy if you want to run it inside a container.

|Flag|Short|Environment|Default|Type|
|----|-----|-----------|-------|----|
|--opa-conf|-o|OPA_CONF|./opa.yml|Existing File|
|--rego-dir|-r|REGO_DIR|./policies|Existing Dir|
|--path-prefix||PATH_PREFIX|/v1|String|
|--port|-p|PORT|8181|Number|
|--envoy-port||ENVOY_PORT|9191|Number|
|--envoy-dry-run||ENVOY_DRY_RUN|false|Boolean|
|--envoy-reflection||ENVOY_REFLECTION|false|Boolean|
|--datastore-conf|-d|DATASTORE_CONF|./datastore.yml|Existing File|
|--api-conf|-a|API_CONF|./api.yml|Existing File|
|--config-watcher-path||CONFIG_WATCHER_PATH|./policies|Existing Dir|

## datastore.yml

The `datastore.yml` defines all available datastores and their entities.

```yaml
# List of all datastores which kelon should be able to connect to
# Each datastore has to have an entity schema configured!
datastores:

  # Datastore's alias (it will be used throughout all your policies).
  mysql:
  
    # Datastore type
    # Kelon supports [mysql, postgres, mongo]
    [type: <string>]

    # Properties that are needed to connect to the datastore
    connection:     
    
      # Hostname or IP of the database
      [host: <string>]

      # Port under which the database is available
      [port: <int>]

      # Name of the database to connect to
      # Many database servers run multiple databases.
      # In this case create one datastore for each database
      [database: <string>]

      # The user which is used for the database connection
      [user: <string>]

      # The password of the user
      [password: <string>]


# Entity-Schemas define the entities of one schema inside a datastore.
# An entity_schema has several purposes:
#   => Help kelon to detect misspellings in regos
#   => Create entity-aliases for complex table-names
#   => Define the structure of nested entities (Just for No-SQL DBs)
entity_schemas:
  
  # Datastore's alias
  # This example shows how each SQL-Datastore should be configured.
  mysql:

    # Schema name
    # Kelon automatically adds the schema name to entities
    # This means that 'data.mysql.users' inside a rego will be translated 
    # to 'appstore.users' in the resulting SQL-Statement.
    #
    # !!! The entities inside all schemas of one Datastore have to be disjunct !!!
    # Otherwise kelon will fail on startup
    appstore:

      # List of all entities of schema 'appstore'
      entities:

        # Entity with the name inside the datastore
        - [name: <string>]

          # And an optional alias (used inside regos)
          [alias: <string> | optional]
  

  # This example shows how each No-SQL-Datastore should be configured.
  mongo:

    # Schema name
    # Because No-SQL-Datastores don't have concept for schemas, 
    # Kelon ignores the schema name for i.e. a MongoDB.
    #
    # !!! The entities inside all schemas of one Datastore have to be disjunct !!!
    # Otherwise kelon will fail on startup
    appstore:

      # List of all entities (in case of MongoDB each entity is a collection)
      entities:

        # Collection with it's name inside the datastore
        - [name: <string>]

          # Collection's optinal alias (used inside regos)
          [alias: <string> | optional]

          # Nested entities (Only supported for No-SQL-Datastores)
          # The depth of nesting entities is unlimited.
          # It makes no difference if the nested entities are arrays or objects
          entities:

            # Name of nested entity
            - [name: <string>]
              # Collection's optinal alias (used inside regos)
              [alias: <string> | optional]
```

## call-operands/{datastore-type}.yml

Open Policy Agent has [builtin call operands](https://www.openpolicyagent.org/docs/latest/policy-language/#operators) which may not be the same as the ones of each datastore.
Therefore each datastore-type has to have a mapping-file inside the folder 'call-operands'. The name of each file inside this directory has to match the type of the datastore.

```yaml
# Call operands map OPA's functions to datastore-native ones.
# You find more advanced files in directory /call-operands
call-operands:
  
  # Operand name inside OPA's context
  - [op: <string>]

    # Number of arguments the function call should take
    # Used for validation
    [args: <int>]

    # Mapping which mappes the OPA-native call to a Datastore-native call.
    # This is done by defining a pattern where all occurences of $<arg-pos>
    # are replaced with the original argument at position <arg-pos> (zero based).
    [mapping: <string>]
```

## api.yml

The `datastore.yml` contains all necessary configuration kelon needs to map incoming api-requests to OPA-Queries executed inside datastores.

```yaml
# Mappings for the APIs that your services expose
apis:
  
  # API-Collection
  # Each collection can have a path prefix which is prepended to every API-Mapping
  # inside the collection.
  - [path-prefix: <URL|string>]

    # All rego-queries mapped by this API-collection will include 
    # this datastore-alias as unknown.
    # This means that the regos this collection targets can contain
    # something like 'data.<datastore-alias>.<entity>.<attribute>'
    [datastore: <datastore-alias|string>]

    # Path-Mappings for incoming paths
    # (the most specific mapping is picked in case of multiple mappings)
    mappings:

      # Path-Mapping
      # Each mapping has to specify a path (regex) and a package which is used
      # by kelon to generate the target package of an mapped query.
      - [path: <regex|string>] 
        
        # OPA's target package of this Path-Mapping
        [package: <string>]
        
        # HTTP-Methods this path should map (optional)
        # If no methods are specified, kellon will match [GET,PUT,POST,DELETE,PATCH]
        methods:
          - [<string> ...]
```


## opa.yml

Internally used by Open Policy Agent. Klick [here](https://www.openpolicyagent.org/docs/latest/configuration/) for more information.

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