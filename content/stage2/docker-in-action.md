---
title: "Docker in Action"
date: 2023-01-16T14:30:45+08:00
---

Introduction
Chapter 1: Welcome to Docker
Docker make it easier for users to install and run the software.
Docker is a tool helping solve common problems such as installing, removing, upgrading, distributing, trusting and running software, helping system administrators focus on higher-value activities.
What is Docker?
Docker is an open source project for building, shipping and running programs.
containers
Install Docker
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
Take care of the setting of proxy
https://docs.docker.com/config/daemon/systemd/#httphttps-proxy
https://docs.docker.com/network/proxy/
Hello World
本机上没有这个image
本机上有这个image
Containers and Docker
Containers can isolate a process from all resources except where explicitly allowed.
However, it is challenging to to manually built correctly.
Docker solved this problem. Docker makes container simpler to use.
Containers are not virtualization
Virtual machines require high resource overhead (because they run a whole OS) and requires a long time to startup.
Docker (containers) solved this problem.
Container is an OS feature.
Running software in containers for isolation

Programs run inside a container can access only their own memory and resources as scoped by the container.
Docker build container using 10 major system features to suit the needs of the contained software and fit the environment the container will run.
Shipping containers
image: component in the container (shipping units)
Docker image is a bundled snapshot of all files needed for program running inside a container.
Containers that were started from the same image don’t share changes to their filesystem.
registries and indexes
What problems does Docker solve?
Get organized

Improve portability 便携性
On macOS and Windows, Docker uses a single, small virtual machine to run all the containers.
Developer could focus on writing programs on a single platform.
Protect your computer
also protect from the software inside a container

Why is Docker Important? & When and where to use Docker?
Docker provides an abstraction.

Keep your computer clean.

Security issues.

Getting help with the Docker command line
docker help 

docker help <command> 

e.g. docker help ps 

Part 1: Process isolation and environment-independent computing
Chapter 2: Running software in containers
Controlling containers: Building a website monitor


NGINX and mailer run as detached containers.
Detached means the container will run in the background, without being attached to any input or output stream.
Watcher runs as interactive container.
Steps:

Creating and starting a new container: detached containers
docker run --detach --name web nginx:latest 
--detach  or -d means the program is started in the background and is not attached to your terminal (cannot be found in ps -l ), called daemon
Running interactive containers
docker run --interactive --tty --link web:web --name web_test busybox:1.29 /bin/sh 
--interactive : keep the standard input open for the container even if no terminal is attached
--tty : allocate a virtual terminal for the container
Usually use both of these when running an interactive program
docker run -it --name agent --link web:insideweb --link mailer:insidemailer dockerinaction/ch2_agent 
Hold ctrl and press P and then Q to return to the shell.
Listing, stopping, restarting, and viewing output of containers
docker ps  shows information for each running containers
docker logs container_name check the logs for container
docker stop container_name stop a container
Solved problems and the PID namespace
Docker creates a new PID namespace for each container by default

docker exec container_name command  run a command in a running container
docker exec container_name ps  to show all the running processes and their PID in the container
docker run --pid host busybox:1.29 ps to create containers without their own PID namespace.
could see all process in the host server.
Conflict problems could be solved by docker due to environment independence

Other conflict problems that docker could solve:

By using Linux namespaces, resource limits, filesystem roots, virtualized network components.
Eliminating metaconflicts: Building a website farm
Flexible container identification: deal with container name collisions
docker rename new-name old-name  rename a container
container ID, could use the first 12 chars.
docker create creates a container in a stopped state.
docker create --cidfile /tmp/web.cid nginx  write container ID (CID) to a known file.
Docker won't create a new container if the provided CID file already exits.
To avoid conflict, we can use a partition path, such as /containers/web/customer1/web.cid
docker ps --latest --quiet get the CID of the last container.
docker could generates human-readable names for each container ( --name flag override it)
Container state and dependencies

docker ps -a  to see all the containers
The order of starting containers
generate an IP addr when creating containers
Building environment-agnostic systems
To help build environment-agnostic system, Docker has three specific features:

Read-only filesystems
Environment variable injection
Volumes
Read-only filesystems
docker inspect --format "{{.State.Running}}" wp  inspect the container metadata directly.
--format with Go template
docker diff CONTAINER Inspect changes to files or directories on a container's filesystem
docker run -d --name wp2 --read-only -v /run/apache2/ --tmpfs /tmp wordpress:5.0.0-php7.2-apache 
#!/bin/sh
DB_CID=$(docker create -e MYSQL_ROOT_PASSWORD=ch2demo mysql:5.7)
docker start $DB_CID
MAILER_CID=$(docker create dockerinaction/ch2_mailer)
docker start $MAILER_CID
WP_CID=$(docker create --link $DB_CID:mysql -p 80 \
--read-only -v /run/apache2/ --tmpfs /tmp \
wordpress:5.0.0-php7.2-apache)
docker start $WP_CID
AGENT_CID=$(docker create --link $WP_CID:insideweb \
--link $MAILER_CID:insidemailer \
dockerinaction/ch2_agent)
docker start $AGENT_CID
Environment variable injection
Environment variable: change programs' configuration without modifying any files
docker run --env (-e) ENVIRONMENT_VAR=my_environment_vat busybox:1.29 env  Injects an environment variable

