## USING DOCKER-COMPOSE TO COMPOSE PROJECTS 
We saw how managing a multi-container project can be and even though we might simplify the process by using shell scripts, using <code>docker-compose</code> is the easiest when dealing with such project.

According to the offical documentation
<blockquote>
Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.
</blockquote>

_NOTE: Although Compose works in all environments, it's more focused on development and testing. Using Compose on a production environment is not recommended at all._

#### Docker Compose Basics
In the <code>api-notes/</code>, let's create a development Dockerfile with these lines:
```
# stage one
FROM node:lts-alpine as builder

# install dependencies for node-gyp
RUN apk add --no-cache python make g++

WORKDIR /app

COPY ./package.json .
RUN npm install ## we want the development dependancies also

# stage two
FROM node:lts-alpine

ENV NODE_ENV=development ## development not production

USER node
RUN mkdir -p /home/node/app
WORKDIR /home/node/app

COPY . .
COPY --from=builder /app/node_modules /home/node/app/node_modules

# nodemon is a tool that gives us the hot-reload deature 
CMD [ "./node_modules/.bin/nodemon", "--config", "nodemon.json", "bin/www" ]
```
This project has two containers:

* <code>notes-db</code> - A database server powered by PostgreSQL
* <code>notes-api</code> - A REST API powered by Exppress.js

In the world of compose, each container that makes up the application is known as a service. In composing a project, the first step is to define those services.

As docker daemon uses a <code>Dockerfile</code> for building images, Docker compose uses a <code>docker-compose.yaml</code> file to read services definition from.
Let's create our <code>docker-compose.yaml</code>

```
version: "3.8"

services: 
    db:
        image: postgres:12
        container_name: notes-db-dev
        volumes: 
            - notes-db-dev-data:/var/lib/postgresql/data
        environment:
            POSTGRES_DB: notesdb
            POSTGRES_PASSWORD: secret
    api:
        build:
            context: ./api
            dockerfile: Dockerfile.dev
        image: notes-api:dev
        container_name: notes-api-dev
        environment: 
            DB_HOST: db ## same as the database service name
            DB_DATABASE: notesdb
            DB_PASSWORD: secret
        volumes: 
            - /home/node/app/node_modules
            - ./api:/home/node/app
        ports: 
            - 3000:3000

volumes:
    notes-db-dev-data:
        name: notes-db-dev-data
```
Every valid docker-compose.yaml file starts by defining the file version. At the time of writing, 3.8 is the latest version. You can look up the latest version here.

Blocks in an YAML file are defined by indentation. I will go through each of the blocks and will explain what they do.

* The <code>services</code> block holds the definitions for each of the services or containers in the application. <code>db</code> and <code>api</code> are the two services that comprise this project.

* The <code>db</code> block defines a new service in the application and holds necessary information to start the container. Every service requires either a pre-built image or a <code>Dockerfile</code> to run a container. For the db service we're using the official PostgreSQL image.

* Unlike the <code>db</code> service, a pre-built image for the <code>api</code> service doesn't exist. So we'll use the <code>Dockerfile</code>.dev file.
  
* The <code>volumes</code> block defines any name volume needed by any of the services. At the time it only enlists <code>notes-db-dev-data</code> volume used by the <code>db</code> service.

Let's now have a closer look at the individual services:
* The definition of the <code>db</code> is as follow:
  
  ```
  db:
    image: postgres:12
    container_name: notes-db-dev
    volumes: 
        - db-data:/var/lib/postgresql/data
    environment:
        POSTGRES_DB: notesdb
        POSTGRES_PASSWORD: secret
  ```

* The <<code>image</code> key holds the image repository and tag used for this container. We're using the <code>postgres:12</code> image for running the database container.
  
* The <code>container_name</code> indicates the name of the container. By default containers are named following <code>\<project directory name>_\<service name></code> syntax. You can override that using <code>container_name</code>.
  
* The <code>volumes</code> array holds the volume mappings for the service and supports named volumes, anonymous volumes, and bind mounts. The syntax <code>\<source>:\<destination></code> is identical to what we've seen before.
* 
The <code>environment</code> map holds the values of the various environment variables needed for the service.

Definition code for the <code>api</code> is as follow:
 
 ```
 api:
    build:
        context: ./api
        dockerfile: Dockerfile.dev
    image: notes-api:dev
    container_name: notes-api-dev
    environment: 
        DB_HOST: db ## same as the database service name
        DB_DATABASE: notesdb
        DB_PASSWORD: secret
    volumes: 
        - /home/node/app/node_modules
        - ./api:/home/node/app
    ports: 
        - 3000:3000
 ```

* The <code>api</code> service doesn't come with a pre-built image. Instead it has a build configuration. Under the <code>build</code> block we define the context and the name of the Dockerfile for building an image. 

