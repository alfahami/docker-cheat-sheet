# Hands on Docker 
An overview of docker and its functionalities.\
A cheat sheet of its common used commands\
A yet minimal, simple to be complex guide on docker.
Based on [The Docker Handbook](https://www.freecodecamp.org/news/the-docker-handbook/#introduction-to-containerization-and-docker "FreeCodeCamp")


## The idea behind containerization

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
  
# Docker achitecture
   * [Container](#container) 
   * [Images](#images)
   * [Registry](#registry)
## What is a Container <a name="container"></a>
The offical Docker <u>[resources](https://www.docker.com/resources/what-container)</u> state:
> A container is an abstraction at the application layer that packages code and dependencies together. Instead of virtualizing the entire physical machine, containers virtualize the host operating system only.

Containers are completely isolated environments from the host system as well as from each other. They're a lot lighter than the traditional VM and a large number of containers can be run simultaneously without affecting the performance of the host system. Containers are considered to be the next generation of virtual machines.*

##### A little comparison between VM and Containers
The main difference is the method of virtualization.


| Traditional VM| Containers |
| -----------| ----------- |
| Created, managed by a hypervisor (OracleVM, VirtualBox,..)     |Utilize the host OS via the container runtime while maintaining isolation    |
| The hypervisor sits between the host OS and the VM to act as a medium communication   | The container runtime (Docker) sits between the container and the host OS      |
| Comes with its own guest OS which is heavy as the host OS   | Doesn't need its own OS as it follows the idea behind containeurization       |
| The app inside the VM communicates with the guest OS which talks with the hypervisor  which talks to the host OS ...(long chain of communication)| The container communicate with the container runtime which communicates with the host OS to get necessary resources.       |
| With it own OS, VM are heavier and more resources consuming | By eliminating the entire host OS, containers are much lighter and less ressource-hogging     |

##### Illustrations

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
   ![alt full process](images/docker-run-hello-world.svg)

   PS : if the the image is not found in the local repository <code>Unable to find image 'image-name'</code> will be printed on the terminal.
   The <mark>Daemon</mark> will reach out to default public registry (Docker Hub) and pulls in the last copy of the image if found.
   A new container will then be created from the fresly pulled image.
   By default, Docker Daemon look for images in the hub that are not present locally and once an image has been fetched, it will stay in the local cache.
   In case of newer images, the Daemon will fetch them automatically.

   ### Basic manipulations 
   * <code>docker \<object> \<command> \<options> </code> command syntax\
             - \<object> : type of docker object to be manipulated\
             - \<command> task to be carried by the <mark>Daemon</mark>\
             - \<options> : any valid parameter that can override the default behaviour of the command.
  * <code>docker run \<image name> </code> or <code>docker container run \<image name></code>: run a container\
  * the \<option> <code>--publish or -p</code> is for port mapping\
  * <code>Ctrl +C </code> : stop the container
  

  ### Publish a port
  As isolated environments as containers, the OS doesn't know nothing about what's going on inside them. Hence applications running inside them reamin inaccessible from the outside.

  To allow access from the outside, We must publish the appropriate port inside the container to a port on your local network. The common sysntax is <code>--publish or -p</code>
  <code>--publish \<host port> \<container port></code>
  <code>docker run container --publish 8080:80 /repo/docker-image</code> : publish the image /repo/docker-image to the port 8080 locally.

### Running a container in the background
Known as the **detach mode**, this mode keeps running the container int he background. The option to be used is <code>--detach or -d</code>
Note that this command output the full _CONTAINER ID_ of the newely created container on the terminal
<code>docker run container --detach --publish 808:80 /repo/docker-image</code>\
output: 9e634a231ea4e37f94ef747a8ca6bd2bc25c473699afc783d5dce836bc341090

PS: The order of options doesn't really matter but anything that is been put after the image name will be passed as an argument to the container entry-point. 


   
           









            
 