Building durable containers
Restore the container service as quickly as possible.

Basic strategy: automatically restarting a process when it exits or fails.

Automatically restarting containers
--restart flag

Backoff strategy: determines the amount of time that should pass between successive restart attempts.
Exponential backoff strategy
Issues: containers waiting to be restarted are in the restarting state, and we cannot do anything that requires the container to be in a running state (such as docker exec )
Using PID 1 and init systems
Init system: a program that's used to launch and maintain the state of other programs.
e.g.: runit, Yelp/dumb-init, tini, supervisord, and tianon/gosu.
docker run -d -p 80:80 --name lamp-test tutum/lamp 
docker top CONTAINER Display the running processes of a container
An alternative method of init system: using a startup script
docker run --entrypoint="cat" wordpress:5.0.0-php7.2-apache /usr/local/bin/docker-entrypoint.sh 
--entrypoint
Docker containers run something called an entrypoint before executing the command
Cleaning up
docker rm CONTAINER
For container in running process, using docker stop first (better), or using docker rm -f CONTAINER (for quick stop).
docker run --rm  Automatically remove the container as soon as it enters the exited state.
docker rm -vf $(docker ps -a -q) Clean (kill) all the containers.
Chapter 3: Software installation simplified


Identifying software
What is a named repository?
It is a named bucket of images, similar to a URL.



A repository can hold several images, identified uniquely with tags.

Name and tag form a composite key, e.g., nginx:latest

Using tags
A tag can only attach to one image, while one image can have several tags.

create useful aliases
different tags for different versioning
different tags for different software configurations
Finding and installing software


Find software using index. Docker Hub is one of the public Docker indexes, and is the default registry and index used by docker.

Working with Docker registries from the command line
docker login before publishing images.

Using alternative registries
Registry address is part of the full repository specification.

[REGISTRYHOST:PORT/][USERNAME/]NAME[:TAG] 

Just specify the registry host, e.g., docker pull quay.io/dockerinaction/ch3_hello_registry:latest 

Working with images as files
Using docker load command.

Before this, we need to save one image from a loaded image docker save -o filename.tar busybox:latest 



Remove image from docker docker rmi busybox 

Load it again docker load -i myfile.tar 

Check it docker images to list images.

Installing from a Dockerfile
git clone https://github.com/dockerinaction/ch3_dockerfile.git

docker build -t dia_ch3/dockerfile:latest ch3_dockerfile 

Using Docker Hub from the website
https://hub.docker.com

Installation files and isolation
layer: a set of files and file metadata that is packaged and distributed as an atomic unit, called intermediate images.

Image layers in action
Layer relationships
Docker can uniquely identifies images and layers, it is able to recognize shared image dependencies between applications and avoid download those dependencies again.



Container filesystem abstraction and isolation: Union filesystem
Union filesystem (UFS) is used to create mount point and abstract the use of layers.

When Docker creates a container, that new container will have its own MNT namespace, and a new mount point will be created for the container to the image.
chroot is used to prevent from referring any other part of the host filesystem in the container.
Benefits of this toolset and filesystem structure
Common layers need to be installed only once.
Layers could manage dependencies and separating concerns.
Create software specializations when you can layer minor changes on top of a basic image.
Weaknesses of union filesystems
Often need to translate between the rules of different filesystems.
Chapter 4: Working with storage and volumes
Manage data with containers: where is the data stored? what happens to that data when you stop the container or remove it?

File trees and mount points
the image that a container is created from is mounted at that container’s file tree root, or the / point
every container has a different set of mount points
Containers get access to storage on the host filesystem and share storage between containers: mount nonimage-related storage at other points in a container file tree.
Three types of storage mounted into containers:
bind mount
in-memory storage
docker volumes

--mount flag in docker run and docker create 
Bind mounts
Bind mounts attach a user-specified location on the host filesystem to a specific point in a container file tree.

host provides files or dictionaries need by the program running in a container.
containerized program produces a file or log need to be processed outside containers.
An example



touch ~/example.log
cat >~/example.conf <<EOF
server {
listen 80;
server_name localhost;
access_log /var/log/nginx/custom.host.access.log main;
location / {
root /usr/share/nginx/html;
index index.html index.htm;
}
}
EOF
CONF_SRC=~/example.conf;
CONF_DST=/etc/nginx/conf.d/default.conf;
LOG_SRC=~/example.log;
LOG_DST=/var/log/nginx/custom.host.access.log;
docker run -d --name diaweb \
> --mount type=bind,src=${CONF_SRC},dst=${CONF_DST},readonly=true \
> --mount type=bind,src=${LOG_SRC},dst=${LOG_DST} \
> -p 80:80 \
> nginx:latest
src : source location on the host file tree (name of the volume, for anonymous volumes, this field is omitted)
dst : destination location on the container file tree
readonly=true prevent modifying the content of the volume
Test: docker exec diaweb sed -i "s/listen 80/listen 8080/" /etc/nginx/conf.d/default.conf 
Problems:

portable
conflict with other containers
In-memory storage
Private files should use in-memory storage, instead of including in an image or writing them to disk.

