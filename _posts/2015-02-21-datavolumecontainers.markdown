---
layout: post
title: Data Volume Containers
tags:
- Docker
- Data Volume Containers
- Containers
---

Currently this blog is running on two Docker containers. One running the  node.js Blog engine and another running the PostgreSQL database server. The data of both the Blog container and the database container is mapped onto the Host virtual machine.

One thing that I need to arrange is to create backup's of the data of both containers. I want to perform this using Data Volume containers. Data Volume containers are special containers that are only responsible for hosting the data volumes and nothing else. These Data Volume containers are then linked to the the containers running the application. 

Currently I have the following two containers running

{% highlight bash %}
$ sudo docker ps -a

CONTAINER ID   IMAGE                   COMMAND             CREATED             STATUS              PORTS                  NAMES
3636ca34d780   dockerfile/ghost:latest "bash /ghost-start" 19 minutes ago     Up 18 minutes       0.0.0.0:80->2368/tcp   kalkie_ghost_1
488a68e877a5        postgres:latest           "/docker-entrypoint.   19 minutes ago      Up 19 minutes       5432/tcp               kalkie_postgres_1
{% endhighlight %}

The following command creates a new data volume container and links it to the Ghost container.

{% highlight bash %}
$ sudo docker run --rm --volumes-from kalkie_ghost_1 -t -i busybox sh
{% endhighlight %}

The command creates a new container and maps the volumes ```--volumes-from``` from the ghost container. The container is started in the foreground and executes a shell ```sh```. It uses the busybox base container.

Inside the container we are able to list the volumes from the Ghost container, for example ```ghost-override```. 

{% highlight bash %}
$ sudo docker run --rm --volumes-from kalkie_ghost_1 -t -i busybox sh
/ # ls /ghost-override/
config.js  content
/ #
{% endhighlight %}

####Backup

Using the same approach we can directly backup the container using a tar file with the following command.

{% highlight bash %}
$ sudo docker run --rm --volumes-from kalkie_ghost_1 -v $(pwd):/backup_ghost busybox tar cvf /backup_ghost/backup_ghost.tar /ghost-override
{% endhighlight %}

And the same can be done for the PostgreSQL container using the following command.

{% highlight bash %}
$ sudo docker run --rm --volumes-from kalkie_postgres_1 -v $(pwd):/backup_postgress busybox tar cvf /backup_postgress/backup_postgress.tar /var/lib/postgresql/data
{% endhighlight %}

Both commands create a tar file inside your home directory that includes the data from both volumes of each container.

#### Restore

Restoring the data using a data only container can be done using the following command which creates a new container and extract the tar archive inside that container that mapped the volume from the Ghost container.

{% highlight bash %}
$ sudo docker run -rm --volumes-from kalkie_ghost_1 -v $(pwd):/backup_ghost busybox tar xvf /backup_ghost/backup_ghost.tar
{% endhighlight %}

And the same can be done for the PostGreSQL container

{% highlight bash %}
$ sudo docker run -rm --volumes-from kalkie_postgres_1 -v $(pwd):/backup_postgres busybox tar xvf /backup_postgres/backup_postgres.tar
{% endhighlight %}

This concludes the setup of this Blog using Docker. We are using two containers, one for running the Ghost blog engine and another for running the PostGreSQL engine. Both containers are [linked together](http://www.simpletechture.com/fiddling-with-fig/) so that the Ghost container knows the ip address of the PostGreSQL database. [Fig](http://www.simpletechture.com/fiddling-with-fig/) is used to make configuring and booting both containers more easy. 

Finally, this post explains how data only containers can be used to backup the data of both containers.

Next, we will be looking at scaling the Blog ;-)