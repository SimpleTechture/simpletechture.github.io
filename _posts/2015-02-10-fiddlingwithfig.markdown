---
layout: post
title: Fiddling with Fig
tags:
- Docker
- Fig
- Containers
---

Although we are currently only using two containers, the command to boot both containers is quite long. Before introducing the third data only container, I want to simplify starting and stopping our containers.
[Fig](http://www.fig.sh/) is a tool to simplify this, with Fig you can describe your containers boot parameters and depedencies inside a configuration file using yaml syntax. 

Fig can be installed using the following command

```
sudo pip install fig
```

or download it from [GitHub](https://github.com/docker/fig/releases/). While writing this post Fig is being renamed by the docker team to Docker compose or Compose for short.

Looking at our current containers, the first container that contains PostgreSQL is started using the following command line

```
$ sudo docker run --name postgres -e POSTGRES_PASSWORD=mysecretpwd -e POSTGRES_USER=ghostdb -v /home/kalkie/postgres:/var/lib/postgresql/data -d postgres
```

while the second container that contains Ghost is started using the following command line

```
$ sudo docker run -d -p 80:2368 -v /home/kalkie/ghostdata:/ghost-override --link postgres:postgres dockerfile/ghost
```

This is how the fig.yml configuration file looks like to boot both container

```
ghost:
  image: dockerfile/ghost
  links:
   - postgres
  ports:
   - "80:2368"
  volumes:
   - /home/kalkie/ghostdata:/ghost-override

postgres:
  image: postgres
  environment:
    POSTGRES_PASSWORD: mysecretpwd
    POSTGRES_USER: ghostdb
  volumes:
   - /home/kalkie/postgres:/var/lib/postgresql/data

```

The file is divided into two sections, ghost and postgres representing both containers. 

* ghost: The name of the instance of the container
* image: The name of the docker image
* links: To which container should this container link
* ports: Defines the port mapping
* volumes: Defines the volume mapping

* postgres: The name of the other container
* image: The name of the docker image
* environment: Defines the environment variables to set 
* volumes: Defines the volume mapping

Basically the fig.yml contains all the parameters of the command line but structured differently.

Both containers can now be started using the command

```
$ sudo fig up
```

or in deamon mode

```
$ sudo fig up -d
```
which is a lot simpler then using the docker command line.

Next, we will be looking at centralizing the data of the Ghost and the PostgreSQL container using a data only container so that creating backups becomes easier. 