docker run --rm \
--mount type=tmpfs,dst=/tmp,tmpfs-size=16k,tmpfs-mode=1770 \
--entrypoint mount \
alpine:latest -v
Set the type option on the mount flag to tmpfs.
File permissions 1770 (1777 for default)
Size limit 16k
Docker volumes
Introduction to volumes
It is used to organizing data: in disk space that can be accessed by containers.

do not need to concern the system (fill on any machine with Docker installed).
easy to clean up for Docker (life cycle)
It could decouple storage from specialized locations on the filesystem that you might specify with bind mounts, and could run o any machine without considering conflict.

Using docker volume subcommand set.

docker volume create \
--driver local \
--label example=location \
location-example
 
docker volume inspect \
--format "{{json .Mountpoint}}" \
location-example


–-driver local: create a directory to store the contents of volume in a part of the host filesystem under control of the Docker engine.


How to share volumes between containers without exposing the exact location of managed containers?

Volumes provide container-independent data management
images are appropriate for packaging and distributing relatively static files such as programs (reusable)
volumes hold dynamic data or specializations (simple to share).
Polymorphic 多态：using volumes, do not need to modify an image.

Using volumes with a NoSQL database
Create a single-node Cassandra cluster, create a keyspace, delete the container, and then recover that keyspace on a new node in another container:



docker volume create \
--driver local \
--label example=cassandra \
cass-shared
 
docker run -d \
--volume cass-shared:/var/lib/cassandra/data \
--name cass1 \
cassandra:2.2
Shared mount points and sharing files
Sharing access to the same set of files between multiple containers.

Bind mounts methods: may lead to conflict.
Use volumes: do not need the knowledge of the underlying host filesystem.
Anonymous volumes and the volumes-from flag
Volume is created without a human-friendly name, and is assigned a unique identifier to eliminate volume-naming conflicts.

--volumes-from : copy the definition from other containers to the new container. (copy the full volume definition, including mount point, permission)

Situations not suit for --volumes-from :
when you need tomount to a different location
conflict: volumes from different containers with the same mount point

docker run --name chomsky --volume /library/ss \
alpine:latest echo "Chomsky collection created."
docker run --name lamport --volume /library/ss \
alpine:latest echo "Lamport collection created."
 
docker run --name student \
--volumes-from chomsky --volumes-from lamport \
alpine:latest ls -l /library/
 
docker inspect -f "{{json .Mounts}}" student
 
# Only one of the two containers is mounted.
when you need to change the write permission of a volume
Cleaning up volumes
docker volume list to list all the volumes

docker volume remove VOLUME_NAME/Unique_identifier to remove the volume

support list argument, such as docker volume remove amazon google microsoft
volume in use cannot be remove
docker volume prune delete all volumes that can be deleted

--filter : filter
–-force : without confirmation step
Advanced storage with volume plugins
For a cluster of machines, need to choose storage infrastructure, such as network storage

need to select proper Docker plugin.

Chapter 5: Single-host networking
How to connect a container to the network.

Networking background
Basics: Protocols, interfaces, and ports
Protocol: communication and networking in a sort of language, such as HTTP
Interface: address, represents a location
IP address: unique, information about their location on their network
Ethernet interface and loopback interface
Port: a recipient or a sender
Bigger picture: Networks, NAT and port forwarding
bridge: an interface that connects multiple networks so that they can function as a single network.
selectively forwarding traffic between the connected networks based on another type of network address.
Docker container networking
Benefits: 1. keep environment agnosticism 2. adopt specific network implementation

Problems: hard to determine the IP address of host, and inhibits container to other services outside the container network.

docker network subcommands

docker network ls to list all network of Docker.


bridge: inter-container connectivity for all containers running on the same machine
host: not create any networking namespace for attached container. Container will interact with host's network like uncontained processes.
none: no network connectivity
scope: local, global and swarm
local: the network is constrained to the machine where the network exists
global: created on every node in a cluster but not route between them
swarm: span all of the hosts participating in a Docker swarm
Creating a user-defined bridge network
Local to the machine where Docker is installed, creates route between participating containers and the wider network where the host is attached.



shape bridge network to fit your environment

docker network create \
--driver bridge \
--label project=dockerinaction \
--label chapter=5 \
--attachable \
--scope local \
--subnet 10.0.42.0/24 \
--ip-range 10.0.42.128/25 \
user-network
 
docker network create \
--driver bridge \
--label project=dockerinaction \
--label chapter=5 \
--attachable \
--scope local \
--subnet 10.0.43.0/24 \
--ip-range 10.0.43.128/25 \
user-network2
attach a running container to more than one network

docker run -it \
--network user-network \
--name network-explorer \
alpine:3.8 \
sh
 
ip -f inet -4 -o addr
 
docker network connect \
user-network2 \
network-explorer
 
ip -f inet -4 -o addr
nmap -sn 10.0.42.* -sn 10.0.43.* -oG /dev/stdout | grep Status
what those networks look like to running software inside an attached container

docker run -d \
--name lighthouse \
--network user-network2 \
alpine:3.8 \
sleep 1d
 
nmap -sn 10.0.42.* -sn 10.0.43.* -oG /dev/stdout | grep Statu
For situations that route traffic between containers on different machines, can use underlay networks provided by the macvlan or ipvlan network drivers.

or use swarm mode
Special container networks: host and none
software inside the container have the same degree of access to the host network as software outside container

