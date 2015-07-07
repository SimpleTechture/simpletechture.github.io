---
layout: post
title: Docker Machine
tags:
- Docker
- Docker Machines
- Containers
---

Before I will start talking about scaling this Blog, I first need to talk about docker-machine. Docker-Machine is a tool for creating and managing hosts that will run Docker containers. If you use docker-machine to create your host, you can then perform the following actions remotely using docker-machine  

* Starting, stopping, restarting
* Upgrading Docker
* Configuring the Docker client to talk to your host

The really great thing is that you can use it to create Docker hosts on several Cloud suppliers. Currently the following drivers as Docker machines calls them are supported:

* Amazon Web Services
* Digital Ocean
* Google Compute Engine
* IBM Softlayer
* Microsoft Azure
* Microsoft Hyper-V
* Openstack
* Rackspace
* Oracle VirtualBox
* VMware Fusion
* VMware vCloud Air
* VMware vSphere

This also includes Virtualization using Microsoft Hyper-V or VMwarevSphere. I will be using the cloud provider on which this Blog is running, Microsoft Azure.

I will be running docker-machine on a linux host. It is however also possible to run the tool on the Windows platform. The easiest way to get docker-machine is to download it from the [releases](https://github.com/docker/machine/releases) page of docker machine on GitHub. The tool is still in beta / pre release, so expect changes in the future.

The first thing you need to do before running docker machine to create hosts on Microsoft Azure is to generate a certificate that you will install in Microsoft Azure to allow docker machine to access Microsoft Azure.

Generating a certificate valid for Azure can be done using the following commands on Ubuntu.
```
$ openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout azurecert.pem -out azurecert.pem
$ openssl pkcs12 -export -out azurecert.pfx -in azurecert.pem -name "My Azure Certificate"
$ openssl x509 -inform pem -in azurecert.pem -outform der -out azurecert.cer
```

The created ```azurecert.cer``` file must be uploaded to the **Management Certificates** under Settings in the Azure management portal. 

![Azure Certificates](../../../img/AzureCertificates.JPG)

After the certificate is uploaded, the docker host, an Ubuntu machine can be created using the following command.
```
$ docker-machine create -d azure --azure-subscription-id="89180bf3-e22d-55b0-b874-c3618233d5d0" --azure-subscription-cert="certificates/azurecert.pem" hostcreatedviadockermachine
```

Docker-machine will connect to Microsoft Azure to create the actual docker host. This will take some time.

```
$ docker-machine create -d azure --azure-subscription-id="89180bf3-e22d-55b0-b874-c3618233d5d0" --azure-subscription-cert="certificates/azurecert.pem" hostcreatedviadockermachine
INFO[0002] Creating Azure machine...
INFO[0094] Waiting for SSH...INFO[0433] "ubuntuviadockermachine" has been created and is now the active machine.
INFO[0433] To point your Docker client at it, run this in your shell: $(docker-machine env ubuntuviadockermachine)
```
When the command finished, the Microsoft Azure portal shows the new host up and running.

![New virtual machine](../../../img/NewlyVirtualMachineViaDockerMacine.JPG)

After performing the command logged at the end of the output of the docker-machine command ```$(docker-machine env ubuntuviadockermachine) ``` your local docker client will point your to the newly created docker host. This means that when you will then issue a docker command, it will actually be executed on the remote host, pretty cool!

If you type the following command docker machine will show you the environment variables and it will show you that it is actually connected to the remote Docker host.

```
$ docker-machine env
export DOCKER_TLS_VERIFY=yes
export DOCKER_CERT_PATH=/root/.docker/machine/machines/ubuntuviadockermachine
export DOCKER_HOST=tcp://ubuntuviadockermachine.cloudapp.net:2376
```

Docker-machine contains several commands to manage Docker hosts. For example, the command ```ls``` to list the available hosts.

```
$ docker-machine ls
NAME                   ACTIVE   DRIVER   STATE
ubuntuviadockermachine *        azure    Running   
```

Now you can simply issue docker command and they will be executed on the remote host. For example, doing a hello world via de busybox docker image.

```
$ docker run busybox echo hello world
Unable to find image 'busybox:latest' locally
511136ea3c5a: Pull complete
df7546f9f060: Pull complete
ea13149945cb: Pull complete
4986bf8c1536: Pull complete
busybox:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Status: Downloaded newer image for busybox:latest
hello world
```

With Docker-machine in place we are now ready to scale this blog using docker swarm. Which will be the topic of the next post.