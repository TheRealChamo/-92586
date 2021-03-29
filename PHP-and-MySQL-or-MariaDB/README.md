# Docker: PHP & MySQL or MariaDB #

This project is a Quick Start Guide to deploy a local test enviroment to work with [PHP](https://www.php.net/) and [MySQL](https://www.mysql.com/) or [MariaDB](https://mariadb.org/) using [Docker](https://www.docker.com). 

Use *Docker* is simple, but there are so many images, versions and ways to create the containers that make this task tedious. This project offers a quick installation, with standard versions and with the minimum amount of modifications to the Docker images. It comes configured with `PHP 7.3` y `MySQL 5.7`.

## Requirements ##

* [Docker Desktop](https://www.docker.com/products/docker-desktop)

## Setting up the development environment ##

You can use the default settings, but sometimes it is recommended to modify the settings to match the production server. The configuration is located in the `.env` file with the following options:

* `PHP_VERSION` PHP version ([PHP available versions](https://github.com/docker-library/docs/blob/master/php/README.md#supported-tags-and-respective-dockerfile-links)).
* `PHP_PORT` Web Server Port.
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
* `/www/` folder for the project's PHP files.

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

In each of the commands that you will in this guide you can substitute `mysql` for `mariadb` and it will work in the same way.

To use the `mariadb` image in Docker, the first thing to do is download it to your computer:

```zsh
docker pull mariadb
```
This command will download the latest version of the image that is available in the docker image repository, if you need a specific version you can use:

```zsh
docker pull mariadb:[tag_version]
```
Then you should modify the `**docker-compose.yml**` file

After that, you only need to execute the command
```zsh
docker-compose up -d
```

And It's Done! Let's try