docker run --rm \
--network host \
alpine:3.8 ip -o addr
software in the container cannot reach anything outside the container (network isolate)

docker run --rm \
--network none \
alpine:3.8 ip -o addr
 
docker run --rm \
--network none \
alpine:3.8 \
ping -w 2 1.1.1.1
Handling inbound traffic with NodePort publishing
Need to tell Docker how to forward traffic from the external network interfaces: specify TCP or UDP port.

NodePort publishing: match Docker and other ecosystem projects.

Provided at container creation time and cannot be changed later docker run/create --publish colon-delimited string argument.

Examples:

docker run --rm \
-p 8080 \
alpine:3.8 echo "forward ephemeral TCP -> container TCP 8080"
Host operating system will select a random host port, and traffic will be routed to port 8080 in the container.
Since ports are scarce resources, and this way will avoid potential conflicts.
Programs running inside a container do not know which port is being forwarded from the host
use docker port listener to look up port mappings
lookup special container port and protocol

docker run -d \
-p 8080 \
-p 3000 \
-p 7500 \
--name multi-listener \
alpine:3.8 sleep 300  
 
docker port multi-listener 3000
Container networking caveats and customizations
No firewalls or network policies
Docker bridge networks do not provide any network firewall or access-control functionality.

Custom DNS configuration
at /etc/hosts inside your container

docker run --hostname  set hostname of a new container, other containers do not know this hostname.
docker run -- dns  use an external DNS server
docker run --dns-search=[] provide a default hostname suffix.
e.g., docker run --rm --dns-search docker.com alpine:3.8 nslookup hub  will resolve the IP addr of hob.docker.com
docker run --add-host 
Externalizing network management
use Docker none network + third party container-aware tools.

Chapter6: Limiting risk with resource controls
Set resource allowances
By default, Docker containers may use unlimited CPU, memory, and device I/O resources.

Memory limits
docker run/create --memory <number><optional unit> where unit = b, k, m or g 
Note that they are not reservations, and only a protection from overconsumption.
Is the memory enough for the container? docker stats CONTAINER_NAME 
Some programs may fail with a memory access fault, whereas others may start writing out-of-memory errors to their logging.
--restart 
CPU
specify the relative weight of a container to other containers
--cpu-shares flag.
Only enforced only when there is contention for time on the CPU
limit the total amount of CPU used by a container
--cpus the number of CPU cores the container should be able to use
enforced
assign a container to a specific CPU set.
avoid context switch
Access to devices
More like resource-authorization control than a limit.

--device : + a map between the device file on the host operating system and the location inside the new container

Sharing memory
Docker creates a unique IPC namespace for each container by default, which prevents processes in one container from accessing the memory on the host or in other containers

Sharing IPC primitives between containers
--ipc flag

run programs that communicate with shared memory in different containers
create a new container in the same IPC namespace as another target container


sharing memory with host --ipc=host 
Understanding users
User is specified by the image metadata by default, often the root user.

Working with the run-as user
docker inspect --format "{{.Config.User}}" busybox:1.29 : if blank, the container will default to running as the root user.

user might be changed
docker container run --rm --entrypoint "" busybox:1.29 whoami 

docker container run --rm --entrypoint "" busybox:1.29 id 

To run as specific user when creating the container, the username must exist on the image you are using.

get a list of available user in an image with docker container run --rm busybox:1.29 awk -F: '$0=$1' /etc/passwd 
docker container run --rm --user nobody busybox:1.29 id 
Users and volumes
User ID space is shared: both root on the host and root in the container have user ID 0.

container's root can access a file owned by root on the host
do not mount the file into that container with a volume
echo "e=mc^2" > garbage
chmod 600 garbage
sudo chown root garbage
docker container run --rm -v "$(pwd)"/garbage:/test/garbage \
-u nobody \
ubuntu:16.04 cat /test/garbage
docker container run --rm -v "$(pwd)"/garbage:/test/garbage \
-u root ubuntu:16.04 cat /test/garbage
# Outputs: "e=mc^2"
Specify the desired user and group ID to avoid file permission conflicts.

mkdir logFiles
sudo chown 2000:2000 logFiles
docker container run --rm -v "$(pwd)"/logFiles:/logFiles \
-u 2000:2000 ubuntu:16.04 \
/bin/bash -c "echo This is important info > /logFiles/important.log"
docker container run --rm -v "$(pwd)"/logFiles:/logFiles \
-u 2000:2000 ubuntu:16.04 \
/bin/bash -c "echo More info >> /logFiles/important.log"
Be careful about which user can control the Docker daemon.

Introduction to the Linux user namespace and UID remapping
By default, Docker containers do not use the USR namespace

A container running with a user ID that is the same as a user on the host machine has the same host file permissions as that user.
When user namespace is enable: UID namespace remapping

remap to unprivileged UID
dockremap user
Adjusting OS feature access with capabilities
Capabilities: OS feature authorization

List all default caoabilities:

docker container run --rm -u nobody \
ubuntu:16.04 \
/bin/bash -c "capsh --print | grep net_raw"


--cap-drop : drop capabilities during container run/create

--cap-add : add capabilities

Running a container with full privileges
Run a system administration task: privileged containers

Privileged containers maintain their filesystem and network isolation but have full access to shared memory and devices and possess full system capabilities.

