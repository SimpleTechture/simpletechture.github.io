---
layout: post
title: Linking Docker containers
tags:
- Docker
- Linking
- Containers
---

If you remember from my previous post, this is my current setup.

{% highlight ruby %}
def foo
  puts 'foo'
end
{% endhighlight %}


![Current setup](../../../img/TwoDockerContainer-3.jpg)

Ghost references the PostgreSQL server using the ip address of the Docker container running PostgreSQL. The problem is that each time I restart the container it gets a new ipaddress and I have to change the ipaddress in the config.js of Ghost. Luckily there is a better way to do this by linking containers.

When starting a container it is possible to give Docker instructions to links this container with another container using the **--link** command. The arguments to the link parameter are the name or id of the container you want to link to and an alias. 

```
--link <name or id>:alias
```

On the basis of this argument Docker moves information of the container you link into the container that you start. Docker creates environment variables and also changes the hosts file in which it includes the name of the linked container and the ipaddress of that container. Lucky for me, this is exactly what I needed to being able to remove the hard coded ipaddress in the config.js file.

The Ghost container can be started with the following command including the linking

```
sudo docker run -p 80:2368 -v /home/kalkie/ghostdata:/ghost-override --link postgres:postgres dockerfile/ghost
```

If we then look into the hosts file it actually has a new entry **postgres** that points to the ipaddress of the postgres container.

```
172.17.0.40     d9055e5fe3da
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.39     postgres
```
 
The config.js file of Ghost can now point to the hostname of the postgres container instead of the ipaddress.

```
config = {
   production:
   {
       url: 'http://www.simpletechture.com',
       mail: {},
       database: {
            client: 'pg',
            connection: {
               host     : 'postgres',
                user     : 'ghostdb',
                password : 'secretpwd',
                database : 'ghostdb',
                charset  : 'utf8'
            }
        },
        server: {
            host: '0.0.0.0',
            port: '2368'
        },
        logging: false
    },
```

One gotcha I found out that linking of the containers is static. If you restart the Postgres container and the ip address changes, the ip address in the hosts file of the container that links to the PostgreSQL container does not automatically gets updated. You have to also restart and relink the Ghost container. 