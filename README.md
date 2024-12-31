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