--privileged for docker container create/run

Strengthening containers with enhanced tools
Specifying additional security options --security-opt 

SELinux: labeling

AppArmor: file path

Building use-case-appropriate containers
Start with the most isolated container you can build and justify reasons for weakening those restrictions.

Or use default container construction.

Applications
running as a user with limited permissions
limit the system capabilities
set limits on how much of the CPU and memory the application can use
specifically whitelist devices that it can access
High-level system services
Often require privileged access, including cron, syslogd, sshd and so on.

Use capabilities to tune their access.

Low-level system services
Require privileged access, including devices and system's network stack. Rare to run inside containers.

Used for short-running configuration containers: change the configuration with a privileged containers.

Part 2: Packaging software for distribution
Chapter 7: Packaging software in images
Building Docker images from a container


Tips: docker run is the same as docker container run. https://stackoverflow.com/questions/51247609/difference-between-docker-run-and-docker-container-run

docker container diff CONTAINER_NAME review the list of files that have been modified in a container (added A, changed C, deleted D)
docker container commit  commit changes to new image
-a flag: author
-m flag: commit message
add --entrypoint
Parameters that carry forward with an image created from the container.  
all environment variables
working directory
the set of exposed ports
all volume definitions
the container entrypoint
command and arguments
Note: If not specifically set for the container, the values will be inherited from the original image
Going deep on Docker images and layers
Exploring union filesystems
Each time a change is made to a union filesystem, that change is recorded on a new layer on top of all of the others



When reading a file, the file will be read from the topmost layer where it exist.

If a file was not created or changed on the top layer, the read will fall through the layers until it reaches a layer where that file does exist
When a file is deleted, a delete record is written to the top layer, which hides any versions of that file on lower layers.

When a file is changed, that change is written to the top layer, which again hides any versions of that file on lower layers



Reintroducing images, layers, repositories, and tags
Metadata of layer: generated identifier, identifier of the layer below it, execution context.

Image: stack of layers, constructed by starting with a given top layer and then following all the links



docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]] commit container with name and tag.

docker tag  copy image by creating a new tag or repo from existing one.

All layers below the writable layer created for a container are immutable.

they can never be modified
make individual layers reusable
share access to images instead of creating independent copies for every container
any time making changes to an image, you need to add a new layer, and old layers are never removed.
Managing image size and layer limits
Union filesystem makes a file as deleted by actually adding a file to the top layer.

docker image history IMAGE show the history of an image.

Exporting and importing flat filesystems
image & TAR file: bad idea, will lose the original image's metadata and change history.

docker image save save the image to a TAR file.
docker image import import the TAR file into docker.
docker import -c "ENTRYPOINT [\"/hello\"]" - \
dockerinaction/ch7_static < static_hello.tar
-c specify a Dockerfile command
- at the end of the first line: the content of the tarball will be streamed through stdin. This position can be file/URL/-
small size, only one layer (called flat image), no layer reuse
use creating branch
docker container export Export a container's filesystem as a tar archive

docker cp  Copy files/folders between a container and the local filesystem

Versioning best practices


latest: default tag, should point to the latest stable build of its software instead of the true latest.



Each row represents a distinct image.

Chapter 8: Building images automatically with Dockerfiles
Dockerfile: text file containing instructions for building an image.

Packaging Git with Dockerfile
# An example Dockerfile for installing Git on Ubuntu
FROM ubuntu:latest
LABEL maintainer="dia@allingeek.com"
RUN apt-get update && apt-get install -y git
ENTRYPOINT ["git"]
docker image build --tag ubuntu-git:auto . 

#: comments
The first instruction of Dockerfile must be FROM
. : the location of the Dockerfile is the current directory.
--file specify the name of Dockerfile, it usually be xxxx.df
–quite / -q suppress the output of the build process
The use of caching: the builder can cache the results of each step. If a problem occurs after several steps, the builder can restart from the same position after the problem has been fixed.

--no-cache : disable caching
A Dockerfile primer
Metadata instructions
Benefits of using Dockerfile: simplify copying files from host computer into an image

define which files should never be copied into any images: .dockerignore 
Each Dockerfile instruction will result in a new layer being created, so merge instructions to minimize the size of images and layer count when possible.

FROM debian:buster-20190910
LABEL maintainer="dia@allingeek.com"
RUN groupadd -r -g 2200 example && \
useradd -rM -g example -u 2200 example
ENV APPROOT="/app" \
APP="mailer.sh" \
VERSION="0.6"
LABEL base.name="Mailer Archetype" \
base.version="${VERSION}"
WORKDIR $APPROOT
ADD . $APPROOT
ENTRYPOINT ["/app/mailer.sh"]
EXPOSE 33333
# Do not set the default user in the base otherwise
# implementations will not be able to update the image
# USER example:example


Some Dockerfile instructions:

FROM : set the layer stack to start from which image.
ENV : set environment variables for an image, similar to --env flag on docker container run/create
can also be used in other instructions in this Dockerfile
LABEL : defile key:value pairs, similar to --label on docker run/create  
add additional metadata, will be available to process running inside a container
WORKDIR : set default working directory
EXPOSE : open TCP port
ENTRYPOINT : set the executable to run at container startup
has two forms: shell form and exec form
if the shell form is used, all other arguments provided by CMD instruction or extra arguments will be ignored.
app/mailer.sh does not exist
USER set user and group for further build steps
set up user and group accounts in base image
let the implementations set the default user
use docker inspect to inspect the environment variables of the image.
ENV, LABEL, WORKDIR, VOLUME, EXPOSE, ADD, COPY 
Filesystem Instructions
Some instructions that modify the filesystem

