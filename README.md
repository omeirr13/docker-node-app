# docker-node-app

- we should have a "Dockerfile" in our app.
- normally if we wanted to run application in our computer, we would have to type npm i in the project directory, that would install product dependencies inside node_modules folder.
- then we would type node app.js to run the app, that would start the application using the node version that i have installed on the computer, but we dont want to do that, we want to run our application inside an isolated container with its own version of node running inside it...to do that we need to make an image...that image should have an initial parent image layer, to say what version of node and linux distribution we want container to be run, after that the after layers will copy the source codes and all the dependencies into it..and some extra information if we want.

 ## What is a DockerFile
 - its like a set of instructions, which tells docker how to create a specific image with all of its different layers, and each instruction will be on a different layer in our final image

the first instruction will be parent image, in this case the node image.
```
FROM node:17-alphine
```
- get version 17 of node, and use the alpine distribution of linux...it will download from docker hub

- tell docker when we run commands on the image in the future after this instruction, do it from this working directory...so RUN npm i will do in here
```
WORKDIR /app
```

```
COPY . .
```
- COPY: means copy some files to the image.
- first dot is a relative path to the directory i want to copy my source files from, since our source files are in same root directory as docker file then path is going to be a single dot, if all our source code was in src folder, then it would be COPY ./src .
- the second dot is the path inside the image that i want to copy my source code to, remember images have their own folder structure...right now we are saying copy in the root directory of the image we're creating, we dont really copy into the root directory because it might clash with other files or folders inside the root directory, so instead we copy to a path like /app
```
COPY . /app
```
- after we added WORKDIR /app here, the path is now relative to /app so it will copy to /app/app, so change it to . again
```
COPY . .
```

- next layer is all dependencies we need to install on the image
- to install dependencies into a project we do npm i, we do a similar thing to the image, we specify commands that we want to run in the image, as the image gets built.
- to specify a command we use RUN instruction
```
RUN npm install
```
- so this instruction tells docker to run a command on the image itself, as this layer is being added on while image is being buil
- **but on slight problem:**
- the command is going to run inside root directory of the image, but our package.json got copied in the app folder...so it will throw error.
- for this to work we need to run it in same folder as package.json
- so the way we do that is specify a working directory for the image.


```
RUN node app.js
```

- we might think we have to add this now, but we dont do it, because when we add RUN instruction it runs the command as image is being built, at build time. the image is not a running applcation its just a blue print for a container, a container is a running instance for image

```
CMD ["node", "app.js"]
```

- CMD allows us to specify any commands that should be run at runtime, when container begins to run, we write command as array of strings in double quotes.

# Expose Instruction
- tells docker which port the container should expose, because in the app.js file we listen for requests on port 4000, now this app will run inside the container, so port will be owned by the container, to make requests to this API we need to send them to the container using this port number
```
EXPOSE 4000
```
- this is kind of optional, we only need it here if we're gonna be running images in spinning up containers using the docker desktop application, because docker desktop will use this information in the docker file to setup something called port mapping, if we are running containers from the command line then its not really needed.


### Final step to build image
- we just have to run a single docker command in the same directory where DockerFile exists
```
docker build -t myapp .
```
- flag -t allows us to give image name
- the . is the relative path to the docker file from the directory we are currently in.

- the image is created now, and its stored in a special docker folder in our computer, but in docker desktop we can see that image with the name we gave it.

# Dockerignore
- sometimes we dont want some files to ```COPY``` to the image, like some nodemodules
- now we wont want nodemodules, because we already installed dependencies on the image in ```RUN npm i```, it will replace it...maybe the old one has outdated stuff
- it takes more time.

make a ```.dockerignore``` inside this file specify any files or folders we want docker to ignore.

# Starting and stopping containers
- in docker desktop we can see the images, we can run that image to create a container which then runs our application
- **Container Name**
- **Ports: ** we can specify port by which we can reach this container, in our application we had a node server which was listening to requests on port 3001, but that is a port that will be exposed by the container, its not a port exposed by our computer directly that we can access via localhost
- but docker allows us to map a localhost port number to the port exposed by the container
- 5000->4000/tcp
- typically we keep them same, but they can be different also.
- **this port mapping option on docker desktop will only be there if we added the expose instruction in docker file, else wont be here**


  ### List down all images we have
  ```docker images```

 ### List down all running containers
 ```docker ps```
 ### List down all containers
 ```docker ps -a```

 ### How to stop container
 ```docker stop <container_id/container_name>```
 
 ### How to start existing container
 ```docker start <container_name/container_id>```
