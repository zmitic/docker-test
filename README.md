# Symfony Docker

A [Docker](https://www.docker.com/)-based installer and runtime for the [Symfony](https://symfony.com) web framework, with full [HTTP/2](https://symfony.com/doc/current/weblink.html), HTTP/3 and HTTPS support.

![CI](https://github.com/dunglas/symfony-docker/workflows/CI/badge.svg)

## Getting Started

1. If not already done, [install Docker Compose](https://docs.docker.com/compose/install/)
2. Run `docker-compose build --pull --no-cache` to build fresh images
3. Run `docker-compose up` (the logs will be displayed in the current shell)
4. Open `https://localhost` in your favorite web browser and [accept the auto-generated TLS certificate](https://stackoverflow.com/a/15076602/1352334)

## Communication between containers using their host name

Services inside a docker-compose file can communicate with each other using the service names, but when we run two instances of a docker-compose file, services inside docker-compose cannot communicate with the services inside the other docker-compose file until we create a shared network for them.
Then the services of each docker-compose file can access the other docker-compose file services using their service names(or hostnames).

### Add network
First we should add a docker network which will be used by all the instances of docker-compose files. To create the new network run:

```
docker network create --driver=bridge mynetwork
```

### Set the default network for all the docker-compose services
Now, we set the network we just created as the default network for the docker compose services. Edit the docker-compose file and add the following lines to the end of the file:

```
networks:
  default:
    external:
      name: mynetwork
```
### Set a hostname for the services
Now with the applied config, when we run the `docker-compose up` command, we can access to services from another running instance of the docker-compose with service names like **docker-test2_caddy_1**. so we can for example access to the first running instance with:

```
curl http://docker-test1_caddy_1
```
We want to access the first instance, using the name *first.wip* instead of *docker-test1_caddy_1*. Edit the docker-compose file and set the ***hostname*** for the *caddy* service:
```
  caddy:
    hostname: ${HOST_NAME}   # added this line
    build:
      context: .
      target: symfony_caddy
    ...
```

Then we can run the docker-compose:
```
# First server
SERVER_NAME=":80" HOST_NAME="first.wip" docker-compose up -d
# Second server
SERVER_NAME=":80" HOST_NAME="second.wip" docker-compose up -d
```

Now we are able to access the first server from the second server:

```
docker exec -it docker-test2_php_1 sh
```
and then:
```
curl http://first.wip
```

## Features

* Production, development and CI ready
* Automatic HTTPS (in dev and in prod!)
* HTTP/2, HTTP/3 and [Preload](https://symfony.com/doc/current/web_link.html) support
* Built-in [Mercure](https://symfony.com/doc/current/mercure.html) hub
* [Vulcain](https://vulcain.rocks) support
* Just 2 services (PHP FPM and Caddy server)
* Super-readable configuration

**Enjoy!**

## Docs

1. [Build options](docs/build.md)
2. [Using Symfony Docker with an existing project](docs/existing-project.md)
3. [Support for extra services](docs/extra-services.md)
4. [Deploying in production](docs/production.md)
5. [Installing Xdebug](docs/xdebug.md)
6. [Troubleshooting](docs/troubleshooting.md)

## Credits

Created by [Kévin Dunglas](https://dunglas.fr), co-maintained by [Maxime Helias](https://twitter.com/maxhelias) and sponsored by [Les-Tilleuls.coop](https://les-tilleuls.coop).