FROM dockerinaction/mailer-base:0.6
RUN apt-get update && \
apt-get install -y netcat
COPY ["./log-impl", "${APPROOT}"]
RUN chmod a+x ${APPROOT}/${APP} && \
chown example:example /var/log
USER example:example
VOLUME ["/var/log"]
CMD ["/var/log/mailer.log"]
docker image build -t dockerinaction/mailer-logging -f mailer-logging.df . 

COPY : at least two argus, source files + destination
file ownership set to root
support shell style and exec style arguments (exec is better)
ADD : 
fetch remote source files if a URL is specified (not good, did not provide mechanism for cleaning up unused files and results in additional layers)
extract the files of any resource determined to be an archive file (auto-extraction)
VOLUME : 
similar to --volume on docker run/create 
CMD : 
related to ENTRYPOINT 

represents an argument list for entrypoint
if no ENTRYPOINT is set, ignore this instruction
If ENTRYPOINT is set and using exec form, you use CMD to set default arguments.
Tip: what is shell style and exec style?



shell style: looks like a shell command with whitespace-delimited arguments
can use "\" to continue a single instruction onto the next line
e.g.:

RUN 'source $HOME/.bashrc;\ 
echo $HOME '
exec form: a string array in which the first value is the command to execute, the remaining values are argus.
e.g.: RUN ["/bin/bash", "-c", "echo hello"] 
Injecting downstream build-time behavior
ONBUILD :  defines other instructions to execute if the resulting image is used as a base for another build.

recorded in the resulting image's metadata under ContainerConfig.OnBuild 
in the downstream Dockerfiles, those ONBUILD instructions are executed after FROM instruction and before the next instruction.
Creating maintainable Dockerfiles
ARG : define variable that users can provide to Docker when building an image.

--build-arg <varname>=<value> 
Multistage Dockerfile: a Dockerfile that has multiple FROM instructions

each FROM instruction makes a new build stage
FROM ... AS <name>  to name build stage, this name can be used in subsequent FROM and COPY --from=<name|index> 
example: 

#################################################
# Define a Builder stage and build app inside it
FROM golang:1-alpine as builder
# Install CA Certificates
RUN apk update && apk add ca-certificates
# Copy source into Builder
ENV HTTP_CLIENT_SRC=$GOPATH/src/dia/http-client/
COPY . $HTTP_CLIENT_SRC
WORKDIR $HTTP_CLIENT_SRC
# Build HTTP Client
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
go build -v -o /go/bin/http-client
#################################################
# Define a stage to build a runtime image.
FROM scratch as runtime
ENV PATH="/bin"
# Copy CA certificates and application binary from builder stage
COPY --from=builder \
/etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=builder /go/bin/http-client /http-client
ENTRYPOINT ["/http-client"]
Using startup scripts and multiprocess containers
Environmental preconditions validation
failing fast and precondition validation are best practices in software design.

Image author can introduce environment and dependency validation prior to execution of the main task, and to better inform user if the container is failed.

e.g., WordPress use a script as the container entrypoint to validate environment variables and container links.
The startup process could validate

Presumed links (and aliases)
Environment variables
Secrets
Network access
Network port availability
Root filesystem mount parameters (read-write or read-only)
Volumes
Current user
Can use shell scripts to finish the above validations.

Example: validate either another container has been linked to the web alias and has exposed port 80, or the WEB_HOST environment variable has been defined:

#!/bin/bash
set -e
if [ -n "$WEB_PORT_80_TCP" ]; then
  if [ -z "$WEB_HOST" ]; then
    WEB_HOST='web'
  else
    echo >&2 '[WARN]: Linked container, "web" overridden by $WEB_HOST.'
    echo >&2 "===> Connecting to WEB_HOST ($WEB_HOST)"
  fi
fi
if [ -z "$WEB_HOST" ]; then
  echo >&2 '[ERROR]: specify container to link; "web" or WEB_HOST env var'
  exit 1
fi
exec "$@" # run the default command
The same containers may fail later for other reasons →  can combine startup scripts with container restart policies.

Initialization processes
Initialization (init) process in UNIX-based computers to start all the other system services, keep them running, and shut them down.

Use light-weight init-style system for container as entrypoint, such as runit, tini, BusyBox init, Supervisord, and DAEMON tools.

--init: tini program by default 
evaluate using which init program.
The purpose and use of health checks
Health checks: check whether the application running inside the container is able to perform its function.

health check command:

HEALTHCHECK instruction in Dockerfile: should be reliable, lightweight, and not interfere with the operation of the main application because it will be executed frequently (every 30 seconds by default)
0: success, 1: unhealthy, 2: reserved
three failed checks from healthy to unhealthy
|| exit 1 

Time-out : a time-out for health check command to run and exit
Start period : grace period 
docker run --health-cmd 
override the health check defined in the image if exists.
Building hardened application images
Hardening an image means reducing the attack surface inside any Docker containers based on it.

