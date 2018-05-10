# Docker
---
 - Dockerfile
  - How to create a docker file?
 - How to build a docker image?
 - How to upload an image to Docker Hub?
---
 - Docker Compose
 - Environment variables
 - Port Mapping
 - Links
 - Volumes
---
 - Network
 - Docker Swarm
  - Services
  - Stacks
  - Tasks
---
### Docker - theory
---
##### Difference between Virtual Machines and Docker
---
##### - Docker Container and Images
Container: Isolated area of an OS with resource usage limits applied

With Docker, you can just grab a portable node or dotnetcore runtime as an image, no installation necessary. Then, your build can include the base node or any other runtime image right alongside your app code, ensuring that your app, its dependencies, and the runtime, all travel together.
---
To use a programming metaphor, if an image is a class, then a container is an instance of a class—a runtime object. Containers are hopefully why you're using Docker; they're lightweight and portable encapsulations of an environment in which to run applications.
[Source](https://stackoverflow.com/questions/23735149/what-is-the-difference-between-a-docker-image-and-a-container)

These portable images are defined by something called a Dockerfile.
---
#### Networking
Create a network and run containers in those network using "--network"
```
docker network create testnw
docker create --name my-nginx --network testnw -p 8080:80 nginx:latest
docker network connect testnw my-nginx
```
---
### Swarm

A swarm is a group of machines that are running Docker and joined into a cluster. After that has happened, you continue to run the Docker commands you’re used to, but now they are executed on a cluster by a swarm manager. The machines in a swarm can be physical or virtual. After joining a swarm, they are referred to as nodes.
---


#### Services
In a distributed application, different pieces of the app are called “services.” For example, if you imagine a video sharing site, it probably includes a service for storing application data in a database, a service for video transcoding in the background after a user uploads something, a service for the front-end, and so on.
contd...

---
contd...
Services are really just “containers in production.” A service only runs one image, but it codifies the way that image runs—what ports it should use, how many replicas of the container should run so the service has the capacity it needs, and so on. Scaling a service changes the number of container instances running that piece of software, assigning more computing resources to the service in the process.
Luckily it’s very easy to define, run, and scale services with the Docker platform -- just write a docker-compose.yml file.
[Source](https://docs.docker.com/get-started/part3/#run-your-new-load-balanced-app)

---

To sum up: Services codify a container’s behavior in a Compose file, and this file can be used to scale, limit, and redeploy our app. Changes to the service can be applied in place, as it runs, using the same command that launched the service: docker stack deploy

---

#### Stack

Run your new load-balanced app
Before we can use the docker stack deploy command we first run:
```
docker swarm init
```

Now let’s run it. You need to give your app a name. Here, it is set to getstartedlab:
```
docker stack deploy -c docker-compose.yml getstartedlab
```

---

Our single service stack is running 5 container instances of our deployed image on one host. Let’s investigate.

Get the service ID for the one service in our application:
```
docker service ls
```

[Source:Docker](https://docs.docker.com/get-started/part3/#run-your-new-load-balanced-app)

# THANK YOU
