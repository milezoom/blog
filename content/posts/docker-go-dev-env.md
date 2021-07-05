---
title: "Docker Go Dev Env"
date: 2021-07-05T08:22:35Z
draft: false
---

Docker is very helpful when you need specific development environtment.
Also if you don't want your OS become "dirty" because too many package installed.
For my case, I use docker container to isolate environment to develop golang application.
Then I use vscode remote to do coding inside the container. Here how I setup my golang dev env.

## 1. Creating Dockerfile
To create container containing golang, I use base image minideb from bitnami, that based on debian.
```dockerfile {linenos=table}
FROM bitnami/minideb as builder

RUN apt-get -y update && \
    apt-get -y install apt-utils iproute2 curl git login unzip procps build-essential && \
    curl -L -o /tmp/gosetup.tar.gz https://golang.org/dl/go1.16.5.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf /tmp/gosetup.tar.gz && rm /tmp/gosetup.tar.gz && \
    export PATH=$PATH:/usr/local/go/bin:/root/.go/bin && export GOPATH=/root/.go && \
    mkdir -p /app

ENV PATH="${PATH}:/usr/local/go/bin:/root/.go/bin"
ENV GOPATH="/root/.go"

WORKDIR /app

CMD tail -f /dev/null
```
Using script above, also installed several useful tools like `curl`, `git`, `zip`, and `build-tools`.
Because I need version 1.16 and default version in os is 1.13, so I need to manually download from official golang site.
I set my `GOPATH` at `/root/.go` and my `WORKDIR` (place to store my code) at `/app`.
And finally I run `tail` at `/dev/null` to make container run forever.

## 2. Creating docker-compose file
Then after Dockerfile created, I create `docker-compose.yml` file.
```docker {linenos=table}
version: "3"
services:
    golang:
        build:
            context: .
            dockerfile: Dockerfile
        image: golang:1.0.0
        container_name: golang
        volumes:
            - golang-data:/app
volumes:
    golang-data:
```
For the container, I use docker volume that mounted to `/app` directory for data persistence.
Put both files in one directory, then run `docker-compose up -d` to build and start the container.

## 3. Connect VSCode to Container
To connect vscode to container, use these extension:
- Remote - Container ([link][ext remote-container]).
- Docker ([link][ext docker]) (sometimes proper setup needed for this ext).  

Then click remote connection button at bottom left, then select **Attach to Running Container...**  
A new window will open, and directory `/app` (according to specified `WORKDIR`) will be ready in file explorer.
And for the integrated terminal, also mapped directly to `/app` directory.
That's all, you can start code inside container using go SDK in the container from vscode. 

[ext remote-container]: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers
[ext docker]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker