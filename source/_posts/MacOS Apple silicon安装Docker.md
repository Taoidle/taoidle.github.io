---
layout:MacOS Apple silicon Install Docker
title: MacOS Apple silicon Install Docker
date: 2022-08-07 09:20:53
categories:
  - - 折腾记录
tags: 
  - Arm
  - Apple
  - 苹果
  - Docker
  - MacOS
---

### Install

```Shell
brew install --cask docker
brew install docker docker-compose
```

### Test

Input:

```Shell
docker pull hello-world
```

Output:

```Shell
latest: Pulling from library/hello-world
70f5ac315c5a: Pull complete 
Digest: sha256:dcba6daec718f547568c562956fa47e1b03673dd010fe6ee58ca806767031d1c
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
```

Input:

```Shell
docker run hello-world
```

Output:

```Shell
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
