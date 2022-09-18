# Building Images

## Purpose

- Creating Docker files
- Versioning images
- Sharing images
- Saving and loading images
- Reducing the images size
- Speeding up builds

## Navigation

1. [Images and Containers](#images-and-containers)
1. [Sample Web Application](#simple-web-application)
1. [Dockerfile Instructions](#dockerfile-instructions)
1. [Choosing the Right Base Image](#choosing-the-right-base-image)
1. [Copying Files and Directories](#copying-files-and-directories)
1. [Excluding Files and Directories](#excluding-files-and-directories)
1. [Running Commands](#running-commands)
1. [Setting Environment Variables](#setting-environment-variables)
1. [Exposing Ports](#exposing-ports)
1. [Setting the User](#setting-the-user)
1. [Defining Entrypoints](#defining-entrypoints)
1. [Speeding Up Builds](#speeding-up-builds)
1. [Removing Images](#removing-images)
1. [Tagging Images](#tagging-images)
1. [Sharing Images](#sharing-images)
1. [Saving and Loading Images](#saving-adn-loading-images)

## Images and Containers

An image includes everything an application needs to run

- Image
  - A cut-down OS
  - Third-party libraries
  - Application files
  - Environment variables

Once we have an image, we can start a container from it. Container is like a virtual machine

- Container
  - Provides an `isolated environment` fo executing an application
    - Container gets it's file system from the image. But each container has it's own right layer
    - Still there is a way to share database between containers
  - Can be stopped & restarted
  - Is just a operating system process

## Simple Web Application

1. Create a react project
   ```bash
   $ npx create-react-app my-app
   ```
1. Steps for start the project
   1. Install Node
   1. npm install
   1. npm start

## Dockerfile Instructions

[Reference: Dockerfile](https://docs.docker.com/engine/reference/builder/)

A Dockerfile contains instructions for building an image

- Dockerfile
  - [FROM](https://docs.docker.com/engine/reference/builder/#from)
    Which specifies the base image
  - WORKDIR
    Which specifies the working directory, once we use this command, all the following will be executed in the current working directory
  - [COPY](https://docs.docker.com/engine/reference/builder/#copy)
    For copying files and directories
  - [ADD](https://docs.docker.com/engine/reference/builder/#add)
  - [RUN](https://docs.docker.com/engine/reference/builder/#run)
    For executing operating system commands
  - [ENV](https://docs.docker.com/engine/reference/builder/#env)
    For setting environment variables
  - [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose)
    For telling Docker that our container is starting on a given port
  - [USER](https://docs.docker.com/engine/reference/builder/#user)
    For specifying the user that should run the application. Typically, we want to run our application using a user with limited privileges
  - [CMD](https://docs.docker.com/engine/reference/builder/#cmd)
  - [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint)
    For specifying the commands that should be executed when we start a container

## Choosing the Right Base Image

What we are going to do in this section is using `FROM` in `Dockerfile`

The final result is

```dockerfile
FROM node:18.9.0-alpine3.15
```

Build the image based on the Dockerfile we create

```bash
$ docker build -t react-app .
```

And run the container

```bash
$ docker run -it react-app sh
```

---

Instructions

1. Add a `Dockerfile` in the root of the project
1. The first instruction we're going to use is `FROM`
1. The base image can be an operating system like `Linux` or `Windows`. Or it can be an operating system plus a runtime environment. e.g. Javascript developer can start from a `Node` image
1. We can see docker samples [here](https://docs.docker.com/samples/)
1. The image can be in any registry, the default registry that Docker uses is `Docker Hub`. If we want to use the registry which is not in Docker Hub, we have to type the full URL.
1. We use `Node` for the project, and there are hundreds of node images on Docker Hub. We can choose one we want to use on [Docker Hub](https://hub.docker.com/_/node/tags) based on the tags.
   - Always use a specific version
   - DO NOT use the `latest`. Because if you build your application against the latest version of node, next time, there's a new version of it. Things can get unpredictable.
     ```dockerfile
     FROM node:latest
     ```
1. We can filter the node version by the `Filter Tags`.
1. Choose the image by the `OS/ARCH`, it means the `operating system` and `CPU architectures`
1. What we want is a small compressed size image now, so we can search `alpine` on the page.
1. `18.9.0-alpine3.15` is fine for me at the monent
1. The content of the `Dockerfile`
   ```dockerfile
   FROM node:18.9.0-alpine3.15
   ```
1. And we run the `build` command to build the image
   ```bash
   $ docker build -t react-app .
   ```
1. See the images we have
   ```bash
   $ docker images
   ```
1. And we can run the image

   ```bash
   $ docker run -it react-app sh
   ```

   alpine linux doesn't come with bash, actually it doesn't have many of the utilities we're familiar with, that's the reason why the compressed size is small.

   So we run it in Shell, the origin Shell program

## Copying Files and Directories

What we are going to do in this section is using `COPY` in `Dockerfile`

The final result is

```dockerfile
FROM node:18.9.0-alpine3.15
COPY . /app
```

Build the image based on the Dockerfile we create

```bash
$ docker build -t react-app .
```

And run the container

```bash
$ docker run -it react-app sh
```

---

Instructions

1. Now we havve a base image, the next step is to copy the application files into the image
1. So for that we have two instructions, one is `COPY`, the other is `ADD`. They have the same syntax, but `ADD` has a couple additioinal features
1. That's start with [COPY](https://docs.docker.com/engine/reference/builder/#copy), it has two forms
   ```dockerfile
   COPY [--chown=<user>:<group>] <src>... <dest>
   COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
   ```
1. The `src` should be inside your project's root directory
   When we run the `build` command

   ```bash
   $ docker build -t react-app .
   ```

   The last argument `.`, the period, means the current directory.

   So when we execute this command, `Docker client` sends the content of the current directory to `Docker engine`, this is called the build context.

   And Docker engine will execute the commands in `Dockerfile` one by one.

   So at that point, Docker engine doesn't have access to files out of the current directory

1. If the `dest` directory doesn't exist, Docker will automatically create it
1. We're going copy everything in the current directory into the `/app` directory

   ```dockerfile
   COPY . /app
   ```

   Also we can use relative path, if we set up the working directory first

   ```dockerfile
   WORKDIR /app
   COPY . .
   ```

1. If we want to copy a file with space in it, we can use another form of `COPY`
   ```dockerfile
   COPY ["hello space.txt", "."]
   ```
1. That's talk about `ADD`

   We can add a file from a URL

   ```dockerfile
   ADD http://some/url/file.json .
   ```

   If we pass a compressed file, `ADD` will automatically uncompressed this file into a directory

   ```dockerfile
   ADD file.zip .
   ```

## Excluding Files and Directories

Build `.dockerignore` file, and listing the ignore files and directories in `.dockerignore`

```.dockerignore
node_modules
```

Then We can build the image again

```bash
$ docker build -t react-app .
```

The image size will be smaller

---

Instructions

1. The noed_modules directory will be really large if we build a complex applicatioin, it will be nice if we don't transfer node_modules directory to Docker engine while we build the image
1. How can we exlude the node_modules directory? Like Git's `.gitignore` file, it's the same concept in Docker
   ```bash
   $ touch .dockerignore && echo node_modules/ >> .dockerignore
   ```

## Running Commands

For now, the `Dockerfile` should look like

```dockerfile
FROM node:18.9.0-alpine3.15
WORKDIR /app
COPY . .
RUN npm install
```

And the `.dockerignore`

```.dockerignore
node_modules/
```

Rebuild the image and run it

```bash
# Rebuild the image
$ docker build -t react-app .
# Run a new container
$ docker run -it react-app sh
```

---

Instructions

1. Without the node_modules in the image, we have to install the project dependencies using npm
1. So use the `RUN` command, which we can execute any commands that we execute in a terminal session

   ```dockerfile
   RUN npm install
   ```

## Setting Environment Variables

Conclution, the `Dockerfile` should look like

```dockerfile
FROM node:18.9.0-alpine3.15
WORKDIR /app
COPY . .
RUN npm install
ENV API_URL=http://hostname/api
```

And the `.dockerignore`

```.dockerignore
node_modules/
```

Rebuild the image and run it

```bash
# Rebuild the image
$ docker build -t react-app .
# Run a new container
$ docker run -it react-app sh
```

---

Instructions

1. Sometimes we need environment variable, for example, we have a front-end project need to talk to a backend, and we setup the API URL as environment variable
1. In this situation, we use `ENV`
   ```docekrfile
   ENV API_URL=http://hostname/api
   ```
   or with the older syntax, without the equal sign
   ```dockerfile
   ENV API_URL http://hostname/api
   ```
1. Make sure to rebuild the image
   ```bash
   $ docker build -t react-app .
   ```
1. We can check the environment variable by running the container
   ```bash
   docker run -it react-app sh
   ```
1. And we can use `printenv` command, or echo the `$API_URL`
   ```bash
   $ printenv
   ```

## Exposing Ports

Conclution, the `Dockerfile` should look like

```dockerfile
FROM node:18.9.0-alpine3.15
WORKDIR /app
COPY . .
RUN npm install
ENV API_URL=http://hostname/api
EXPOSE 3000
```

And the `.dockerignore`

```.dockerignore
node_modules/
```

Rebuild the image and run it

```bash
# Rebuild the image
$ docker build -t react-app .
# Run a new container
$ docker run -it react-app sh
```

---

Instructions

1. Inside the container, from the project directory, we run
   ```bash
   $ npm start
   ```
1. Now this start at development server on port 3000, e.g. `http://localhost:3000`
1. In the future, when we run this application inside a Docker container, this port, 3000, will be `open on the container` not on the host
1. On the same machine, we can have multiple containers running the same image, all these containers will be listening to port 3000
1. But the port 3000 on the host is not going to be automatically mapped to these containers. At [Setting the User](), We'll figure it out how to map a port on the host to a port on this container
1. We'll use `EXPOSE` to tell what port this container will be listening on
   ```dockerfile
   EXPOSE 3000
   ```
1. NOTE: the `EXPOSE` command doesn't automatically publish the port on the host. It just a form of documentation to tell us this container will eventually listen on port 3000

## Setting the User

After this section, the `Dockerfile` should look like

```dockerfile
FROM node:18.9.0-alpine3.15
# From here, as `root` user
WORKDIR /app
COPY . .
RUN npm install
ENV API_URL=http://hostname/api
EXPOSE 3000
RUN addgroup app && adduser -S -G app app
# To here, as `root` user
USER app
# From here and below, as `app` user
```

And the `.dockerignore`

```.dockerignore
node_modules/
```

Rebuild the image and run it

```bash
# Rebuild the image
$ docker build -t react-app .
# Run a new container
$ docker run -it react-app sh
```

check the user in the container

```bash
$ whoami
```

---

Instructions

1. By default, Docker run the application with the root user
1. That can open up a security hole in our system
1. So we should create a regular app user with limited privilege
1. We use `RUN` command to create the app user
   ```dockerfile
   # addgroup: create a group
   # adduser -G: set up the primary group of the user
   # adduser -S: create a system user
   # we use the same name for group and user in the application. e.g. `app`
   # finally, we have a user named `app`, which has a primary group `app`
   RUN addgroup app && adduser -S -G app app
   ```
1. After we add the user in the container, we can use `USER` command to set the user
   ```dockerfile
   USER app
   ```
1. After we set the user, all the following command will be executed using the user, we can check it out in the container

   ```bash
   # Rebuild the image
   $ docker build -t react-app .
   # Run a new container
   $ docker run -it react-app sh
   ```

   Inside the container

   ```bash
   $ whoami
   # app
   ```

1. The `app` user should not be able to write any of the files in the container

## Defining Entrypoints

After this section, we should have `Dockerfile` like

```dockerfile
FROM node:18.9.0-alpine3.15
RUN addgroup app && adduser -S -G app app
USER app
WORKDIR /app
COPY --chown=app:node . .
RUN ["npm", "install"]
ENTRYPOINT ["npm", "start"]
```

And we can build the image and run the container by

```bash
# Build image
$ docker build -t react-app .
# Run container
$ docker run react-app
```

The apllication should run the react app

1. Our Dockerfile is almost ready, and we can start the application by running
   ```bash
   $ npm start
   ```
1. We can start the container
   ```bash
   # We don't want to interac with the container, so no `-i`, also no need the pseudo TTY `-t`
   # We just want to start the application
   $ docker run react-app
   ```
1. Of course, our container just started and stop, because we didn't specify a command or a program to execute. We can start the application after the container is running

   ```bash
   $ docker run react-app npm start
   ```

   And we'll get permision denied: `[eslint] EACCES: permission denied,`
   With this Dockerfile

   ```dockerfile
   FROM node:18.9.0-alpine3.15
   WORKDIR /app
   COPY . .
   RUN npm install
   ENV API_URL=http://hostname/api
   EXPOSE 3000
   RUN addgroup app && adduser -S -G app app
   # We set the user at the end, and we execute the previous instructions as the root user. app user only has limited privileges
   USER app
   ```

1. We Update the Dockerfile, move `USER` to the top
   ```dockerfile
   FROM node:18.9.0-alpine3.15
   RUN addgroup app && adduser -S -G app app
   USER app
   WORKDIR /app
   COPY . .
   RUN npm install
   ENV API_URL=http://hostname/api
   EXPOSE 3000
   ```
1. Rebuild the docker image, and run it in the container

   ```bash
   # Build image
   $ docker build -t react-app .
   # Run container
   $ docker run react-app npm start
   ```

   BUT we still got ERROR! `Error: EACCES: permission denied, open '/app/package-lock.json'`

   Based on the [discussion](https://forum.codewithmosh.com/t/docker-npm-permission-denied/4766)

   Some alpine version have default use NODE which is overriding our app users, so we have to add some lines in Dockerfile, then we can use both NODE user and our app users

   ```dockerfile
   # Using custom user i.e. app
   FROM node:18.9.0-alpine3.15
   RUN addgroup app && adduser -S -G app app
   USER app
   WORKDIR /app
   COPY . .
   COPY --chown=app:node . .
   RUN npm install
   EXPOSE 3000
   ```

1. We can use `CMD` instruction to supply a default command to be executed

   ```dockerfile
   # Shell form
   CMD npm start
   # Execute form
   CMD ["npm", "start"]
   ```

   NOTE: if we have multiple `CMD`, only the last one will take effect. And We can always override the default `CMD` while we starting a container

   ```bash
   $ docker run react-app echo hello
   # the CMD in Dockerfile will be override
   ```

   | CMD Shell form                                                                                                              | CMD execute form                                                                                           |
   | --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
   | Docker will execute this command inside a seperate shell. In Linux, that shell is `/bin/sh`. In Windows, the shell is `cmd` | It's a better practice. Docker will execute it directly, there is no need to spin up another shell process |

1. Difference between `RUN` and `CMD`

   | RUN                                                                                                                                                                                          | CMD                                                          |
   | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
   | Build time instruction, it is executed at the time of building the image. Like `RUN npm install`, it means we're installing NPM dependencies, and these dependencies are stored in the image | Runtime instruction, it's executed when starting a container |

1. Define the `ENTRYPOINT`, also has `shell form` and `execute form`
   We can't easily override `ENTRYPOINT` when running a container, if we want to do that, we have to use `--entrypoint` option

   So we often use `ENTRYPOINT` when we know for sure that this is the command that should be executed whenerver we start a container, there is no exception.

   ```dockerfile
   ENTRYPOINT ["npm", "start"]
   ```

1. `CMD` or `ENTRYPOINT`, which instruction to use is a matter of personal preference

## Speeding Up Builds

After this section, we should have `Dockerfile` like

```dockerfile
FROM node:18.9.0-alpine3.15
RUN addgroup app && adduser -S -G app app
USER app
WORKDIR /app
COPY --chown=app:node package*.json .
RUN ["npm", "install"]
COPY --chown=app:node . .
ENTRYPOINT ["npm", "start"]
```

And we can build the image and run the container by

```bash
# Build image
$ docker build -t react-app .
# Run container
$ docker run react-app
```

The apllication should run the react app

1. Concept of layers in Docker
   An image is essentially a collection of layers

   When Docker trying to build an image for us, it executes each of the instruction in the Dockerfile, and create a new layer, and the layer only includes the files that were modified as a result of that instruction

1. We can see the image's layer by `history` command

   ```bash
   $ docker history react-app
   ```

   And the result should look like

   | IMAGE        | CREATED           | CREATED BY                                    | SIZE   | COMMENT                |
   | ------------ | ----------------- | --------------------------------------------- | ------ | ---------------------- |
   | 8b78c74fbdab | 20 minutes ago    | ENTRYPOINT ["npm" "start"]                    | 0B     | buildkit.dockerfile.v0 |
   | <missing>    | 20 minutes ago    | COPY . . # buildkit                           | 1.6MB  | buildkit.dockerfile.v0 |
   | <missing>    | 20 minutes ago    | RUN npm install # buildkit                    | 272MB  | buildkit.dockerfile.v0 |
   | <missing>    | About an hour ago | COPY package\*.json . # buildkit              | 1.19MB | buildkit.dockerfile.v0 |
   | <missing>    | 2 hours ago       | WORKDIR /app                                  | 0B     | buildkit.dockerfile.v0 |
   | <missing>    | 2 hours ago       | USER app                                      | 0B     | buildkit.dockerfile.v0 |
   | <missing>    | 2 hours ago       | RUN /bin/sh -c addgroup app && adduser -S -G… | 4.87kB | buildkit.dockerfile.v0 |
   | <missing>    | 9 days ago        | /bin/sh -c #(nop) CMD ["node"]                | 0B     |                        |
   | <missing>    | 9 days ago        | /bin/sh -c #(nop) ENTRYPOINT ["docker-entry…  | 0B     |                        |
   | <missing>    | 9 days ago        | /bin/sh -c #(nop) COPY file:4d192565a7220e13… | 388B   |                        |
   | <missing>    | 9 days ago        | /bin/sh -c apk add --no-cache --virtual .bui… | 7.88MB |                        |
   | <missing>    | 9 days ago        | /bin/sh -c #(nop) ENV YARN_VERSION=1.22.19    | 0B     |                        |
   | <missing>    | 9 days ago        | /bin/sh -c addgroup -g 1000 node && addu…     | 151MB  |                        |
   | <missing>    | 9 days ago        | /bin/sh -c #(nop) ENV NODE_VERSION=18.9.0     | 0B     |                        |
   | <missing>    | 5 weeks ago       | /bin/sh -c #(nop) CMD ["/bin/sh"]             | 0B     |                        |
   | <missing>    | 5 weeks ago       | /bin/sh -c #(nop) ADD file:4b51a9d40f20d2beb… | 5.33MB |                        |

   We can see the `RUN npm install` layer got the size of `272MB`

1. Docker has an optimization mechanism built into it.

   Every time we ask Docker to build this image, it's going to look at each the instruction and see if the instruction is changed or not

   If it's not changed, Docker is not going to rebuild the layer, it's going to reuse it from it's cache

1. Based on the mechanism, we can seperate the installation of the third party dependencies from copying our application files

   ```dockerfile
   COPY package*.json .
   RUN ["npm", "install"]
   COPY . .
   ```

   We can see Docker reuses the layer cache for the third party package install instruction `CACHED [5/6] RUN ["npm", "install"]`

1. To optimize the build, we should organize the `Dockerfile` from `Stable instructions` to `Changing instructions`

## Removing Images

In this section, we learn `prune` command and `rm` command

```bash
# remove dangling containers & images
$ docker container prune
$ docker image prune
```

```bash
$ docker image rm [IMAGE ID/REPOSITORY]
```

1. We might have many images have no name and no tags. These are `dangling images`, meaning loose images. These are essentially layers that have no relationship with a tagged image
1. As we changing our Dockerfile and rebuilding our image, Docker was creating these layers, and at some point, these layers lost their relationship with our react-app image
1. We can use `prune` command to delete these images
   ```bash
   $ docker image prune
   ```
1. But we still have containers running on an older react-app image, also we can use `prune` command to delete that

   ```bash
   $ docker container prune
   ```

   Then we can delete the dangling images

   ```bash
   $ docker image prune
   ```

1. And if we want to remove the image which is not dangling image, we can use `rm` command
   ```bash
   # $ docker image rm [IMAGE ID/REPOSITORY]
   $ docker image rm react-app
   ```

## Tagging Images

1. Whenever we build an image or pull it from Docker Hbu, by default, Docker uses the latest tag, that latest tag is just a label. It doesn't necessarily mean this is the latest version of your image, if you don't tag your image's properly, latest can point to an older image
1. We shouldn't use latest tag for production. Because if something goes wrong, we can't easily troubleshoot issues. Always use explicit tag to identify what version we're running in each environment
1. Now we hav images like

   | REPOSITORY | TAG    | IMAGE ID     | CREATED     | SIZE  |
   | ---------- | ------ | ------------ | ----------- | ----- |
   | react-app  | latest | 8b78c74fbdab | 3 hours ago | 440MB |

1. Tag image while we building it

   ```bash
   $ docker build -t react-app:1 .
   ```

   Now we have images like

   | REPOSITORY | TAG    | IMAGE ID     | CREATED     | SIZE  |
   | ---------- | ------ | ------------ | ----------- | ----- |
   | react-app  | 1      | 8b78c74fbdab | 3 hours ago | 440MB |
   | react-app  | latest | 8b78c74fbdab | 3 hours ago | 440MB |

   Different preference for tag

   - Code name, e.g. Maverick
     Everyone in the team know what the code name mean, and we use it as a tag
   - Sementic versioning, e.g. 3.1.4
     This approach is common amongst teams that don't release that often
   - Build number, e.g. 76 or 77
     This approach is for teams release frequently

1. We can remove a tag by
   ```bash
   $ docker image remove react-app:1
   ```
1. Also, we can tag image after we build it
   ```bash
   $ docker image tag react-app:latest react-app:1
   ```
1. After we made some change in the project, and create a new image

   ```bash
   $ echo edit >> README.md && docker build -t react-app:2 .
   # show the images
   $ docker images
   ```

   We got

   | REPOSITORY | TAG    | IMAGE ID     | CREATED       | SIZE  |
   | ---------- | ------ | ------------ | ------------- | ----- |
   | react-app  | 2      | 894a624c73a7 | 7 seconds ago | 440MB |
   | react-app  | 1      | 8b78c74fbdab | 3 hours ago   | 440MB |
   | react-app  | latest | 8b78c74fbdab | 3 hours ago   | 440MB |

   We can notice that the latest tag image is pointing the 1 version, which is older

   We can manually update the latest to the newr one by

   ```bash
   # $ docker image tag [IMAGE ID] react-app:latest
   $ docker image tag 894 react-app:latest
   ```

## Sharing Images

1. Go to Docker Hub and create an account
1. After signin, in [Repositories](https://hub.docker.com/repositories), click `Create repository`
1. Back to the terminal, publish the latest version of react-app image.
   To push our image to the repository, we have to give our image the same name in the Docker Hub we just created, e.g. `[user_name]/[repository_name]`

   ```bash
   # Build an image for publish to Docker Hub
   $ docker image tag 894 enhsu/react-app:2
   # Login Docker
   $ docker login
   # Push the image
   $ docker push enhsu/react-app:2
   ```

1. Docker will push each layers in the image. One we push it for the first time, our future pushes will be faster
1. After push the image, we can pull it from Docker Hub by `pull` command
   ```bash
   $ docker pull enhsu/react-app:2
   ```

## Saving adn Loading Images

1. If we want to put our image to another machine without going through the Docker Hub, we can use `save` command
   ```bash
   docker image save -o react-app.tar react-app:2
   ```
1. Then we got `react-app.tar` file, and we can use `load` command to load the image
   ```bash
   $ docker image load -i react-app.tar
   ```
