### How to Containerize a Javascript Application
This section introduces to **volumes and multi-staged builds**, two of the most concepts in Docker.

### Writing the development Dockerfile
Let's plan on what and how our app should run by enumerating :
    - Get a good image by running JavaScript applications like **Node**
    - Set the default working directory inside the image
    - copy the <code>package.json</code> file into the image
    - Install necessary dependancy
    - Copy the rest of the project files
    - Start the <code>vite</code> development server by executing <code>npm run dev</code>  command

Our <code>Dockerfile.dev</code> having the above plan should then look like:
```
FROM node:lts-alpine

EXPOSE 3000

USER node

RUN mkdir -p /home/node/app 

WORKDIR /home/node/app  #set the default working directory

COPY ./package.json .
RUN npm install

COPY . .

CMD [ "npm", "run", "dev" ]
```
The <code>FORM</code> instruction sets the the official Node.js image as the base giving us all the goodness of Node .js necessary to run any JavaScript application.

The <code>lts-alpine</code> indicate that we want to use the Alpine variant, long term support version of the image.

The <code>USER</code> instruction sets the default user for the image to <code>node</code>. By default Docker run container as the root user. But according to Docker and Node.js best practice, this can pose a security threat. So it better to run a non-root user whenever possible. The node image comes with a non-root user named <code>node</code> whoch we can set as the default user using the <code>USER</code> instruction.

<code>WORKDIR</code> set the default directory to the specified with this instruction. By default the working image of any image is the root, but we don't want unecessary files sprayed all over our root directory. The working directory will be applicable to any subsequent <code>COPY</code>, <code>ADD</code>, <code>RUN</code> and <code>CMD</code> instructions.

The second <code>COPY</code> instruction copies the rest of the content from the current directory (<code>.</code>) of the host filesystem to the working directory (<code>.</code>) inside the image.

Finally, the <code>CMD</code> instruction here sets the default command for this image which is <code>npm run dev</code> written in <code>exec</code> form.

The <code>vite</code> development server by default runs on port <code>3000</code> , and adding an <code>EXPOSE</code> command seemed like a good idea, so there you go.

To build an image from our Dockerfile we'll use the command below:
```
docker image build --file Dockerfile.dev --tag hello-dock:dev .
```
Given the filename is not a <code>Dockerfile</code>, we have to explicitly pass the filename using the <code>--file</code> option. 
Let's execute the following commad to run a container using the iamge we just created.
```
docker container run --rm --detach --publish 3000:3000 --name hello-dock-dev hello-dock:dev 
```
What we did is okay but there is one big issue with it and a few places to be improved.

### Working with Bind Mounts in Docker

Normally, the development features in JS Framework usually come with a hot reload i.e the server reload and reflect each change made locally immediately.
Unfortunately, docker is not working that way because the server is running in a container and changes are made locally.

![alt ](/images/local-vs-container-file-system.svg)

TO solve that issue we can make use of bind mounts, using so, we can easily mount one of our local file system directory inside a container. Instead of making a whole other copy of the local syste, the bind mount can reference the local file system directly inside a container.

![alt ](images/bind-mounts.svg)

This way the hot reload can work as a charm and with any change made locally the server will immediately reflect them.
As seen in part I, bind mounts are achieved by using the <code>--volume</code> or <code>-v</code> option for the <code>container run</code> or <code>container start</code> commands.
```
--volume <local file system directory absolute path>:<container file system directory absolute path>:<read write access>
```
Let's apply bind mounts to our vite js app
```
docker container run --rm --publish 3000:3000 --name hello-dock-dev --volume $(pwd):/home/node/app hello-dock:dev
```
Although the usage of volume solve the issue of hot reloads, it introduces another problem. 
While the dependancies of a node project live inside the <code>nodes_modules</code> directory on the project root.

