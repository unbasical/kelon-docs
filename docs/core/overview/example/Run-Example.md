# Run the example

## Install kelon 

(not needed if you want to use the docker-comopose file from the example)

### From source

```bash
# Clone the repository
$ git clone git@github.com:Foundato/kelon.git

# Build the container
$ docker build . -t kelon:latest

# Or install kelon on your local machine
$ go mod download
$ go build -o ./kelon ./cmd/kelon
```

### Docker image

```bash
$ docker pull kelonio/kelon
$ docker run kelonio/kelon --help
```

## Start the environment

The entire example environment can be started with the provided docker-compose in the project's root directory with following command

```bash
$ docker-compose up -d
```

This will start following components:

* Kelon
    * Port: 8181
* MySQL
    * Port: 3306
* PostgreSQL
    * Port: 5432
* MongoDB
    * Port: 27017-27019

After all containers are built and up, you can verify that everything is up by performing a valid request:

to PostgreSQL
```bash
$ curl --request POST \
  --url http://localhost:8181/v1/data \
  --header 'Content-Type: application/json' \
  --data '{"input": {"method": "GET", "path": "/api/pg/apps/2", "user": "Arnold", "password": "pw_arnold"}}'

> 200
```

to MySQL
```bash
$ curl --request POST \
  --url http://localhost:8181/v1/data \
  --header 'Content-Type: application/json' \
  --data '{"input": {"method": "GET", "path": "/api/mysql/apps/2", "user": "Arnold", "password": "pw_arnold"}}'

> 200
```

to MongoDB
```bash
$ curl --request POST \
  --url http://localhost:8181/v1/data \
  --header 'Content-Type: application/json' \
  --data '{"input": {"method": "GET", "path": "/api/mongo/apps/2", "user": "Arnold", "password": "pw_arnold"}}'

> 200
```

## Play around

You now have a complete setup of Kelon which you can use to play around.

You can i.e. try to access an app you shouldn't be allowed to:
```bash
$ curl --request POST \
  --url http://localhost:8181/v1/data \
  --header 'Content-Type: application/json' \
  --data '{"input": {"method": "GET", "path": "/api/pg/apps/2", "user": "Anyone", "password": "pw_anyone"}}'

> 403
```

Or access an app with 5 stars (which the user 'Anyone' is allowed to):
```bash
$ curl --request POST \
  --url http://localhost:8181/v1/data \
  --header 'Content-Type: application/json' \
  --data '{"input": {"method": "GET", "path": "/api/pg/apps/3", "user": "Anyone", "password": "pw_anyone"}}'

> 200
```

There is also a POSTMAN-COLLECTION in the example directory which contains a bunch of queries agains all three datasources (PostgreSQL, MySQL, MongoDB).
Feel free to play around with the existing regos and databases to get a feeling of how to write policies and configure Kelon.

## Good to know

Due to **Kelon's hot reloading of policies**, you can edit the policies which are mounted into the Docker-Container and directly see the result of your work within seconds. Please also check the Container-Logs. If you make any mistake while writing your policies, Kelon will let you know.

## Clean up

After you are done, just stop the entire setup by running:

```bash
$ docker-compose down
```