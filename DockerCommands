//Install Docker on Linux
wget -qO- https://get.docker.com/ | sh

//Update Docker in Windows
Install-Module DockerProvider -Force
Install-Package Docker -ProviderName DockerProvider -Force

//sudo service docker restart

// run a container,if image not present in cache then it gets "docker pull" automatically
docker run -it ubuntu bash
apt-get update
apt-get install curl
exit

docker ps -a
// made some changes to a container? commit it to create a new image. We should create an image but this is just to check out "commit"
docker commit containerId
docker images
docker tag imageId myImageName
docker images

// running containers
docker ps

// all containers
docker ps -a

//with Id
docker ps -aq

//last run container
docker ps -l

//remove container 
docker rm containerId

// images
docker images

//remove images 
docker rmi imageId
// to force delete
docker rmi -f imageId


//Create a Dockerfile
touch Dockerfile
vim Dockerfile
 FROM ubuntu
 RUN apt-get update
 RUN apt-get install -y curl

//save and exit and build image from this dockerfile
docker build -t newimagename .

//check new image
docker images

docker run -it myubuntuimage bash

//execute a command on running a container
 FROM ubuntu
 RUN apt-get update
 RUN apt-get install -y curl
 CMD curl -i www.google.com
 
 docker build -t mycurlimage .
 
 //edit dockerfile to use entrypoints so as to pass parameters while docker "run" the container
 FROM ubuntu
 RUN apt-get update
 RUN apt-get install -y curl
 ENTRYPOINT ["curl", "-i"]
 
 docker build -t mycurlimage .
 
 docker run mycurlimage www.google.com
 docker run mycurlimage https://reqres.in/api/users?page=1
 
 //edit dockerfile to use entrypoints
 FROM ubuntu
 RUN apt-get update
 RUN apt-get install -y curl
 ENTRYPOINT ["curl", "-i"]
 CMD https://reqres.in/api/users?page=2
 //if no value is passed in docker run then the CMD line's page 2 api will be called.
 
 docker build -t mycurlimage .
 
 docker run mycurlimage
 //outputs response of https://reqres.in/api/users?page=2
 
 docker run mycurlimage https://reqres.in/api/users?page=1
 //outputs response of https://reqres.in/api/users?page=1
 docker run mycurlimage https://reqres.in/api/users?page=3
 //outputs response of https://reqres.in/api/users?page=3
 
 //Override entry point; here replacing curl with bash
 docker run -it --entrypoint bash mycurlimage
 
 // -d to run it in backgorund and --publish-all or -P exposes it to others on random ports
 docker run -d --publish-all imagenameofwebapporapi
 
 //or expose localhost port 80 to containers port 8080 
 docker run -d -p 80:8080 imagenameofwebapporapi
 
 //Get IP address of the container
 docker inspect --format '{{ .NetworkSettings.IPAddress}}' containerid
 returnedIp
 ping returnedIp
 
 
 // Create api image: docker build -t username/imagename:tag .
 
 $ docker swarm init
Swarm initialized: current node (jfzjw0wo8p9rax7ofcy5XXXXX) is now a manager.

// To add a worker to this swarm, run the following command:
//    docker swarm join --token XXXXXXXXXXx-1-e2qregpnucqkzk6u87tgc0j08dnjp0zambwkg26p3ce8aovk-e8hb2xge 192.168.XX.X:PORTNO

// To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

docker stack deploy -c docker-compose.yml webapi

docker service ls

// outputs webapi_web i.e. webapi appended with the service name in the docker-compose.yml file

// A single container running in a service is called a task. Tasks are given unique IDs that numerically increment, 
// up to the number of replicas you defined in docker-compose.yml. List the tasks for your service:
```
docker service ps webapi_web
```

docker container ls -q

curl -5 http://localhost:8080/test

// We can also change number of replicas in our docker-compose.yml and re run: 
docker stack deploy -c docker-compose.yml <appname>

// this doesnt require us to tear down a swarm or stop and rerun containers
docker services ls -q

//Taking down the app and the swarm
 - Take the app down with docker swarm rm:
 docker stack rm cardsapi
 - Take down the swarm.
 docker swarm leave --force

// Summary
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager

//Create a Nw switch from HyperV Manager with eg: name myswitchname
//Creating VMs
docker-machine create -d hyperv --hyperv-virtual-switch "myswitchName" apovm1
docker-machine create -d hyperv --hyperv-virtual-switch "myswitchName" apovm2

docker-machine ssh apovm1 "docker swarm init --advertise-addr <ipadd:port>"

docker-machine env apovm1
// configure your shell to talk to myvm1.
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env apovm1 | Invoke-Expression

docker stack deploy -c docker-compose.yml apistack

docker stack rm apistack
docker-machine ssh apovm2 "docker swarm leave"
docker-machine ssh apovm1 "docker swarm leave --force"

// unset the docker-machine environment variables in your current shell with 
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env apovm1 -u | Invoke-Expression