```
docker container run --rm --publish 3000:3000 --name hello-dock-dev --volume $(pwd):/home/node/app hello-dock:dev

> hello-dock@0.0.0 dev /home/node/app
> vite

sh: vite: not found
npm ERR! code ELIFECYCLE
npm ERR! syscall spawn
npm ERR! file sh
npm ERR! errno ENOENT
npm ERR! hello-dock@0.0.0 dev: `vite`
npm ERR! spawn ENOENT
npm ERR! 
npm ERR! Failed at the hello-dock@0.0.0 dev script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.
npm WARN Local package.json exists, but node_modules missing, did you mean to install?

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/node/.npm/_logs/2021-07-05T03_24_54_158Z-debug.log
```
After we mount the project root on our local file system as a volume inside the container, the content inside the container gets replaced along with the <code>node_modules</code> directory containing all the dependencies. This means that the vite package has gone missing.

### Working with Anonymous Volume in Docker
To solve the above problem we use **anonymous volume**. An anonymous volume is identical to a bind mount except that we don't need to specify the source directory.
The generic syntax is as follow
```
--volume <container file system directory absolute path>:<read write access>
```
The final command for starting the <code>hello-dock</code> container is as follow:
```
docker container run --rm --detach --publish 3000:3000 --name hello-dock-dev --volume $(pwd):/home/app/node --volume /home/node/app/node_modules hello-dock:dev
7546a5587d28a1d2b47bfe3ad53da10e24a1fcb9504dcc6740eb071626d5fe21
```
Here, Docker will take the entire <code>node_modules</code> directory from inside the container and tuck it away in some other directory managed by the Docker daemon on our host file system and will mount that directory as node_modules inside the container.

#Performing a Multi-Stages build in docker
The image we've built so far was in a development mode. To build one in a production mode, we gon have to face some challenges.

In production mode, the <code>npm run build</code> compile all the JS code to HTML/CSS a JS files. No other runtinme is needed to run those files but a server like <code>apache</code> or <code>nginx</code>

We may take the following step to create an image to be run in a production mode:
 * Using node as the base image to build the application
 * Install nginx inside the node image and use that to serve the static files.
One problem here is that the node image to too heavy and most of the stuff in there aren't needed for the static genereated files.
A best approach is :
 * Use node image as the base and build the application.
 * Copy the files created using the node image to an nginx image.
 * Create the final image based on nginx and discard all node related stuff.
This way the image contains only what needed and become very handy.
This approach is a <code>multi-staged build</code>.
To perform a such build, we create a Dockerfile as follow:
```
FROM node:lts-alpine as builder ## First stage of the build using node:lts-alpine as the base image, this stage is assigned with as builder

WORKDIR /app

COPY ./package.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:stable-alpine ## second stage of the build using nginx:stable-alpine as the base image

EXPOSE 80  ## nginx server runs at port 80 by default

COPY --from=builder /app/dist /usr/share/nginx/html 
```
* The last line is a <code>COPY</code> instruction. The <code>--from=builder</code> part indicates that you want to copy some files from the <code>builder stage</code>. After that it's a standard copy instruction where <code>/app/dist</code> is the source and <code>/usr/share/nginx/html</code> is the destination. The destination used here is the default site path for <code>NGINX</code> so any static file you put inside there will be automatically served.
Let's run our new container by executing the following command:
```
docker container run --rm --detach --name hello-dock-prod --publish 8080:80 hello-dock:prod 
```
Multi-staged builds can be very useful when building large applications with a lot dependancies. If configured properly, images built in multiple stages can be very optimized and compact.

## Ignore unnecessary files
With <code>.dockerignore</code> files and directories can be ignored and/or excluded from the image builds. The <code>.dockerignore</code> file contains a list of files and directories to be ignored. Docker uses the same concept as <code>.git</code>. Example of the content of such file:

```
*Dockerfile
.git
*docker-compse*
node_modules
```
The <code>.dockerignore</code> has to be in the build context. Files and directories mentionned here will be ignored by the <code>COPY</code> instruction. When using a bind mount, the <code>.dockerignorefile</code> won't have any effect.

