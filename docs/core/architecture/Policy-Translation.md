# Policy Translation

## Kelon in a Nutshell

Internally each request sent to Kelon goes through the same processing chain to finally become a translated native datastore-query.
Following info graphic shows all configuration files and their interaction throughout this chain.

![Kelon_in_a_nutshell](/img/kelon/Kelon_In_A_Nutshell.png)

## Deconstructing the example

To become a deep understanding of how Kelon works in action, we now deconstruct the entire internal process of Kelon while executing the example setup provided inside the repository. Therefore the following infographic shows an example query and its entire way through Kelon.

## (My)SQL

![Example_Query_MySQL](/img/kelon/Example_Query_MySQL.png)

The example for PostgreSQL is nearly the same (just swap 'mysql' with 'pg').

## And now No-SQL

When we run the example above against the MongoDB, there is also not much difference in each processing step.
Only the generated output are now following native MongoDB-Query:

```js
users.find({ 
    "$or": [ 
        {
            "name": "Arnold", 
            "friend": "Kevin"
        } 
    ] 
})

apps.find({ 
    "$or": [ 
        {
            "id": 2, 
            "stars": { 
                "$gt": 2 
            }, 
            "rights.right": "OWNER", 
            "rights.user.name": "Arnold"
        }, 
        {
            "stars": 5, 
            "id": 2
        } 
    ]
})
```