- this time we dont have to reconfigure the port mapping or anything like that, because it was already created for the container when we ran the image.
  ### How to run image
  ```docker run --name <container_name> <name_image/id_image```
 
- container_name: is the container name we will make
- after running like this, if we try to run in browser it wont work, because port listening for request is exposed by container and not our computer, so we cant directly access it, we need to map localhost port to container port
- we did it docker desktop, we can also do in command line
```docker run --name <container_name> -p 4000:4000 <image_id/image_name>```
- -p: publish containers port to the host computer / map container port to a port on host computer
- port that we want to map container port on our host computer : port exposed by container

 ### What if we want to run our container without blocking terminal?
 - we can add detached flag
 - if we run in detached mode, our terminal is detached from the process
```docker run --name <container_name> -p 4000:4000 -d  <image_id/image_name>```

### Layer Caching
- Every line we write inside docker file kind of represents a new layer in the image we are creating
- each line adds something new into the image

1) ```FROM node17:alpine``` => pulling the node image
2) ```WORKDIR /app```specify the working directory
3) ```COPY . .``` => copy the source files to the image
4) ```RUN npm install```=> install dependencies to the image

- each time it adds something to the image, it takes some time to do, like it takes some time to install dependencies, takes a few seconds to download the parent image..
- if we change the parent image to a different version of node, that we havent downloaded in the past ```node16-alpine```, we'll see the whole process of building the image again.

**How to build a docker image from Dockerfile:**
```docker build -t myapp .```


**What if we change change source code?**
- if we change source code we will have to create the image again, as old source files were copied before.
- if we run ```docker build -t myapp3 .``` again
- it will take a lot less time to build the image, this is because docker **caches image layers**.
- everytime docker builds an image, it stores that image at each layer, in the cache
- when we build an image again, before docker starts the whole build process from scratch it looks in our cache and it tries to find an image in the cache that it can use for the new image we are creating, so workload is reduced..in our case we made change to app.js that effected the COPY layer, and therefore also everything after the COPY layer, but it doesnt effect the layers before.

- single layer is not caches but upto that point the stack of layers is cached, like up until the ```RUN npm install``` not just that layer is cached, but whole stack of layers up until that point is cached.

### Why cant we add the ```RUN npm install``` layer before ```COPY . .``` layer? so that is cached too?
- we cant do that because up until that point package.json will not be copied..it needs that to npm i
- **solution:** we could copy package.json as a separate layer first before we run ```RUN npm install```
```
FROM node16:alphine
WORKDIR /app
COPY package.json .
RUN nom install
COPY . .
EXPOSE 4000
CMD ["node", "app.js"]
```


## How to delete an image?
```
docker image rm myapp
```
- it could display an error saying: unable to remove repository reference myapp
- thats because its being used by some container
- if we open Docker Desktop, we see all containers even if none are running however they are using that image..if we were to run them it would use that image.
- if we go in images, we can see all the images that are currently **IN USE**
  
- **What if we still want to delete that image even if it is in use?**
- **First way:**
  ```docker images rm myapp -f```
- This will leave the containers in a dangling/orphan state
- existing created containers of that image will keep running, but we cant create new containers now of that image.
- **Second way:**
- delete the container first
### How to delete container?
- ```docker container rm myapp_c2```
- we have to make sure we first delete all containers, using that image, and then delete that image.

# How to version our images?
- we could have pulled various version of images
- we pulled node17:alpine
- which means use node version 17 on top of alpine linux distribution.
- we could have alaos used a different version along with a different linux distribution.
- all of these are just variation/versions of the same node image.
- the way those versions are denoted are by tags.
- we create a tag by adding a colon(:) after the image name.
- tag names can be anything we want alphanumeric, and we can use them to create multiple versions of node, with slight variations.

# Managing images
### How to delete all images, and containers to start from scratch
```docker system prune -a```
- this will remove all containers, all images, and all volumes.

### How to create images with tags
```docker build -t myapp:v1 .```
- this create an image with that tag name

### Things to remember:
- images have tags
- containers have names
