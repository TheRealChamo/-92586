# Docker: MySQL & phpMyAdmin #



## Requirements ##

* [Docker Desktop](https://www.docker.com/products/docker-desktop)

## Setting up the development environment ##

You can use the default settings, but sometimes it is recommended to modify the settings to match the production server. The configuration is located in the `.env` file with the following options:

* `MYSQL_VERSION` MySQL Version ([MySQL available versions](https://hub.docker.com/_/mysql)).
* `MYSQL_USER` Username to connect to MySQL.
* `MYSQL_PASSWORD` Password to connect to MySQL.
* `MYSQL_DATABASE` Default database name.

## Install the development environment ##

Installation should be done on command line:

```zsh
docker-compose up -d
```
You can validate that it has been installed correctly by accessing: [http://localhost/info.php](http://localhost/info.php)

## Available commands ##

Once installed, you can use the following commands:

```zsh
docker-compose start    # Start the development environment
docker-compose stop     # Stop the development environment
docker-compose down     # Stop and delete the development environment
```

## Files Structure ##

* `/docker/` contains the Docker configuration files.

## Access ##



### Web ###

* http://localhost/

### Database ###

There are two domains to connect to the database.

* `mysql`: for connection from PHP files.
* `localhost`: for external connections to the container.

The default credentials for the connection are:

| User | Password | Database |
|:---:|:---:|:---:|
| dbuser | dbpass | dbname |


## Note ##