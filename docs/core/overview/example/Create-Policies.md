# Create Policies

In order to enable Kelon to make decisions on incoming requests, you have to write policies in OPA's own [Policy-Language](https://www.openpolicyagent.org/docs/latest/policy-language/) which you can put in different Rego-Files. To keep your policies organized, you can define a package per Rego-File which is also the central way to address all policies inside this package. As a result of that you should keep your package-names clean and in some way connected to the API you want to secure with Kelon.


For this example we want to create four simple policies which show all basic capabilities of kelon:

1. Users with right 'OWNER' on app can access it always
    * Endpoint: GET /api/{datastore-alias}/apps/:app_id
2. All apps with 5 stars are public
    * Endpoint: GET /api/{datastore-alias}/apps/:app_id
3. The first app is public
    * Endpoint: GET /api/{datastore-alias}/apps/:app_id
4. All users that are a friends of Kevin are allowed see everything
    * Endpoint: GET /api/{datastore-alias}/*

## Custom built-ins
Due to Kelon's architecture, the [print statement](https://www.openpolicyagent.org/docs/latest/policy-reference/#debugging) provided by OPA is not working. But we provide other logging functionality which smoothly integrate into Kelon's logs. These logging functions work similar to the original print statement.
* log_info
* log_debug
* log_warn
* log_error
* log_fatal

## SQL

### policies/mysql_applications.rego

The file `policies/pg_applications.rego` has almost the same content. The only difference is that each "mysql" is replaced by a "pg" (which only occurs, because of this simple szenario).

```rego
package applications.mysql

# First App does not need authentication
verify = true {
    input.path == ["api", "mysql", "apps", "1"]
}

# All other apps need an authenticated user
verify = true {
    some user

    data.mysql.users[user].name == input.user
    user.password = input.password
}

# Deny all by default
allow = false

# Path: GET /api/mysql/apps/:app_id
# Users with right 'OWNER' on app can access it always
allow = true {
    some appId, u, r
    input.method == "GET"
    input.path = ["api", "mysql", "apps", appId]

    # Join
    data.mysql.users[u].id == data.mysql.app_rights[r].user_id

    # Where
    u.name == input.user
    r.right == "OWNER"
    r.app_id == appId
}

# Path: GET /api/mysql/apps/:app_id
# All apps with 5 stars are public
allow = true {
    some app, appId
    input.method == "GET"
    input.path = ["api", "mysql", "apps", appId]

    data.mysql.apps[app].id == appId
    app.stars == 5
}

# Path: GET /api/mysql/apps/:app_id
# The first app is public
allow = true {
    input.method == "GET"
    input.path == ["api", "mysql", "apps", "1"]
}

# Path: GET <any>
# All users that are a friends of Kevin are allowed see everything
allow = true {
    some user
    input.method == "GET"

    # Query
    data.mysql.users[user].name == input.user
    old_or_kevin(user.age, user.friend)
}

old_or_kevin(age, friend) {
    age == 42
}

old_or_kevin(age, friend) {
    friend == "Kevin"
}
```

With a closer look at the first policy in the above Rego you will notice, that the data inside the tables of your datastore can be 'magically' accessed inside Regos with the following Syntax:

`data.{datastore-alias}.{entity}.{column}`

Additianally the join between the table `users` and `app_rights` on the condition 

`users.id == app_rights.user_id` 

is done with the statements
```rego
    data.mysql.users[u].id == data.mysql.app_rights[r].user_id
    u.name == input.user
    r.right == "OWNER"
    r.app_id == appId

    # SELECT count(*) FROM users
    #   INNER JOIN app_rights
    #       ON users.id = app_rights.user_id
    # WHERE users.name = {input.user}
    #   AND app_rights.right = 'OWNER'
    #   AND app_rights.app_id = {appId}
```

## No-SQL

### policies/mongo_applications.rego

Writing policies for No-SQL datastores is nearly the same as writing them for SQL-Datastores. You can just write a join (as if it was for a SQL-Datastore) which is translated by kelon using your [configured entity_schema](./Configure-Kelon.md)

The only difference to writing policies in comparison to SQL-Datastores is:

1. The **first entity** of your policy **must be** the the **collection** (in case of MongoDB) you want to address
2. The **Join-Condition is ignored** (You still have to add any condition for now...)

```rego
package applications.mongo

# Deny all by default
allow = false

# Path: GET /api/mongo/apps/:app_id
# Users with right 'OWNER' on app can access it always
allow = true {
    input.method == "GET"
    input.path = ["api", "mongo", "apps", appId]

    # This query fires against collection -> apps
    data.mongo.apps[app].id == appId

    # Nest elements
    data.mongo.rights[right].id == app.id
    data.mongo.users[user].id == right.id

    # Query root
    app.stars > 2

    # Query nested
    right.right == "OWNER"
    user.name == input.user
}

...
```

## Common policy patterns

Following examples show basic policy patterns which you can use to write your policies

### Join single entity

```rego
# Datastore-Alias:          mysql
# Table-Join:               users -> app_rights

data.mysql.users[user].id == data.mysql.app_rights[appRight].user_id
```

### Join multiple entities

```rego
# Datastore-Alias:          mysql
# Table-Join:               users -> app_rights -> apps

data.mysql.users[user].id == data.mysql.app_rights[appRight].user_id
data.mysql.apps[app].id == appRights.app_id
```

### Self-Join

Self-Join is currently not supported by Kelon!