Strategy:

minimize the software included with the container
enforce that your images are built from a specific image
have a sensible default user
eliminate a common path for root user escalation from programs with SUID or SGID attributes set
Content-addressable image identifiers
Content-addressable image identifier (CAIID) refers to a specific layer containing specific content, to enforce a build from a specific and unchanging starting point.



Append an symbol @ followed by the digest in place of the standard tag position

Useful when incorporating known updates to a base into your images and identifying the exact build of the software running on your computer.

Prevent from changing without your knowledge.

User permissions
Docker cannot prevent from user run with root permission.

USER instruction in Dockerfile to set user and group, similar to docker container run/create .

The key is to determine the earliest appropriate time to drop privileges.

too early: active user may not have permission to complete the instructions in Dockerfile
Also need to consider the permissions and capabilities needed at runtime, such as system port range (1-1024)
Which user should be dropped into?

image user do not know about Docker daemon configuration about namespace remap.

gosu 
SUID and SGID permissions
SUID: file will always execute as its owner, such as /usr/bin/passwd.

Have the risk of compromise the root account inside a container.

The files with SUID and SGID are rarely required for application use cases.

Unset the SUID and SGID permission for all files in current image:

RUN for i in $(find / -type f \( -perm /u=s -o -perm /g=s \)); \
do chmod ug-s $i; done


Chapter 9: Public and private software distribution
Choosing a distribution method
How to choose the appropriate method for your situation.

A distribution spectrum


Selection criteria
Cost
Visibility: secret projects or public works
Transportation: transportation speed and bandwidth overhead
Longevity
Availability
...
Publishing with hosted registries
Hosted registry: a Docker registry service that is owned and operated by a third-party vendor.

e.g., Docker Hub, Quay.io, Google Container Registry
by default, Docker publishes to Docker Hub
Publishing with public repositories: "Hello World!" via Docker Hub
docker login  Maintains a map of your credentials for registries that you authenticate.

docker image push insert Docker Hub username>/hello-dockerfile  Push your repo to the host registry.

docker search username Searching repo owned by specific user.

Private hosted repositories
Before you can use docker image pull  or docker container run to install an image from a private repository, you need to authenticate with the registry where the repository is hosted

Introducing private registries
Using Docker's registry software

Using the registry image
Start a local registry in a container:

docker run -d -p 5000:5000 -v "$(pwd)"/data:/tmp/registry-dev --restart=always --name local-registry registry:2
docker image tag dockerinaction/ch9_registry_bound localhost:5000/dockerinaction/ch9_registry_bound
docker image push localhost:5000/dockerinaction/ch9_registry_bound
Where can we find the image localhost:5000/dockerinaction/ch9_registry_bound?



Registry stores the image on /var/lib/registry on its union filesystem, which is volume on the host filesystem by default. So we can find it in the volume of local-registry container. Use docker volume ls to list all the volumes. Use docker volume inspect xxx to display detailed information on the volume.

Note that we can use -v xxx:/var/lib/registry to mount the /var/lib/registry on a specific location.

For "$(pwd)"/data:/tmp/registry-dev , mount the volume into the container. The first part "$(pwd)"/data is the location of the volume on host machine, the second part is the location on the container union filesystem. Note that in this example, we did not mount /var/lib/registry on this location, so we cannot find the image push to the registry on "$(pwd)"/data .

When you’ve started the registry, you can use it like any other registry with docker pull, run, tag, push commands.

Conduct tight version control: pull images from external sources such as Docker Hub and copy them into their own registry. To ensure that an important image does not change or disappear
unexpectedly when the author updates or removes the source image.

Consuming images from your registry
docker image rm dockerinaction/ch9_registry_bound localhost:5000/dockerinaction/ch9_registry_bound
 
docker image pull localhost:5000/dockerinaction/ch9_registry_bound
This will prevent remote Docker clients from using your registry.

Manual image publishing and distribution
Work with images as files.

docker image save  + docker container export 

docker image load + docker image import 



Image source-distribution workflows
Using github.

Much flexible.

Chapter 10: Image pipelines
Image build pipeline: preparing image material, building an image, testing, publishing images to registries.

Goals of an image build pipeline


Application artifacts: runtime scripts and configuration files produced by software authors.

CI tools.

Patterns for building images
How many images an application includes?

All-in-One
Build plus Runtime
Build plus Multiple Runtimes


All-in-one images
Build the application into an image.

Downsides:

1. contain more tools than necessary, easy for attackers

2. the size will be large (708MB)

ENTRYPOINT ["java","-jar","/app.jar"] 

Separate build and runtime images
The build-specific tools such as Maven and intermediate artifacts are no longer included in the runtime image.

Much smaller size (from 708MB to 401MB)
Smaller attack surface.
Variations of runtime image via multi-stage builds
Multi-stage builds can be used to keep the specialized image synchronized with the application image and avoid duplication of image definitions.

Name a build stage:

easily to mention the stage by other stages.
build processes can build a stage by specifying that name as a build target.
docker image build --target : select the stage to build the image.

If do not specify a build stage, docker will build image from the last stage defined in the Dockerfile.

Add a trivial build stage at the end of the Dockerfile.

# Ensure app-image is the default image built with this Dockerfile
FROM app-image as default
Record metadata at image build time
Use LABEL instruction to capture metadata, at least including:

