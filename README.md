Docker Commands
===============

### Containers

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ shell
docker build -t friendlyname .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyname  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyname         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Services

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ shell
docker stack ls              # List all running applications on this Docker host
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file; depends on docker-compose.yml
docker stack services <appname>       # List the services associated with an app
docker stack ps <appname>   # List the running containers associated with an app
docker stack rm <appname>                             # Tear down an application
docker node ls      # one-node swarm is still up and running?
docker swarm leave --force      # take down swarm
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Swarm

docker swarm init Swarm initialized: current node (zze8q1hakyll5iigkwha7cg4o) is
now a manager.

To add a worker to this swarm, run the following command:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
docker swarm join --token SWMTKN-1-60dv8qodi1dct5tt0ih64uvdb4kuymisf0rs0io6iv3bzagg6e-8lqy9s6mic4hsuci1574diexl 10.2.164.77:2377
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To add a manager to this swarm, run 'docker swarm join-token manager' and follow
the instructions.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ shell
docker-machine create --driver virtualbox myvm1 # Create a VM (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # View basic information about your node
docker-machine ssh myvm1 "docker node ls"         # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"        # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # View join token
docker-machine ssh myvm1   # Open an SSH session with the VM; type "exit" to end
docker-machine ssh myvm2 "docker swarm leave"  # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f" # Make master leave, kill swarm
docker-machine start myvm1            # Start a VM that is currently not running
docker-machine stop $(docker-machine ls -q)               # Stop all running VMs
docker-machine rm $(docker-machine ls -q) # Delete all VMs and their disk images
docker-machine scp docker-compose.yml myvm1:~     # Copy file to node's home dir
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # Deploy an app
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 

 

Docker Tut
==========

https://get.docker.com

 

sudo usermod -aG docker adrian

docker version

 

docker requires root access.... for RHEL distros you eather be root or use
sudo... the add of the user will not work

install the docker-machine and docker-compose

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
curl -L https://github.com/docker/machine/releases/download/v0.14.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && \
sudo install /tmp/docker-machine /usr/local/bin/docker-machine
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
sudo curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

docker-machine create --driver : will run more nodes, docker VMs

 

Containers - building block of docker toolkit

docker info - shows most configuration values for the engine

 

Docker command format

new “management commands” format:

docker \<command\> \<sub-command\> (options)

old way will works: docker \<command\> (options)

 

Image vs Container

-   a Image is the application we want to run

-   a Container is an instance of that image running as a process

-   you can have many containers running off the same image

-   Docker Hub - default image “registry”

 

docker container run --publish 80:80 nginx

docker container run --publish 80:80 --detach nginx

docker container ls

docker container stop CONTAINER_ID

docker container ls -a

docker container run --publish 80:80 --detach --name=webhost1 nginx

 

docker container logs webhost1

docker container top webhost1

docker container --help

 

docker container rm ID1 ID2 IDn

docker container rm -f CONTAINER_ID_RUNNING

 

docker image ls

 

**docker container top** - process list in one container

**docker container inspect** - details of one container config

**docker container stats** - performance stats for all containers

 

### Getting a Shell inside containers

 

**docker container run -it** - start new container interactively

**docker container exec -it** - run additional command in existing container

**docker container start -ai ubuntu**

 

docker container port webhost1

docker container inspect --format ‘{{ .NetworkSettings.IPAddress }}’ webhost1

 

### Networks: CLI Management

**docker network ls** - show networks

**docker network inspect** - inspect a network

**docker network create --driver** - create a network

**docker network connect** - attach a network to container

**docker network disconnect** - detach a network from container

 

 

Docker Networks - DNS

-   understand how DNS is the key to easy inter-container comms

-   see how it works by default with custom networks

-   learn how to use **--link** to enable DNS on default bridge(docker0) network

-   static IP’s and using IP’s for talking to containers is an anti-pattern; do
    your best to avoid it;

-   docker daemon has a buit-in DNS server that containers use by default

-   docker defaults the hostname to the container’s name, but you can also set
    aliases(eg. docker container exec -it my_nginx ping new_nginx; php talks to
    mysql and viceversa - resolution works both ways)

-   the default bridge does not have the DNS built in by default

-   docker container run --rm -it centos:7 bash

-   DNS Round Robin Test

-   ever since Docker Engine 1.11, we can have multiple containers on a create
    network respond to the same DNS address

    -   create a new virtual network(default bridge driver)

    -   create two containers from elasticsearch:2 image

    -   research and use **--net-alias search** when creating them to give them
        an additional DNS name to respond to

    -   run **alpine nslookup search** with **--net** to see the two containers
        list for the same DNS name

    -   run **centos curl -s search:9200** with **--net** multiple times until
        you see both “name” fields show

 

docker network create dude

docker container run -d --net dude --net-alias search elasticsearch:2

docker container run -d --net dude --net-alias search elasticsearch:2

docker container ls

 

docker container run --rm --net dude apline nslookup search

docker container run --rm --net dude centos curl -s search:9200

 

docker container ls

 

### What is an image?

-   an image contains app binaries and dependencies and also

-   metadata about the image data and how to run the image

-   it’s not a complete OS; no kernel, kernel modules(eg. drivers)

-   small as one file(your app binary) like a golang static library

-   big as Ubuntu distro with app, apache...

 

### Image layers and the union file system

-   history and inspect commands

-   copy on write

-   docker history nginx:latest - show layers of changes made in image

-   every image starts with a blank layer - scratched - and every change will be
    another layer

-   it uses a unique SHA for each layer

-   CoW - Copy on Write - container will hold the diff related to base image

-   **docker image inspect** return metadata about the image

-   images are made up of file system changes and metadata

-   each layer is uniquely identified and only stored once on a host

-   this saves storage space on host and transfer time on push/pull

-   a container is just a single read/write layer on top of image

-   **docker image history** and **inspect **commands can teach us

 

 

### Image Tagging and Pushing to Docker Hub

docker image tag --help

docker image tag nginx curiaa/nginx

docker image push curiaa/nginx

docker login/logout

cat .docker/config.json

 

### Building images

docker build -f some-dockerfile

docker image build -t customnginx .

**package manager** - PM’s like apt and yum are one of the reasons to build
containers FROM Debian, Ubuntu, fedora or CentOS

**environment variables** - one reason they where chosen as preferred way to
inject key/value is they work everywhere, on every OS and config

 

### Container Lifetime and Persistent Data

-   containers are usually immutable and ephemeral

-   “immutable infrastructure”: only re-deloy containers, never change

-   this is the ideal scenario, but what about databases, or unique data?

-   a container should not contain the unique data; Docker gives us features to
    ensure these “separation of concerns”

-   Docker has two solutions to this problem: Persistent Volumes and Bind Mounts

    -   Persistent Volumes - make special location outside of container UFS

    -   Bind Mount - link container path to host path

 

### Persistent Data : Volumes

docker volume prune - cleanup unused volumes

docker volume ls

docker volume inspect 3424fgjdskf

named volumes - friendly way to assign vols to containers

docker volume create - required to do this before “docker run” to use custom
drivers and labels

 

### Persistent Data : Bind Mounting

-   maps a host file or directory to a container file or directory

-   basically just two locations pointing to the same file(s)

-   again, skips UFS, and host files overwrite any in container

-   can’t use in Dockerfile, must be at **container run**

-   ... run -v /users/adrianc/stuff:/path/container

-    

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 
