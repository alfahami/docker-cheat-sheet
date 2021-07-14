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