application name
application version
build date and time
version-control commit identifier
Commonly used labels http://label-schema.org/rc1/: label schema, e.g., LABEL org.label-schema.version="${BUILD_ID}" 

Orchestrating the build with make
Make: used to build programs, understands dependencies between the steps of a build process.



Steps: gathering metadata, building application, building, testing, tagging the image.

Use linting tool named hadolint to check Dockerfile.

Testing images in a build pipeline
Container Structure Test tool (CST) https://github.com/GoogleContainerTools/container-structure-test verifies the construction of a Docker image.

verify desired file permissions and ownership, commands execute with expected output, and the image contains particular metadata such as a label or command
the tool operates on arbitrary images without requiring any tooling or libraries to be included inside the image.
Patterns for tagging images
Important image-tagging features:

Tags are human-readable
Multiple tags may point to a single image ID
Tags are mutable and may be moved between images.
Background
Image tag mutation is commonly used to identify the latest image in a series.

However, it is difficult to identify what does the latest lag mean?

Continuous delivery with unique tags
Using unique BUILD_ID tag.

Inconvenient for users.

Configuration image per deployment stage
A generic, environment-agnostic application image.
A set of environment-specific configuration image.
Semantic versioning
A version number of the form Major.Minor.Patch. Author could increment the following:

Major version when making incompatible API changes
Minor version when adding functionality in a backward-compatible manner
Patch version when making backward-compatible bug fixes
make tag BUILD_ID=$BUILD_ID TAG=1.0.0 

Higher-level abstractions and orchestration
Chapter 11: Service with Docker and Compose
Service: any processes, functionality, or data that must be discoverable and available over a network is called a service.

A service "Hello World!"
Docker services are available only when Docker is running in swarm mode.

docker swarm init 



docker service create --publish 8080:80 --name hello-world dockerinaction/ch11_service_hw:v1 



Task is a swarm concept that represents a unit of work. Each task has one associated container.



Automated resurrection and replication
docker service ls  to list the running services.

docker service ps hello-world to list the containers associated with a specific service.



The desired state is what the user wants the system to be doing, or what it is supposed to be doing.

The current state describes what the system is actually doing.

Orchestrators remember how a system should be operating and manipulate it without being asked to do so by a user.

you need to understand how to describe systems and their operation.
docker service inspect hello-world to output the current desired state definition for the service:

Name of the service
Service ID
Versioning and timestamps
A template for the container workloads
A replication mode: replicated mode and global mode.
replicated mode (default) 
docker service scale hello-world=3 (use docker container ps  and docker service ps hello-world to show the effects.)
If you scale the service back down (docker service scale hello-world=2), you’ll also notice that higher-numbered containers are removed first.
global mode
Run one replica on each node in the swarm cluster.
Rollout parameters
Similar rollback parameters
A description of the service endpoint
Automated rollout


Illustration of a Docker swarm's action when deploying an update:



Service converged: the current state of the service is the same as the desired state described by the command.

Service health and rollback
docker service update --rollback hello-world 回滚



--update-failure-action to tell Swarm that failed deployment should roll back.

update-max-failure-ratio to tell Swarm to tolerate start failures as long as 0.6 of the fleet is good. After that, the whole deployment will be marked as failed and a rollback will be initiated.

Add health check:



At last, remove the service: docker service rm hello-world 

Declarative service environments with Compose V3
Imperative tools: such as docker service create 
Declarative tools: describe the new state of a system, rather than the steps required to change from the current state to the new state.
When the service is complex, the number of imperative commands required to achieve the goals is too large. Then we can adopt a higher-level declarative abstraction (docker stack).
Docker stack describes collections of services, volumes, networks, and other configuration abstractions.

These environments are described using Docker Compose V3 file format.

Compose files use YAML language, such as docker-compose.yml.

A YAML primer
Comment support: ( #)

Three types of data: maps (key: value, the value can be double-quote style or plain style), lists (start by -), scalar values

Two styles: flow collections, block style.

Use indentation to indicate content scope. (only use spaces)

Collections of services with Compose V3
docker stack deploy subcommands: create and update stacks.

Docker compose cannot handle delete well: e.g., The Compose file you provided to the stack deploy  command did not have any refer to the mariadb service, so Docker did not make any changes to that service.

using docker service remove 
executing docker stack deploy  with the --prune  flag
Stateful services and preserving data
Define volumes in compose file: 

volumes:
  volume_name: property (empty definition uses volume defaults)

The top-level property defines the volumes that can be used by services within the file.

Docker can attach the new service to the original volume.

Load balancing, service discovery, and networks with Compose
Port publishing for a service is different from publishing a port on a container: containers directly map the port on the host interface to an interface for a specific container, services might be made up of many replica containers.

For docker services, docker creating virtual IP (VIP) addresses and balancing requests for a specific service between all of the associated replicas.



First network ingress: handles all port forwarding from the host interface to services. Created when you initialize Docker in swarm mode.
Second network: shared between all of the services in your stack.
Services default attach to a network named default.
The address listed here is the container address, or the address that the internal load balancer would forward requests onto. Not the virtual IP addr you would see when inspecting.
Define network in compose file:

top-level networks property that includes network definitions
networks property of services






