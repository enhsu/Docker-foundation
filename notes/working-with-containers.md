# Working with Containers

## Purpose

- Starting & stopping containers
- Publishing ports
- Viewing logs
- Executing commands in containers
- Removing containers
- Persisting data using volumes
- Sharing source code

## Navigation

1. [Starting Containers](#starting-containers)
1. [Viewing the Logs](#viewing-the-logs)
1. [Publishing Ports](#publishing-ports)
1. [Executing Commands in Running Containers](#executing-commands-in-running-containers)
1. [Stopping and Starting Containers](#stopping-and-starting-containers)
1. [Removing Containers](#removing-containers)
1. [Persisting Data using Volumes](#persisting-data-using-volumes)
1. [Copying Files between the Host and Containers](#copying-files-between-the-host-and-containers)
1. [Sharing the Source Code with a Container](#sharing-the-source-code-with-a-container)

## Starting Containers

`run` command, run a command in a new container

- `-d` option, run container in background and print container ID
- `-name` option, assign a name to the container. It's easier to work with than a randomly generated name by Docker

```bash
$ docker run -d --name maverick react-app
```

```bash
# see the process
$ docker ps
```

| CONTAINER ID | IMAGE     | COMMAND     | CREATED       | STATUS       | PORTS | NAMES    |
| ------------ | --------- | ----------- | ------------- | ------------ | ----- | -------- |
| 64dc0382ec55 | react-app | "npm start" | 6 seconds ago | Up 5 seconds |       | maverick |

## Viewing the Logs

1. Now we have container running in the background, but we don't know what's going on inside these containers
1. We use `logs` command, fetch the logs of a container

   - `-f` option, follow log output. It's useful if the container is continuously producing output
   - `-n` option, Number of lines to show from the end of the logs (default "all")
   - `-t` option, Show timestamps

   ```bash
   # log the container we just run
   $ docker logs -f 64d
   ```

## Publishing Ports

1. We can't access the application in the container on `http://localhost:3000`
1. Because port 3000 is publish on the container, not on the host
1. We can use `-p` option in `run` command, publish a container's port(s) to the host

   ```bash
   # `-p` optioin, 3000<local port>:3000<container port>
   $ docker run -d -p 3000:3000 --name maverick react-app
   ```

   And at docker process, we can see the port

   | CONTAINER ID | IMAGE     | COMMAND     | CREATED        | STATUS        | PORTS                  | NAMES    |
   | ------------ | --------- | ----------- | -------------- | ------------- | ---------------------- | -------- |
   | eff68244f453 | react-app | "npm start" | 27 seconds ago | Up 26 seconds | 0.0.0.0:3000->3000/tcp | maverick |

## Executing Commands in Running Containers

1. We have `exec`, for running a command in a running container
1. Difference between `docker exec` and `docker run`

   | `$ docker exec`                              | `$ docker run`                             |
   | -------------------------------------------- | ------------------------------------------ |
   | We executed a command in a running container | We start a new container and run a command |

1. Try to list all files in `maverick` container

   ```bash
   $ docker exec maverick ls
   ```

1. Or we can iteractive it with shell
   ```bash
   $ docker exec -it maverick sh
   ```

## Stopping and Starting Containers

1. We can use `stop` command to stop container
   ```bash
   $ docker stop maverick
   ```
1. And use `start` to start one or more stopped containers

   ```bash
   $ docker start maverick
   ```

   Difference between `docker start` and `docker run`

   | `$ docker start`          | `$ docker run`        |
   | ------------------------- | --------------------- |
   | Start a stopped container | Start a new container |

## Removing Containers

1. There are 2 ways to remove a container

   1. `docker container rm [container]`
   1. `docker rm [container]`

   NOTE: We can't remove a running container. We have to stop a container before we remove it, or use the `-f` option

   ```bash
   $ docker rm -f [container]
   ```

1. Also we can use `prune` to remove all stopped containers
1. To check all containers, we can use `ps` command
   ```bash
   $ docker ps -a
   ```

## Persisting Data using Volumes

1.  Each container has it's own file system, that is invisible to other containers. And that means if we delete one dontainer, it's file system will also go with it and we'll lose our data
1.  We should NEVER store our data in a container's file sysmte, instead we use `volume`
1.  A `volume` is a storage outside of containers, it can be a directory on the host, or somewhere in the cloud
1.  We use `volume` command handle volume

    ```bash
    # Create a volumne named app-data
    $ docker volume create app-data
    # Display detailed information on app-data
    $ docker volume inspect app-data
    ```

    We can see the information on app-data

    ```json
    [
      {
        "CreatedAt": "2022-09-19T11:53:32Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/app-data/_data",
        "Name": "app-data",
        "Options": {},
        "Scope": "local"
      }
    ]
    ```

    1. `Driver`
       `local` is the default value, it means this is a directory on the host.
    1. `Mountpoint`
       The value is where that directory is created on the host

       Docker on Mac runs inside a lightweight Linux virtual machine, so it's display a path inside the virtual machine

1.  `-v` option in `run` command, use the volume in the container

    ```bash
    # `-v` <volume_name>:<absolute_path in the file system of the container>
    # if `volume_name` or `absolute_path` not exist, it will create a new one automatically

    # create v0 container
    $ docker run -d -p 4000:3000 -v app-data:/app/data --name v0 react-app

    # create v1 container, to check is volume work
    $ docker run -d -p 4001:3000 -v app-data:/app/data --name v1 react-app
    ```

    NOTE: If we let Docker automatically create directory for us, Docker will use `root` user. If we want to access the file by `app` user, we'll get permission error. We can update `Dockerfile`, creating the mapping directory manually to prevent this issue.

    ```dockerfile
    # ... some stuff
    USER app
    WORKDIR /app
    # make sure create the mapping directory by `app` user
    RUN mkdir data
    # ... some stuff
    ```

    And don't forget to rebuild the image

    ```bash
    $ docker build -t react-app .
    ```

1.  Create some data in container: v0

    ```bash
    $ docker exec -it v0 sh
    ```

    In v0 continaer, add some data in the container

    ```bash
    $ cd /app/data && echo data >> data.txt
    ```

1.  Try to get the data in container: v1

    ```bash
    $ docker exec -it v1 sh
    ```

    In v1 container

    ```bash
    $ cat /app/data/data.txt
    # data
    ```

1.  Volume is the right way to persist data in Docker as application, because `volume` and `container` have different life cycles

## Copying Files between the Host and Containers

1. Sometimes we need to copy files between the host and a container
1. We can use `cp` command
   Copy file from container to host

   ```bash
   # copy file from container to host
   # $ docker cp [container]:[container/file/path/] [destination/host/path]
   $ docker cp 447f133635d9:/app/data/data.txt .
   ```

   Copy file from host to container

   ```bash
   # copy file from host to container
   # $ docker cp [host/file/path] [container]:[destination/container/path]
   $ docker cp ./some-file.txt 447f133635d9:/app/data
   ```

## Sharing the Source Code with a Container

1. Handle pulishing changes
   - For productioin machine: always build a new image
   - For development machine
     - ~~Build a new image~~
     - ~~Copy files~~
     - V: Create a `mapping` between `a directory on the host` and `a directory inside the container`
1. Use `volume` for mapping
   ```bash
   $ docker run -d -p 5001:3000 -v $(pwd):/app --name dev-mapping react-app
   ```
1. In that way, while we change the file in the host, the application in the host gets automatically update
1. Log the container, so we can see the changes as they come up
   ```bash
   $ docker logs -f dev-mapping
   ```
