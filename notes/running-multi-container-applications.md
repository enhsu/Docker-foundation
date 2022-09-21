# Running Multi-container Applications

## Purpose

- Docker Compose
- Docker networking
- Database migration
- Running automated tests

## Navigation

1. [Installing Docker Compose](#installing-docker-compose)
1. [Cleaning Up our Workspace](#cleaning-up-our-workspace)
1. [The Sample Web Application](#the-sample-web-application)
1. [Creating a Compose File](#creating-a-compose-file)
1. [Building Images](#building-images)
1. [Starting and Stopping the Application](#starting-and-stopping-the-application)
1. [Docker Networking](#docker-networking)
1. [Viewing Logs](#viewing-logs)
1. [Publishing Changes](#publishing-changes)
1. [Migrating the Database](#migrating-the-database)
1. [Running Tests](#running-tests)

## Installing Docker Compose

[Reference: Install Docker Compose](https://docs.docker.com/compose/install/)

If you have Docker Desktop, you’ve got a full Docker installation, including Compose.

```bash
# Verify is docker compose installed
$ docker-compose --version
# Docker Compose version v2.10.2
```

## Cleaning Up our Workspace

How can we delete all containers and images

```bash
# Use `-q` option get all container ID as the arguments
# Use `-a` get the stopped containers as well
$ docker container rm -f $(docker container ls -aq)
# Do the same things to images
$ docker image rm -f $(docker image ls -q)
```

Or we can use Docker Desktop application, inside `Troubleshoot` > `Clean/Purge data`

## The Sample Web Application

We're looking at a application with multiple building blocks, `front-end`, `back-end`, and `database`

1. If we have a full stack project, with frontend, backend, and a database
1. There are a number of steps if we want to run this application outside of Docker.
   - Run backend
     1. go to backend folder
     1. run `npm i`
     1. start the webserver
   - Run frontend
     1. go to frontend folder
     1. ...
1. With Docker, we don't have to do any of these things
1. We can create a `docker-compose.yml` file in the root of the project, and use `docker-compose up` command to run the whole project

## Creating a Compose File

- [Reference: Quick start for YAML file](https://www.cloudbees.com/blog/yaml-tutorial-everything-you-need-get-started)
- [Reference: Docker compose file](https://docs.docker.com/compose/compose-file/)

That's create the `docker-compose.yml` file

1. First, set the `services` property
   ```yml
   services:
     frontend:
     backend:
     database:
   ```
   These name are arbitrary, so we can also name it like
   ```yml
   services:
     web:
     api:
     db:
   ```
1. The idea here is that we're defining every service, each service MAY also include a Build section, which defines how to create the Docker image for the service.
1. We need to tell Docker how to build a image for that service

   ```yml
   services:
     web:
       # build: where can find the Dockerfile
       build: ./frontend
       ports:
         - 3000:3000
     api:
       build: ./backend
       ports:
         - 3001:3001
       environment:
         DB_URL: mongodb://db/database_name
     db:
       # For database, we're not going to build a image. Instead, we're going to pull a image from docker hub
       # Choose linux version, not windows
       image: mongo:4.0.28-xenial
       # mongodb listent to port 27107 by defaul
       ports:
         - 27017:27017
       volumes:
         - database_name:/data/db
   # To reuse a volume across multiple services, a named volume MUST be declared in the top-level volumes key.
   volumes:
     database_name:
   ```

   - [Reference: environment](https://docs.docker.com/compose/compose-file/#environment)
   - [Reference: volumes](https://docs.docker.com/compose/compose-file/#volumes)

   NOTE: About the `version` property, the Compose spec merges the legacy 2.x and 3.x versions, aggregating properties across these formats and is implemented by Compose 1.27.0+. So don't worry about it

## Building Images

1. We can use `--help` option in `docker-compose` command, to see all the commands
1. In terminal, project root, withe `docker-compose.yml` file, we can simply build images by `build` command
   ```bash
   $ docker-compose build
   # Or we can full rebuild it by --no-cache option
   $ docker-compose build --nocache
   ```
1. We can get images like

   | REPOSITORY     | TAG    | IMAGE ID     | CREATED        | SIZE  |
   | -------------- | ------ | ------------ | -------------- | ----- |
   | vidly-frontend | latest | c42cf2de368e | 17 minutes ago | 298MB |
   | vidly-backend  | latest | afded62068b4 | 17 minutes ago | 183MB |

## Starting and Stopping the Application

1. Start by `up` command, in the project root
   ```bash
   $ docker-compose up
   ```
1. With `--build` option, we can rebuild the image everytime before start it
   ```bash
   $ docker-compose up --build
   ```
1. With `-d` option, we can run the application in the background
   ```bash
   $ docker-compose up -d
   ```
1. And we can use `ps` command, see all the containers relevant to the app

   NOTE: mongoDB will install at the first time the project is on

   ```bash
   $ docker-compose ps
   ```

   And we'll see

   | NAME             | COMMAND                | SERVICE  | STATUS  | PORTS                    |
   | ---------------- | ---------------------- | -------- | ------- | ------------------------ |
   | vidly-backend-1  | "docker-entrypoint.s…" | backend  | running | 0.0.0.0:3001->3001/tcp   |
   | vidly-db-1       | "docker-entrypoint.s…" | db       | running | 0.0.0.0:27017->27017/tcp |
   | vidly-frontend-1 | "docker-entrypoint.s…" | frontend | running | 0.0.0.0:3000->3000/tcp   |

1. Stop the project by `down` command
   ```bash
   $ docker-compose down
   ```

## Docker Networking

1. When we run the application with Docker compose, Docker compose will automatically create a network and add our containers on that network. So these containers can talk to each other
1. That's see the network

   ```bash
   # Running the app
   $ docker-compose up -d
   # Show all networks
   $ docker network ls
   ```

   And we can see `vidly_default`

   | NETWORK ID   | NAME          | DRIVER | SCOPE |
   | ------------ | ------------- | ------ | ----- |
   | d76d6ab26f80 | bridge        | bridge | local |
   | 80a931f992b1 | host          | host   | local |
   | 5f0bfbeaf2d3 | none          | null   | local |
   | 734c51e611bb | vidly_default | bridge | local |

   [Reference: network](https://docs.docker.com/network/)

1. What's Docker network
   Docker comes with an embedded `DNS server` that contains the name and IP of these containers

   Inside each `container`, we have a component called the `DNS resolver`. This DNS resolver talks to the DNS server to find the IP address of the target container.

   Based on the mechanism, the container can directly talk to the other container using the IP address.

   So each container has its' own IP address, and is part of the network

1. That's see the `docker-compose.yml` file
   ```yaml
   ...
   api:
     ...
     environment:
       DB_URL: mongodb://db/database_name
       # The `db` is the name of a host
       # The api container can talk to the db container, because both of these containers in this application are part of the same network
       # So the `db` host is only available inside the Docker environment
       # If we open up the browser, and go to localhost/db, we're not going to get anything
       # And if we want to access the `db` container, we need port mappings
   db:
     ...
     # That's the port mapping we need for access the `db` container
     # So that we can access the `db` container on `localhost/27017`
     ports:
       # map <from host>:<to container>
       - 27017:27017
   ```

## Viewing Logs

1. Use `logs` command, we can view the logs across all containers of this application in one place
   ```bash
   $ docker-compose logs
   ```
1. Also we can use `logs` in docker, to see specific container's log
   ```bash
   $ docker logs <container_id>
   ```

## Publishing Changes

1. We don't want to rebuild our application images every time we change our code. So we're going to map our project directory to the app directory inside our container
   ```yaml
   services:
     web:
       build: ./frontend
       ports:
         - 3000:3000
     api:
       build: ./backend
       ports:
         - 3001:3001
       # Add volumes for mapping project directory to app directory inside the container
       # Our nodemon will install in the container, but not in the project, so it'll get Error: `sh: nodemon: not found`
       # To solve the problem, go to backend directory, and run `$ npm i`
       volumes:
         - ./backend:/app
       environment:
         DB_URL: mongodb://db/database_name
     db:
       image: mongo:4.0.28-xenial
       ports:
         - 27017:27017
       volumes:
         - database_name:/data/db
   volumes:
     database_name:
   ```

## Migrating the Database

1. In `docker-compose.yml` file, we can override the `CMD` in the `Dockerfile`

   ```yaml
   ...
   api:
     ...
     command: migrate-mongo up && npm start
   ...
   ```

   The `command` will override the `CMD` in the backend folder's Dockerfile

   ```dockerfile
   ...
   CMD ["npm", "start"]
   ```

1. The question is that, it's possible our database is not ready at the time of executing `migrate-mongo up`. We can use waiting script to solve it
   Search for `docker wait for container`, See [docker startup order](https://docs.docker.com/compose/startup-order/) documentation

   We can see the solution uses a tool such as wait-for-it, dockerize, Wait4X, sh-compatible wait-for, or RelayAndContainers template.

   So we can update the `command`

   ```yaml
   ...
   api:
    ...
    command: ./wait-for db:27017 && migrate-mongo up && npm start
   ```

   And the `wait-for` script might looks like, [Reference: wait-for-it](https://github.com/vishnubob/wait-for-it)

   ```sh
    #!/bin/sh

    # The MIT License (MIT)
    #
    # Copyright (c) 2017 Eficode Oy
    #
    # Permission is hereby granted, free of charge, to any person obtaining a copy
    # of this software and associated documentation files (the "Software"), to deal
    # in the Software without restriction, including without limitation the rights
    # to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    # copies of the Software, and to permit persons to whom the Software is
    # furnished to do so, subject to the following conditions:
    #
    # The above copyright notice and this permission notice shall be included in all
    # copies or substantial portions of the Software.
    #
    # THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    # IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    # FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    # AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    # LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    # OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    # SOFTWARE.

    set -- "$@" -- "$TIMEOUT" "$QUIET" "$HOST" "$PORT" "$result"
    TIMEOUT=15
    QUIET=0

    echoerr() {
      if [ "$QUIET" -ne 1 ]; then printf "%s\n" "$*" 1>&2; fi
    }

    usage() {
      exitcode="$1"
      cat << USAGE >&2
    Usage:
      $cmdname host:port [-t timeout] [-- command args]
      -q | --quiet                        Do not output any status messages
      -t TIMEOUT | --timeout=timeout      Timeout in seconds, zero for no timeout
      -- COMMAND ARGS                     Execute command with args after the test finishes
    USAGE
      exit "$exitcode"
    }

    wait_for() {
    if ! command -v nc >/dev/null; then
        echoerr 'nc command is missing!'
        exit 1
      fi

      while :; do
        nc -z "$HOST" "$PORT" > /dev/null 2>&1

        result=$?
        if [ $result -eq 0 ] ; then
          if [ $# -gt 6 ] ; then
            for result in $(seq $(($# - 6))); do
              result=$1
              shift
              set -- "$@" "$result"
            done

            TIMEOUT=$2 QUIET=$3 HOST=$4 PORT=$5 result=$6
            shift 6
            exec "$@"
          fi
          exit 0
        fi

        if [ "$TIMEOUT" -le 0 ]; then
          break
        fi
        TIMEOUT=$((TIMEOUT - 1))

        sleep 1
      done
      echo "Operation timed out" >&2
      exit 1
    }

    while :; do
      case "$1" in
        *:* )
        HOST=$(printf "%s\n" "$1"| cut -d : -f 1)
        PORT=$(printf "%s\n" "$1"| cut -d : -f 2)
        shift 1
        ;;
        -q | --quiet)
        QUIET=1
        shift 1
        ;;
        -q-*)
        QUIET=0
        echoerr "Unknown option: $1"
        usage 1
        ;;
        -q*)
        QUIET=1
        result=$1
        shift 1
        set -- -"${result#-q}" "$@"
        ;;
        -t | --timeout)
        TIMEOUT="$2"
        shift 2
        ;;
        -t*)
        TIMEOUT="${1#-t}"
        shift 1
        ;;
        --timeout=*)
        TIMEOUT="${1#*=}"
        shift 1
        ;;
        --)
        shift
        break
        ;;
        --help)
        usage 0
        ;;
        -*)
        QUIET=0
        echoerr "Unknown option: $1"
        usage 1
        ;;
        *)
        QUIET=0
        echoerr "Unknown argument: $1"
        usage 1
        ;;
      esac
    done

    if ! [ "$TIMEOUT" -ge 0 ] 2>/dev/null; then
      echoerr "Error: invalid timeout '$TIMEOUT'"
      usage 3
    fi

    if [ "$HOST" = "" -o "$PORT" = "" ]; then
      echoerr "Error: you need to provide a host and port to test."
      usage 2
    fi

    wait_for "$@"
   ```

## Running Tests

1. One way to run the test: go to the project folder(e.g. frontend), and use `npm test` command
1. Also we can run the test inside the container

   ```yaml
   # The original setting
   web:
     build: ./frontend
     ports:
       - 3000:3000
     volumes:
       - ./frontend:/app
   ```

   ```yaml
   # Running test setting
   web-tests:
     image: vidly_web
     volumes:
       - ./frontend:/app
     command: npm test
   ```

1. NOTE: running the test inside the container is much more slower than just run it in the project
