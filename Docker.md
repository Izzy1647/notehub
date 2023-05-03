
![[Screenshot 2023-04-08 at 21.36.22.png]]

What happens after running `docker run hello-world`:

![[Screenshot 2023-04-08 at 21.38.10.png]]

- `docker run hello-world` means we want to start up a new container, using the image with the name of hello-world
- Local Docker Client executes the command, talks to local **Docker Server(daemon)**
- Docker server checks if there is a copy of the image in local computer, by looking into **image cache**
- There is no local copy, so docker server reaches out to **docker hub**, which is repository of free public images
- Docker client pulls hello-world image from docker hub, saves it into local image cache
- Docker server creates a container out of the image, runs the executable in the container
- Docer client streams the output back to Docker client

## What is a container??
### A quick overview: operating system
![[Screenshot 2023-04-08 at 22.52.46.png]]

- A process or a set of processes that have a grouping of resources specifically
![[Screenshot 2023-04-08 at 23.00.48.png]]

## Commands
`docker run <image name> <?command>`
- if provided, **command** will override the default command in the image

`docker ps`
- Lists running containers
- `--all` shows all history

`docker logs <containerId>`
- Look at the logs of a previous container
- Not re-running the container

`docker stop <containerId>`
- Issues a `SIGTERM` to the container
- Programs inside the container may do some clean-up after receiving the signal
- Try for 10 sec before falling back to `docker kill`

`docker kill <containerId>`
- Issues a `SIGKILL` to the container
- Shut it down immediately

## Container lifecycle

- docker run = docker create + docker start
![[Screenshot 2023-04-15 at 21.35.06.png]]

- `docker create` is about initializing a File System snapshot
- `docker start` is about executing the startup command

```shell
➜  ~ docker ps --all           

CONTAINER ID   IMAGE     COMMAND     CREATED         STATUS                     PORTS     NAMES

04cea1cb9365   busybox   "echo Hi"   8 seconds ago   Exited (0) 7 seconds ago             exciting_sammet

➜  ~ docker start -a 04cea1cb9365            

Hi

#######################

➜  ~ docker create hello-world   

a067b6cbe40624b3a8714176927ccaa04722b498ada382aca25c88cffd96c2a5

➜  ~ docker ps --all          

CONTAINER ID   IMAGE         COMMAND     CREATED              STATUS                      PORTS     NAMES

a067b6cbe406   hello-world   "/hello"    7 seconds ago        Created                               affectionate_lewin

04cea1cb9365   busybox       "echo Hi"   About a minute ago   Exited (0) 45 seconds ago             exciting_sammet

➜  ~ docker start -a a067b6cbe406            

Hello from Docker!

This message shows that your installation appears to be working correctly.

...
```


## Execute commands in running containers

`docker exec -it <container id> <command>`

![[Screenshot 2023-04-16 at 23.10.32.png]]
```shell
➜  ~ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS      NAMES
f1c62b0c0fed   redis     "docker-entrypoint.s…"   19 minutes ago   Up 19 minutes   6379/tcp   competent_solomon
➜  ~ docker exec -it f1c62b0c0fed redis-cli
127.0.0.1:6379>
```

![[Screenshot 2023-04-17 at 19.29.44.png]]
- `-it` is equivalent to `-i -t`, `-i` connects user input to STDIN, `-t` makes the output formats pretty

## Get terminal access inside the container
`docker exec -it <containerId> sh`
which is basically a variation of: 
`docker exec -it <containerId> <command>`
as `sh` is just a command to invoke a command processor.
And this is very commonly used.


## Creating my own docker image
![[Screenshot 2023-04-18 at 21.51.41.png]]
**Steps:**
- Specify a base image
- Run some commands to install additional programs
- Specify a program to run on container startup

### Example
**Goal**: Create an image that runs redis-server

The first Dockerfile:

```dockerfile
# Use an existing docker image as a base
FROM alpine

# Download and install a dependancy
RUN apk add --update redis
# apk: the package manage tool preinstalled in alpine

# Tell the image what to do when it starts as a container
CMD ["redis-server"]
```

Run `docker build .` to build a image in the base directory.

**Workflow behind docker build:**

1. `FROM alpine`: pulls an `alpine` image
2. `RUN apk add --update redis`:
- creates a temporary container out of the image pulled from last step;
- executes the command inside the temporary container; 
- takes the file system of the temporary container and saves it as a fs snapshot of the image;
- removes the temporary container once the job above is done.
3. `CMD ["redis-server"]`: Sets the primary cmd of the built image as `redis-server`

![[Screenshot 2023-05-02 at 18.57.25.png]]

## Tag a docker image
```shell
docker build -t ${docker_id}/${image_name}:${version} ${context}
# example:
docker build -t izzy1647/redis-server:0.0.1 .
docker build -t izzy1647/redis-server:latest .
```



## Project: a web application in Docker
1. Create a simple node app
`package.json`:
```json
{
	"dependencies": {
		"express": "*"
	},
    "scripts": {
	    "start": "node index.js"
	}
}
```

`index.js`:
```javascript
const express = require("express");

const app = express();

app.get("/", (req, res) => {
	res.send("hi there");
})

app.listen(8080, () => {
	console.log("Listening on port 8080");
});
```

2. Build a docker image out of it
`Dockerfile`:
```dockerfile
# Specify a base image
FROM node:alpine

WORKDIR /usr/app

COPY ./ /usr/app

RUN npm install

# Set up a default command
CMD [ "npm","start" ]
```

- `WORKDIR` command sets a work directory in the container
- `COPY` copies files from local working directory into the container

In terminal run `docker build -t izzy1647/simpleweb .`

3. Init a container out of the image and start it
```shell
docker run -p 8080:8080 izzy1647/simpleweb
```

An important thing: PORT MAPPING. Use **-p** flag to do it

![[Screenshot 2023-05-03 at 17.12.51.png]]

