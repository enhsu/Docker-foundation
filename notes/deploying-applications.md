# Deploying Applications

## Purpose

- Deployment options
- Getting a Virtual Private Server(VPS)
- Using Docker machine to provision hosts and connect with them
- Creating optimized production images
- Deploying the application

## NOTE

I found out the courese of Mosh Hamedani is deploy it to Digital Ocean. And there are so many platform we can use, such as GCP and AWS.
I decided to skip this section, and start again until I have to deploy the project, and follow the [document](https://docker-docs.netlify.app/machine/overview/) base on the platform I want to deploy.
So temporaryily this course is end for me

## Navigation

1. [Deployment Options](#deployment-options)
1. [Getting a Virtual Private Server](#getting-a-virtual-private-server)
1. [Installing Docker Machine](#installing-docker-machine)

## Deployment Options

1. To deploy our Docker as applications, we have 2 options
   1. Single-host deployment
      - Cons
        - If our server goes offline, our application will not be accessible
        - If our application grow rapidly, and we get hundreds or thousadns of users, a single server is not able to handle that load
        - So we better use clusters
   1. Cluster deployment(a group of servers)
      - Pros
        - With high availability and scalability
1. We need special tools called `orchestration tools` for cluster deployment
   - Docker has it's own orchestration tool built into it called `Docker swarm`
   - Or most people these days use another tool called `Kubernetes`, which is a Google product
1. Kubernetes is fairly complex, so in this section our focus will be on a single host deployment

## Getting a Virtual Private Server

1. To deploy our application, we need a private server(VPS), there're ton of options
   - Digital Ocean
   - Google Cloud Platform(GCP)
   - Microsoft Azure
   - Amazon Web Services(AWS)
1. Digital Ocean is the simplest and most beginner friendly one

## Installing Docker Machine

1. Once we have a server, we need to use to tool called `Docker machine` to talk to the Docker engine on that server. So this way we can execute Docker commands in our terminal and the commands will be sent to the Docker engine on our server
1. Install the `Docker machine`
   - [Reference: Docker machine github](https://github.com/docker/machine)
   - [Reference: Install Docker machine](https://docker-docs.netlify.app/machine/install-machine/)