### Network Manipulation in Docker
 Working with more than one container can be a little bit difficult, if we don't grasp the nuances of container isolation. 
 The question here is : How to connect two completely isolated containers to each other? 
 Some Approaches:
 * Accessing one container using an exposed port (NOT RECOMMANDED)
 * Accessing one container using its IP adress and defalut port (in case of server) [NOT RECOMANDED]

**Best Approach** : connect them by putting them under a user-defined bridge network.

#### Docker Network Basics
A network in docker is another logical object like a contianer or image. There is a plethora of commands under the <code>docker network</code> group for manipulating network.
```
docker network ls # List the networks in our system

# NETWORK ID     NAME      DRIVER    SCOPE
# c2e59f2b96bd   bridge    bridge    local
# 124dccee067f   host      host      local
# 506e3822bf1f   none      null      local
```
By default, Docker has five networking drivers. They are as follows:

* <code>bridge</code> - The default networking driver in Docker. This can be used when multiple containers are running in standard mode and need to communicate with each other.
* <code>host</code> - Removes the network isolation completely. Any container running under a host network is basically attached to the network of the host system.
* <code>none</code> - This driver disables networking for containers altogether. I haven't found any use-case for this yet.
* <code>overlay</code> - This is used for connecting multiple Docker daemons across computers and is out of the scope of this book.
* <code>macvlan</code> - Allows assignment of MAC addresses to containers, making them function like physical devices in a network.

As you can see, Docker comes with a default bridge network named bridge. Any container you run will be automatically attached to this bridge network:
```
docker container run --rm --detach --name hello-dock --publish 8080:80 fhsinchy/hello-dock
# a37f723dad3ae793ce40f97eb6bb236761baa92d72a2c27c24fc7fda0756657d

docker network inspect --format='{{range .Containers}}{{.Name}}{{end}}' bridge
# hello-dock
```
Containers attached to the default bridge network can communicate with each others using IP addresses which is **NOT RECOMMENDED AT ALL**

##### HOW ABOUT USER-DEFINED BRIDGE
A user-defined bridge, however, has some extra features over the default one. According to the official docs on this topic, some notable extra features are as follows:

* __User-defined bridges provide automatic DNS resolution between containers__: This means containers attached to the same network can communicate with each others using the container name. 
* __User-defined bridges provide better isolation__: All containers are attached to the default bridge network by default which can cause conflicts among them. Attaching containers to a user-defined bridge can ensure better isolation.

* __Containers can be attached and detached from user-defined networks on the fly__: during a container’s lifetime, you can connect or disconnect it from user-defined networks on the fly. To remove a container from the default bridge network, you need to stop the container and recreate it with different network options.

A network can be created using the network create command. The generic syntax for the command is as follows:
```
docker network create <network name>
## Example: command to create skynet network
docker network create skynet
228a8a20f776a975eb1da55c2c9669ae6920bdcd8718132e139b87c28d14e651
```
##### ATTACHING A CREATED NETWORK TO A CONTAINER
Two possible ways to attach a container to a network. 
* Using the network command:
  ```
  docker network connect <network identifier> <container identifier>
  ```
  We might the inspect command to check if the network is successfully connected to our container:

  ```
  docker network inspect --format='{{range .Containers}} {{.Name}} {{end}}' skynet
  hello-dock
  ###
  docker network inspect --format='{{range .Containers}} {{.Name}} {{end}}' bridge
  hello-dock
  ```
