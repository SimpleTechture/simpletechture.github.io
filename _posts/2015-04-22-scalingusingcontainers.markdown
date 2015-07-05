---
layout: post
title: Scaling using docker
tags:
- Docker
- Scaling
- Nginx
- Ghost
---

Finally, we have all the parts in place to start talking about scaling this Blog using containers. The configuration we are going to create looks like this.

![Nginx](../../../img/DockerScalingWithNGinx.png)

As you can see, we are going to use a load balancer to be able to redirect the web requests to a possible pool of servers/containers. In this case I use Nginx because it offers great flexibility and is widely known.

I will not create my own docker container with nginx but will reuse the container which [https://github.com/jwilder/nginx-proxy](Jason Wilder) created. The container he created has some interesting functionality in relation with docker, but later more about that.

First, I will set up the configuration using the official nginx docker image ```dockerfile/nginx``` but before I start that I need to change my [existing fig configuration](http://www.simpletechture.com/fiddling-with-fig/) to start another Ghost container. 


```
ghost1:
  image: dockerfile/ghost
  links:
   - postgres
  ports:
   - 2368:2368
  volumes:
   - /home/kalkie/ghostdata:/ghost-override

ghost2:
  image: dockerfile/ghost
  links:
   - postgres
  ports:
   - 2369:2368
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

As you can see, I have just added another ghost entry using almost the same parameters. The only difference is in the port mapping, we need to use different ports on the host side.

```
$ sudo fig up -d
```

Now we are going to add the new nginx container which is going to be load balancer.  

```
$ sudo docker run -d -p 80:80 -v /home/kalkie/logs:/var/log/nginx -v /home/kalkie/conf/:/etc/nginx/conf.d dockerfile/nginx
```

I am mapping the exposed volume /var/log/nginx to be able to access the nginx logs and the volume /etc/nginx/conf.d to be able to write the nginx load balancing configuration.

In my local conf folder I create a ghost.conf file with the following contents:

```
    upstream ghost {
        server [server1address];
        server [server2address];
    }

    server {
        location / {
            proxy_pass http://ghost;
        }
    }
```
This is nginx specific configuration which defines the two Ghost web servers in the upstream section and the server section links the base of the request ```/``` to the upstream configuration.

[server1address] and the [server2address] need to be replaced with the actual ip address and port of both containers. We retrieve them using:

```
$ sudo docker inspect f8045f6d2384 | grep IPAddress
        "IPAddress": "172.17.0.117",
```

If we do the same for the other container we are able to enter both ip addresses in the ghost.conf, which now looks like this.

```
    upstream ghost {
        server 172.17.0.117:2368;
        server 172.17.0.118:2368;
    }
    server {
        location / {
            proxy_pass http://ghost;
        }
    }
```

If we restart nginx inside the nginx container the system is up and running.

But as we already saw in [fiddling with fig](http://www.simpletechture.com/fiddling-with-fig/), restarting containers also assigns new ip addresses to the container. Therefore the current solution and configuration work but is not really maintenance friendly.

Lucky for me, [Jason Wilder](https://github.com/jwilder/nginx-proxy) already created a special nginx-proxy container that is very flexible. 

The nginx-proxy container runs nginx and docker-gen. Docker-gen automatically generates reverse proxy configurations for nginx. But also automatically reloads nginx when containers are started and stopped. Really cool!

Docker-gen uses the Docker remote API to inspect the containers IP, Port and other configuration data. The Docker remote API also provides events can can be captured when containers are started or stopped. If you are interested in the source, [docker-gen](https://github.com/jwilder/docker-gen) is written in Go and is open-source.

The nginx-proxy automatically enumerates all the containers that have an environment variable set called ```VIRTUAL_HOST```. There we have to change our fig configuration to include the ```VIRTUAL_HOST``` to the environment variables.

I tried to use fig together with nginx-proxy but when started using fig it seem to not find the docker container events. Therefore, I switched back to my existing bash files to start the containers. 

I have 3 bash script files

For starting PostGreSQL
```
#!/bin/bash
sudo docker run --name postgres -e POSTGRES_PASSWORD=MySecretPwd -e POSTGRES_USER=ghostdb -v /home/kalkie/postgres:/var/lib/postgresql/data -d postgres
```

Another one for starting Ghost with the setting of the environment variable that nginx-proxy uses.

```
#!/bin/bash
sudo docker run -e "VIRTUAL_HOST=www.simpletechture.com" -d -v /home/kalkie/ghostdata:/ghost-override --link postgres:postgres dockerfile/ghost
```

And finally one for nginx-proxy
```
!/bin/bash
docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock jwilder/nginx-proxy
```

The really nice thing with this setup is that I can start or stop Ghost containers with the Ghost script and they are automatically added to the pool of webservers in the Nginx configuration.

I still don't know why starting all containers with fig is not working. But I am sure that I will figure it out later. 