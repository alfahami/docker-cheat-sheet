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
