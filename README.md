

# Hands on Docker 

An overview of docker and its functionalities.\
A cheat sheet of its common used commands\
A yet minimal, simple to be complex guide on docker.
Based on [The Docker Handbook](https://www.freecodecamp.org/news/the-docker-handbook/#introduction-to-containerization-and-docker "FreeCodeCamp") but not only.

## Table of contents
<details>
  <summary>Click to expand</summary>

+ #### [Containerization](#containerization)
+ #### [Dcoker architecture](#docker-architecture)
+ ### [Diverses manipulations](#div-manips)
  + [Basic manipulations](#basic-manips)
  + [Publish a port](#publish-port)
  + [Running a container in the background](#run-bkgd)
  + [Listing Containers](#listing-containers)
  + [Naming & Renaming Container](#naming-renaming-container)
  + [Stopping and killing a container](#stop-kill-container)
  + [Start and Restarting a Container](#start-restart-container)
  + [Creating a container w/o running](#creating-cont-wo-running)
  + [Removing Dangling Container](#remove-dang-container)
  + [Running a container in a interactive mode](#run-container-it-mode)
  + [Executing command inside a container](#exec-inside-cont)
  + [Working with executable images](#work-exec-images) 
</details>

## The idea behind containerization <a name="containerization"><a>


According to IBM
> Containuerization involves encapsulating or packaging software code and all its dependacies so that it can run uniformely and consistenly on any infrastructure

In a simple way, its lets you bundle up your software code along with all its depndacies in a self-contained package so that it can be run without going through a troublesome of setup process.

The key ideas are:
  - Develop and run the app inside an isolated environment (**container**) that matches your final deployement environment
  - Put the application in a single file (**image**) along with all its dependancies and necessary deployement configurations
  - Share that image through a central server (**registry**) that is accessible by anyone with proper authorization.

**Docker** is just an implementation like others (Podman by RedHat, Kaniko by Google, rkt by CoreOS) of the concept of containeurisation. In short **docker** allows you to containerize your applications, share them using public or private registries, and also to or [orchestrate](https://docs.docker.com/get-started/orchestration/) them.

### [Installation (Debian 10)](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-debian-10 "Digitalocean Installation guide")


# Very Firsts Commands

        $ docker run hello-world 
            + run the hello-world image
        
        $ docker ps -a 
            + list all past and current running containers
  
# Docker achitecture <a name="docker-architecture"></a>

   * [Container](#container) 
   * [Images](#images)
   * [Registry](#registry)
## What is a Container <a name="container"></a>


The offical Docker <u>[resources](https://www.docker.com/resources/what-container)</u> state:
> A container is an abstraction at the application layer that packages code and dependencies together. Instead of virtualizing the entire physical machine, containers virtualize the host operating system only.

Containers are completely isolated environments from the host system as well as from each other. They're a lot lighter than the traditional VM and a large number of containers can be run simultaneously without affecting the performance of the host system. Containers are considered to be the next generation of virtual machines.*

##### A little comparison between VM and Containers
-----------------------------------------
The main difference is the method of virtualization.


| Traditional VM| Containers |
| -----------| ----------- |
| Created, managed by a hypervisor (OracleVM, VirtualBox,..)     |Utilize the host OS via the container runtime while maintaining isolation    |
| The hypervisor sits between the host OS and the VM to act as a medium communication   | The container runtime (Docker) sits between the container and the host OS      |
| Comes with its own guest OS which is heavy as the host OS   | Doesn't need its own OS as it follows the idea behind containeurization       |
| The app inside the VM communicates with the guest OS which talks with the hypervisor  which talks to the host OS ...(long chain of communication)| The container communicate with the container runtime which communicates with the host OS to get necessary resources.       |
| With it own OS, VM are heavier and more resources consuming | By eliminating the entire host OS, containers are much lighter and less ressource-hogging     |

##### Illustrations
-------------------

| Traditional VM| Containers |
| -----------| ----------- |
| ![alt Virtual Machine](/images/virtual-machines.svg)| ![alt Containers](/images/containers.svg) |

These commands shows that docker uses the kernel of the host OS

        $ uname -a 
        $ docker run alpine uname -a

## Waht is a Docker Image <a name="images"></a>

Images are multi-layered self-contained files that act as the template for creating containers.They are like a frozen, read-only copy of a container and can be exchanged through registries. The OCI (Open Container Initiative) defined a standard specification for images which is complied by the major containerization engines out there. a docker image can be used with another runtime like Podman without any additional hassle.
Containers are just image in running states.

## What is a Docker Registry <a name="registry"></a>

A centralized place where you can upload your images and can also download images created by others. [Docker Hub](https://hub.docker.com/) is the default public registry for Docker and [Quay](https://quay.io/) is another popular registry by Red Hat.

## Docker Architecture Overview

Docker as a software was designing in consist of three major components:
  1. **Docker Daemon** : The daemon( <mark>dockerd</mark> is a process running in the background and awaits for commands from the client. 
  2. **Docker Client** : The client (</mark>docker</mark>) is a CLI program mostly responsible for transporting commands isued by users
  3. **RESt API** : The rest API acts as a bridge between the daemon and the client. Any commands issued using the client passes through the API to finally reach the daemon.
   
   Docker officials state: 

   "Docker uses a client-server architecture. The Docker *client* communicates with the the Docker _daemon_ which does heavey lifting of building, running, and distributing your Docker containers."

   ### The big picutre
   -------------------
   ![alt full process](images/docker-run-hello-world.svg)

   PS : if the the image is not found in the local repository <code>Unable to find image 'image-name'</code> will be printed on the terminal.
   The <mark>Daemon</mark> will reach out to default public registry (Docker Hub) and pulls in the last copy of the image if found.
   A new container will then be created from the fresly pulled image.
   By default, Docker Daemon look for images in the hub that are not present locally and once an image has been fetched, it will stay in the local cache.
   In case of newer images, the Daemon will fetch them automatically.

   ## Diverses manipulations <a name="div-manips"></a>

   ### Basic manipulations <a name="basic-manips"></a>
   -----------------------
   * <code>docker \<object> \<command> \<options> </code> command syntax\
             - \<object> : type of docker object to be manipulated\
             - \<command> task to be carried by the <mark>Daemon</mark>\
             - \<options> : any valid parameter that can override the default behaviour of the command.
  * <code>docker run \<image name> </code> or <code>docker container run \<image name></code>: run a container\
  * the \<option> <code>--publish or -p</code> is for port mapping\
  * <code>Ctrl +C </code> : stop the container
  

  ### Publish a port <a name="publish-port"></a>
  ------------------
  As isolated environments as containers, the OS doesn't know nothing about what's going on inside them. Hence applications running inside them reamin inaccessible from the outside.

  To allow access from the outside, We must publish the appropriate port inside the container to a port on your local network. The common sysntax is <code>--publish or -p</code>
  <code>--publish \<host port> \<container port></code>
  <code>docker run container --publish 8080:80 /repo/docker-image</code> : publish the image /repo/docker-image to the port 8080 locally.

### Running a container in the background <a name="run-bkgd"></a>
-----------------------------------------
Known as the **detach mode**, this mode keeps running the container int he background. The option to be used is <code>--detach or -d</code>
Note that this command output the full _CONTAINER ID_ of the newely created container on the terminal
<code>docker run container --detach --publish 808:80 /repo/docker-image</code>\
output: 9e634a231ea4e37f94ef747a8ca6bd2bc25c473699afc783d5dce836bc341090

PS: The order of options doesn't really matter but anything that is been put after the image name will be passed as an argument to the container entry-point. 

### Listing Containers <a name="listing-containers"></a>
------------------------
<code>docker container ls</code> is the command to be used to list running containers.
the last column
        
     docker container ls
     CONTAINER ID            IMAGE                 COMMAND                 CREATED           STATUS        PORTS                                   NAMES
     9e634a231ea4            fhsinchy/hello-dock   docker-entrypoint.…"    3 hours ago       3 hours ago   0.0.0.0:8080->80/tcp, :::8080->80/tcp   optimistic_tereshkov

The _CONTAINER ID_ is the first 12 characters of the full container ID printed out after running <code>docke container run</code>
The _NAME_ is generated by docker

<code>docker container ls -all | -a</code> will list all container that has ever run with the session in the OS.

### Naming and Renaming Container <a name="naming-renaming-container"></a>
---------------------------------
The option <code>--name</code> can be used to achieve the naming of a container and the option <code>--rename</code> can be used to rename an **existing container**.

<code>docker container run --detach --publish 8888:80 --name hello-dock-container fhsincy/hello-dock </code>
The above code will create a new container with a user defined name 'hello-dock-container' and not a docker generated name.

<code>docker container rename \<container identifer>  \<new name></code> will rename the name attached to container identifier(Id or name) to new name.
```shell
docker container rename optimistic_tereshkova new-nice-name
docker container ls
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                   NAMES
9e634a231ea4   fhsinchy/hello-dock   "/docker-entrypoint.…"   4 hours ago     Up 4 hours     0.0.0.0:8080->80/tcp, :::8080->80/tcp   new-nice-name
```
### Stopping and killing a container <a name="stop-kill-container"></a>
--------------------------------------
While container running on foreground can be stopped by closing the terminal or combining <code>Ctrl+C</code>, container running in the background need another docker command.
```shell
docker container stop <container ID or name>
```
Note: The above command will shut down a container gracefully by sending a <code>SIGTERM</code> signal. we may use the below command to send a <code>SIGKILL</code> signal.
```shell
docker container kill <container ID or name>
```
_PS: Stopped container remain in the system and can restarted._
### Start and Restarting a container <a name="start-restart-container"></a>
-----------------------------------

Two scenarios to be noticed here:
      
    - Restarting a container that has been previously stopped or killed
    - Rebooting a running container 
The command below will start any stopped or killed container
```shell
docker container start <container ID or name>
```
Stopped or killed container can be viewed with an <code>EXIT</code> status in the output of <code>docker contaienr ls --all</code>
```shell
docker container start hello-dock-container
```
Note that the container start will start a container in detached mode and will retain any port configuration made previously.
Rebooting a running container can be done with:
```shell
docker container restart <container ID or name>
```
The main difference between starting and restarting a container is that the <code>container restart</code> command attemps to stop the target container and start it back while the <code>start</code> command aims to start an already stopped command.
In case of a stopped container, both commands are exactly the same. But in case of a running container, you must use the container restart command.

### Creating a container without running <a name="creating-cont-wo-running"></a>
-----------------------------------------
```shell
docker container run <container-image>
```
The command above used two differents commands which are:
<code>docker container create</code> creates a container from a given name
<code>docker container start</code> starts an already created container
```shell
docker container create --publish 8080:80 fhsinchy/hello-dock
docker container start fhsinchy/hello-dock
```
The command above will create the hello-dock container.

### Removing dangling containers <a name="remove-dang-container"></a>
--------------------------------
Containers that have been stopped or or killed remain in the system. These dangling containers can take up spaces or create conflict with newer containers.
The command <code>container rm \<container ID or name></code> can be used in order to remove a stopped container.
```shell
docker container rm busy_mashivara
```
Note that multiple containers can be removed by passing their ID one after another separated by space.
To remove all dangling container, we can use the command below:
```shell
docker container prune
```
The option <code>--rm</code> for the <code>docker container run</code> allows to indicated that our container should be removed once stopped or killed.
```shell
docker container run --rm --detach --publish 8383:80 --name hello-volatile fhsinchy/hello-dock
```
### Running a container in an interactive mode <a name="run-container-it-mode"></a>
----------------------------------------------
Images can hold or can carry on programs that are interactive with the user running them, all images are not so simple as what've been seen previously.
Popular distribution such as Fedora, Ubuntu, Debian, ... have docker image in the hub. Programming language such as python, php, java, javascript, ... etc do also have their official image in the hub. Such images are configured to run a shell by command, in case of the OS it can be something like <code>bash</code> or <code>sh</code> and programming languages usually use their default shell.
The option <code>-it (or --interactive --tty)</code> for <code>container run</code> command will run an image in interactive mode as :
```shell
docker container run --rm -it ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
345e3491a907: Pull complete 
57671312ef6f: Pull complete 
5e9250ddb7d0: Pull complete 
Digest: sha256:cf31af331f38d1d7158470e095b132acd126a7180a54f263d386da88eb681d93
Status: Downloaded newer image for ubuntu:latest
root@20fc27827b42:/# uname -a
Linux 20fc27827b42 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64 x86_64 x86_64 GNU/Linux
root@20fc27827b42:/# ls
bin   dev  home  lib32  libx32  mnt  proc  run   srv  tmp  var
boot  etc  lib   lib64  media   opt  root  sbin  sys  usr
root@20fc27827b42:/# 
```
### Executing commands inside a container <a name="exec-inside-cont"></a>
-----------------------------------------
Image can also be configured to receive arguments and perform a certain task. Docker is set to treat every string and/or numeric that comes after the image name to be treated as an argument. 
```shell
docker run --rm alpine uname -a
Linux 68bc1f23d028 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64 Linux
```
In the command above the uname -a is passed through the alpine image and get executed. 

### Working with executable images <a name="work-exec-images"></a>
----------------------------------
Executable images are meant to behave as executable programs. 
Some of these images are designed to perform task with local OS files, it could be deleting corrupted files, creating or installing programs, ... etc.
Using **bind mounts** helps granting a container direct access to our local files system. A bind mount lets us perform a two way data binding between the container of a local file system directory (source) and another directory inside a container (destination). This way any changes made in the destination directory will take effect on the source directory and vise versa. 
```shell
touch hvt.pdf
fabric@km:~$ touch a.pdf c.pdf e.pdf off.pdf
fabric@km:~$ docker container run --rm -v $(pwd):/zone fhsinchy/rmbyext pdf
Unable to find image 'fhsinchy/rmbyext:latest' locally
latest: Pulling from fhsinchy/rmbyext
801bfaa63ef2: Already exists 
8723b2b92bec: Pull complete 
4e07029ccd64: Pull complete 
594990504179: Pull complete 
140d7fec7322: Pull complete 
23038161e8da: Pull complete 
0b40a42464a5: Pull complete 
Digest: sha256:58969069a70a7f9b29be83abd1465cf10c568049c9d183e9d7a7d8726d074048
Status: Downloaded newer image for fhsinchy/rmbyext:latest
Removing: PDF
c.pdf
hvt.pdf
e.pdf
a.pdf
off.pdf
```
The image fhsinchy/rmbyext is an executable one that delete file by extensions. We used bind mount to bind the zone/ directory in the contianer to our local filesystems. We created some .pdf files and deleted them with the executable container.
The option <code>-v (or --volume) $(pwd):zone/</code> a seen is used for creating a bind mount for a container, it can take three fields separated by (<code>:</code>). The generic systax is as follow :
```shell
--volume <local file system directory absolute path>:<container file system directory absolute path>:<read write access>
```
The third field is optional but you must pass the absolute path of your local directory and the absolute path of the directory inside the container.
The difference between a regular image and an executable one is that the entry-point for an executable image is set to a custom program instead of sh, in this case the rmbyext program. And as you've learned in the previous sub-section, anything you write after the image name in a container run command gets passed to the entry-point of the image.

So in the end the <code>docker container run --rm -v $(pwd):/zone fhsinchy/rmbyext pdf</code> command translates to <code>rmbyext pdf</code> inside the container. Executable images are not that common in the wild but can be very useful in certain cases.

<div align="right">

[Back to top](#) &#8593;

</div>
