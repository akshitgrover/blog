---
layout: post
title: "Docker tips & tricks"
categories: [docker]
share: true
---

Docker has revolutionized the cloud environment. Any company which operates any kind of cloud environment uses docker either to test or to deploy production workloads.
Knowing the importance of docker, it is still tedious sometimes to work with the CLI.

There are many commands and options which are not much explored but turn out to be in handy while working with docker. We are going to discuss a few of those commands in this blog post.
<!--more-->

### Copying a file inside a running container

While developing an image or testing an application one can come across the need to copy some files into or from the container to host, `docker cp` is used just to do the same.

Consider you have a container named **post** and a file in the host /home/akshitgrover/hello.txt, which you want to copy into the container at the path /h.

**Usage:**<br>
~~~shell
docker container cp <source> <destination>
~~~ 

**Copy file into container:**<br>
~~~shell
docker container cp /home/akshitgrover/hello.txt post:/h
~~~

**Copy file from container:**<br>
~~~shell
docker container cp post:/h /home/akshitgrover/h.txt
~~~

**Note:** Both the source and destination paths should be absolute

### Executing a command inside a running container

At times one wants to execute a command in the running container or wants a shell to interact with the container. `docker exec` is just the command for the purpose.

**Usage:**<br>
~~~shell
docker container exec <containerName> <command>
~~~

**Example:**<br>
~~~shell
docker container exec <containerName> ls /etc
~~~

**Open an interactive shell:**<br>
~~~shell
docker container exec -it <containerName> sh
~~~
*Flags -*<br>
*i* : Opens an input stream to the process of command specified<br>
*t* : Used to create a pseudo tty for the container

### Creating a backup of a volume

Often we have to backup data of production workloads. Containers use volumes to persist data as container are considered to be ephemeral when it comes to legacy production workloads. Container based application are designed to handle errors in container execution and avoid service downtime.

Since volumes are used to persist data between the life span of containers, Taking a backup of volumes is the key to backup production data.

**Sample backup.sh**<br>

~~~shell
#!/bin/bash

# Run a sample container

docker container run -idv data:/data --name data_container busybox;

mkdir $HOME/backup;

# Creates a temporary container which has volumes from data container and creates 
# backup of data directory without disrupting service of database container.
#
# Temporary container creates volume mount at backup directory present in the home 
# directory. The backup directory will contain a backup.tar file
 
docker container run -v $HOME/backup:/backup --volumes-from data_container \
--name backup busybox tar -cvf /backup/backup.tar /data;

docker container rm backup --force;
~~~

### Importing data from a volume backup

We discussed backing up a volume in the previous section. Now, Let's see how to use that backup in another container.

**Sample import_backup.sh**<br>

~~~shell
#!/bin/bash

cd $HOME/backup;

# Create the container which will use the backup

docker container run -idv new_data:/data --name new_data_container busybox;

# Copying backup.tar in the container

docker container cp $(pwd)/backup.tar new_data_container:/backup/backup.tar

# Extracting backup.tar file

docker container exec new_data_container sh -c "tar -C / -xvf backup.tar";
~~~

### Deleting all containers

While working with docker on a dev machine we tend to create a lot of container during trial and error. Deleting all these containers at once is a blessing.

~~~shell
docker container rm $(docker container ls -aq) --force
~~~
*Flags -*<br>
*a* : List all container running/stopped
*q* : List only container IDs
*force*: Delete running containers as well

### Delete only stopped containers

~~~shell
docker container prune --force
~~~

### Delete unused volumes

Volumes take up a lot of space in the host system.
Deleting all those volumes which are not being used by any container running/stopped is a must.

~~~shell
docker volume prune --force
~~~

### Delete unused images

Docker images are heavier than an iceberg. Deleting all those volumes at once which are not being used is achievable.

~~~shell
# This command will delete every dangling image
docker image prune --force

# This command will delete all unused images
docker image prune --all --force
~~~

*Dangling image*<br>
An image which has been overwritten because of re-build with the same tag.

**Note:**
In production it is advisable to use --all flag but not in dev machine.


### Limiting resources used by a container

It is necessary to limit the resources used by a container in a multi-container environment.

~~~shell
# This will allow the container to have access only to 1 cpu core
docker container run -it --cpus 1 busybox sh
~~~

### Reducing image build size

In production, it is important to keep image size low.

Using an `alpine image` as base of application reduces image at least by 50%.

~~~dockerfile
FROM alpine
...
...
...
...
CMD ["sh"]
~~~

## What's next

Refer these articles below to gather some more tips and tricks which makes docker even more happening to use.

* [Docker bash alises](https://hackernoon.com/handy-docker-aliases-4bd85089a3b8)
* [Few more commands](https://medium.com/@clasikas/docker-tips-tricks-or-just-useful-commands-6e1fd8220450)