* Using the <code>--network</code> option for the <code>container run</code> or <code>container create</code>
  ```
  --network <network identifier>
  ```   
  Let's run another container and attach it to skynet and ping <code>hello-dock</code> from it to see if the two can communicate
  ```
  docker container run --network skynet --rm --name alpine-box -it alpine sh
  / # ping hello-dock 
  PING hello-dock (172.19.0.2): 56 data bytes
  64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.141 ms
  64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.255 ms
  64 bytes from 172.19.0.2: seq=2 ttl=64 time=0.234 ms
  --- hello-dock ping statistics ---
  3 packets transmitted, 3 packets received, 0% packet loss
   round-trip min/avg/max = 0.141/0.210/0.255 ms
  ```
  As you can see, running ping hello-dock from inside the alpine-box container works because both of the containers are under the same user-defined bridge network and automatic DNS resolution is working.

_**NOTE**: Keep in mind, though, that in order for the automatic DNS resolution to work you must assign custom names to the containers. Using the randomly generated name will not work._

##### DETACHING A CONTAINER TO A NETWORK
We use the <code>network disconnect</code> command for this task.
```
docker network disconnect <network identifier> <container identifier>
###
docker network disconnect skynet hello-dock

```
##### GETTING RID OF NETWORKS IN DOCKER
Network can be removed using the <code>network rm</code> command with the syntax:
```
docker network rm <network identifier>
###
docker network rm skynet
```
We may use the <code>network prune</code> command to remove any unused networks from our system; this command also has the <code>-f</code> or <code>--force</code> and <code>a</code> or <code>--all</code> options.

### Containerize a Multi-Container JavaScript Application

The JS project will consist of a <code>notes-api</code> powered by Express.js and PostgreSQL.

The project comprise of two containers in total that will be connected using a network. Concepts about environments variables and named volumes are also discussed in this.

##### RUNNING THE DATABASE POSTGRESQL
The database of the <code>notes-api</code> is a simple PostgreSQL that uses the official <u>postgres</u> image.

<blockquote>
  According to the official docs, in order to run a container with this image, you must provide the POSTGRES_PASSWORD environment variable. Apart from this one, I'll also provide a name for the default database using the POSTGRES_DB environment variable. PostgreSQL by default listens on port 5432, so you need to publish that as well.
</blockquote>

Let's run the DB server by executing the following:
```
docker network create notes-api-network # creating the network
#
docker container run --detach --name=notes-db --env POSTGRES_DB=notesdb --env POSTGRES_PASSWORD=secret --network=notes-api-network postgres:12
```
The --env option for the container run and container create commands can be used for providing environment variables to a container. 
Databases like PostgreSQL, MongoDB, and MySQL persist their data in a directory. PostgreSQL uses the /var/lib/postgresql/data directory inside the container to persist data.

Now what if the container gets destroyed for some reason? You'll lose all your data. To solve this problem, a named volume can be used.
This is where named volume come in hand.

##### WORKING WITH NAMED VOLUME
Similar to anonymous volume, except that you can refer to a named volume by using its name.

generic syntax:
```
docker create volume <volume name>
```
Let's create a volume named <code>notes-db-data</code>
```
docker create volume notes-db-data
notes-db-data
#
docker volume ls

DRIVER    VOLUME NAME
local     notes-db-data
```
Now let's mount this volume to the <code>/var/lib/postgresql/data</code> inside the <code>notes-db</code> container:

We've got to first stop and remove the old <code>notes-db</code> container.
```
docker stop container notes-db
docker rm container notes-db
```
Let's run a new container and assign the volume using the <code>--volume</code> option:
```
docker container run --detach --volume notes-db-data:/var/lib/postgresql/data --name notes-db --env POSTGRES_DB=notesdb --env POSTGRES_PASSWORD=secret --network=notes-api-network postgres:12
```
Let's do some inspection and see if the mounting was sucessful.
```
docker container inspect --format='{{range .Mounts}} {{ .Name }} {{end}}' notes-db
 notes-db-data 
```
Now the data will safely be stored inside the notes-db-data volume and can be reused in the future. A bind mount can also be used instead of a named volume here, but I prefer a named volume in such scenarios.

##### ACCESS LOGS FROM A CONTAINER
Command:
```
docker container logs <container identifier>
```