* The image key holds the name of the image to be built. If not assigned, the image will be named following the <code>\<project directory name>_\<service name></code> syntax.
  
* Inside the <code>environment</code> map, the <code>DB_HOST</code> variable demonstrates a feature of Compose. That is, we can refer to another service in the same application by using its name. So the <code>db</code> here, will be replaced by the IP address of the api service container. The <code>DB_DATABASE</code> and <code>DB_PASSWORD</code> variables have to match up with <code>POSTGRES_DB</code> and <code>POSTGRES_PASSWORD</code> respectively from the <code>db</code> service definition.
  
* In the <code>volumes</code> map, you can see an anonymous volume and a bind mount described. The syntax is identical to what you've seen in previous sections.

* The <code>ports</code> map defines any port mapping. The syntax, <code>\<host port>:\<container port></code> is identical to the <code>--publish</code> option you used before.

Finally, the code for the volumes is as follow:

```
volumes:
    db-data:
        name: notes-db-dev-data
```

Any named volume used in any of the services has to be defined here. If we don't define a name, the volume will be named following the <code>\<project directory name>_\<volume key></code> and the key here is <code>db-data</code>.

#### Starting services in docker-compose
There is a few ways in starting services defined in a YAML file. <code>up</code> is the first command we'll learn. This command builds any missing images, creates containers and start them in one go.
Every <code>docker-compose</code> command should be executed in the same folder as the <code>docker-compose.yaml</code>.

This is how we would run docker-compose for our project:

```
docker-compose --file docker-compose.yaml up --detach

## --file | -f is a must when the yaml file is not named docker-compose
```
The <code>start</code> command only start existing containers but doesn't create missing containers, same as <code>docker container start</code>

The <code>build</code> option for the <code>up</code> forces a rebuild of the images.

#### LISTING CONTAINERS IN DOCKER COMPOSE

Although service containers started by Compose can be listed using the container <code>ls</code> command, there is the <code>ps</code> command for listing containers defined in the YAML only.

```
docker-compose ps
    Name                   Command               State                Ports             
----------------------------------------------------------------------------------------
notes-api-dev   docker-entrypoint.sh ./nod ...   Up      0.0.0.0:3000->3000/tcp,:::3000-
                                                         >3000/tcp                      
notes-db-dev    docker-entrypoint.sh postgres    Up      5432/tcp  
```
#### EXECUTING A COMMAND INSIDE A RUNNIG SERVICE IN DOCKER COMPOSE

```
docker-compose exec <service name> commad 

## example

docker-compose exec api npm run db:migrate
> notes-api@ db:migrate /home/node/app
> knex migrate:latest

Using environment: development
Batch 1 run: 1 migrations
```
Unlike the container exec command, you don't need to pass the <code>-it</code> flag for interactive sessions. <code>docker-compose</code> does that automatically.

#### ACCESS LOGS FROM A RUNNING SERVICE IN DOCKER COMPOSE

We can also use the <code>logs</code> command to retrieve logs from a running service. The generic syntax for the command is as follows:

```
docker-compose logs <service-name>

#example

docker-compose logs api
Attaching to notes-api-dev
notes-api-dev | [nodemon] 2.0.12
notes-api-dev | [nodemon] reading config ./nodemon.json
notes-api-dev | [nodemon] to restart at any time, enter `rs`
notes-api-dev | [nodemon] or send SIGHUP to 1 to restart
notes-api-dev | [nodemon] ignoring: *.test.js
notes-api-dev | [nodemon] watching path(s): *.*
notes-api-dev | [nodemon] watching extensions: js,mjs,json
notes-api-dev | [nodemon] starting `node bin/www`
notes-api-dev | [nodemon] forking
notes-api-dev | [nodemon] child pid: 20
notes-api-dev | [nodemon] watching 18 files
notes-api-dev | app running -> http://127.0.0.1:3000
```
This is just a portion from the log output. You can kind of hook into the output stream of the service and get the logs in real-time by using the <code>-f</code> or <code>--follow</code> option. Any later log will show up instantly in the terminal as long as you don't exit by pressing <code>ctrl + c</code> or closing the window. The container will keep running even if you exit out of the log window.

#### STOPPING A SERVICE IN DOCKER COMPOSE
Two approaches to be taken, the first one is the <code>down</code>. The <code>down</code> command stops all running containers and removes them from the system. It also removes any network.
```
docker-compose down --volumes ## Removing all named volumes

Stopping notes-db-dev  ... done
Stopping notes-api-dev ... done
Removing notes-db-dev  ... done
Removing notes-api-dev ... done
Removing network notes-api_default
Removing volume notes-db-dev-data
``` 
The <code>stop</stop> command can also be used to stop all the containers for the application and keep them which can later be started by the <code>start</code> command.