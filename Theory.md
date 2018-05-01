# 1) Kernels:

Container: Isolated area of an OS with resource usage limits applied

Namespaces and Control Groups are Linux Kernel Primitives but now we have equivalence on Windows too

Namespaces: Namespaces are about isolation
Carve OS into multiple isolated virtual operated systems

In Hypervisor we carve out multiple VMs from a physical machine (RAM, CPU, etc.)
where each one gets a own slice of RAM, CPU, virtual nw, virtual storage etc.
In Virtual server or HyperV each VM or virtual server looks like a physical server/machine

In Containers, we take namespaces to take a single Operating System with all its resources that are high level constructs like root file systems, process trees and users
are carved into multiple Virtual Operating Systems called Containers. Each container gets it own virtual/containerized root file system, process trees, root users etc.
In Containers each container looks like a regular Operating System but in reality it isn't as all the containers share a single kernel on the host.
But all these containers are isolated and are unaware of each others.

Linux Namespaces are :
Process Id (pid)
Network (net)
Filesystem/mpunt (mnt)
UTS (uts)
User (user)
A docker container is an organized collection of these Namespaces.
So every container has its own pid, net, mnt, uts, user etc. within a secured boundary.

Control Groups(cgroups in linux and Job Objects in Windows) : grouping objects and setting limits
Group process and impose limitations on resource utilization
eg: 1 Container will have x amount of RAM, x amount of CPU etc.

Union File System: 
Layers - Union fs/mount $ COW (Also check LCOW)


# 2) Docker Engine 

Docker Engine (Linux) consists of these four major components:
 
a) Client - where we run commands like ```docker container run imageName```
b) Docker API = Daemon - implments REST API - implements orchestration, build, stacks, overlay nw etc.
c) containerD = container supervise that handles containers execution - lifecycle operations- eg: start, stop, pause, unpause
	implements things like basic nw, basic storage, basic image distribution etc.
d) OCI layer - it does the interfacing with the kernel.

So Client (a) asks the Daemon (b) for the container.
Daemon(b) gets containerD(c) to start, and manage container.
runC(d) at the OCI layer actually builds the container.
"runc" is the reference implementation of the OCI runtime spec

In Windows:
a)  Client
b) Daemon
c) Compute services layer (instead of containerD and OCI in linux)
Also implemented NTFS and Reg for layering system.
Client(docker.exe) and daemon(dockerd.exe)
We can run containers on windows via 2 methods:
1) Native windows container : ```docker container run ...```
	uses Namespaces and control groups to run isolated containers using host OS (kernel)
2) Hyper-V containers: ```docker container run --isolation=hyperv ...```
	In this case it spins up lightweight VM (even can have diff VMs with diff OSs) for each container 
When we run a command in Client:
Client calls a REST API in the Daemon (POST /vX.X/containers/create HTTP/1.1) 
Daemon calls containerD over GRPC API on a local unix socket eg: client.NewContainer(context, ...)
containerD starts a "shim" process for every container and "runc" creates a container and exists.
"runc" is called for every container and exits but shim still runs. 
This happends with every container and this is how containerD effectively manages multiple "runc" or shims.

Because a), b), c) and d) are all loosely coupled:
We can even restart daemon or containerD and it wont affect running containers. WOW!
We can upgrade docker to newer versions even in production without affecting running containers.
When it comes back up rediscovers running containers and shime and re-connects.


Images are read only templates for creating application container. It containes all code and supporting files(OS files and objects, app files, config, dependencies, manifest-json file) to run an application.
Images are bunch of layers stacked on top of each other.
Images are build time constructs.
Containers are runtime, container is a running image.

Containers should be ephemeral (short lived) and inmmutable i.e. if any modifications are required then we should not ```docker commit containerId```
but we should create a new version of image and then run container from the modified image.
##LOGS:

Docker Logs are generated at:
/var/log/messages in Linux
 - systemd: journalctl -u docker.service
~/AppData/Local/Docker

Can configure docker to use a logger like Syslog, Gelf, Splunk, Fluentd...
We can set default logging driver in daemon.json 
Override per container with
--log-driver -- log-opts

docker logs containerName

## SWARM
docker swarm init
//First Manager is the root CA of the swarm

//With the previous command we get 
a) root CA
b) mutual TLS
c) encypted store (cluster store)
d) certificate rotation
e) secure join tokens for managers and workers

//To set up a external CA
docker swarm init --external-ca ...
//to check swarm mode is inactive
docker system info
//To list all nodes in swarm
docker node ls 
//Add another manager to the swarm and use following command to get the command or token
docker swarm join-token manager
docker node ls
//we shouldn't go with even no so lets add one more and have 3 manager nodes in out swarm
docker swarm join --token mgtkn-....
docker node ls

//If one of the manager goes down then another node is selected as a manager using RAFT Consensus algo
// So we should use either 3, 5 or 7 no of managers i.e. odd no of managers in a swarm and not more than 7.

//To get token and command for workers to join in this swarm use:
docker swarm join-token worker

docker swarm join --token workertoken....

//To rotate the worker token, use:
docker swarm join-token --rotate worker

//To check the certificates use openssl command:
sudo openssl x509 -in /var/lib/docker/swarm/certificates/swarm-node.crt -text

//When a manager is restarted then to prevent it from automatically re-joining the swarm
//or preventing accidently restoring old copies of the swarm we can use _Swarm Autolock_

//Autolocking a new swarm :
docker swarm init --autolock

//Autolocking existing swarm :
docker swarm update --autolock=true
//we get the unlock key, to be kept safely

sudo service docker restart
docker node ls
docker swarm unlock 

//update docker swarm certificate expiry time
docker swarm update --cert-expiry 48h
docker system info

## Networking

###Bridge networking:
####Single-host nw
docker0
Linux: bridge driver
Windows: nat driver
Each bridge nw is isolated from each other, i.e. containers in one bridge cannot connect to other bridge's containers.
And that's why we require port mappings.
```docker network create -d bridge mybridgenw```
Encrypted nw:
```docker network create -o encrypted ```

###Overlay networking:
####Multi-host nw - container only
docker network create -o encrypted	
Spans multiple hosts with even on diff networks
```docker network create -d overlay myoverlaynw```
It is scoped to swarm and this means it is available on every node on the swarm having multiple nodes.

Overlay nw is container only and cannot communicate to other VMs or other physical servers on your newtwork.

###MACVLAN ("*transparent*" driver on Windows) :
It assigns every container its own IP address and MAC address on existing nw.
//but requires promiscuous mode, which most cloud providers doesn't allow

###IPVLAN:
Similar to MACVLAN but doesnt requires promiscuous mode

```docker node ls
docker network ls

docker network inspect bridge
//for windows 
docker network inspect nat```

```docker container run --rm -d alpine sleep 1d```
this will be in default bridge nw can check via docker network inspect bridge

```docker container run --rm -d --name web -p 8080:80 nginx ```
Lists out port mapping of the container
```docker port web ```

```docker container run --rm -d --network mybridgenw alpine sleep 1d ```

``` docker network ls ```
// myoverlaynw is listed with scope set to swarm
//Creating a native swarm service and assign overlay nw to it.
```docker service create -d --name pinger --replicas 2 --network myoverlaynw alpine sleep 1d ```
```docker service ls ```
//checking which nodes these services are running in
```docker service ps pinger ```
//switch to one of the nodes in the swarm
```docker inspect network myoverlaynw ```
//we get nw details along with pinger alpine container running on first node with it's IP x.x.x.x
//Switch to other(second) node where our pinger service is running and get the containerId
```docker container ls```
```docker container exec -it containerid sh ```
//In the shell ping the IP of container running in the other(first) node 
	``` ping x.x.x.x ```

Networking in Docker has built in 
Service Discovery: Every service on a swarm is discoverable by a built in DNS
Load Balancing: Every node in the swarm knows about every service, 
		we can also point external load balancers at any/every node in the swarm.

##Volumes 
Container are :
a) Non persistent - you run a container, when it's stopped everything goes away, no lasting data generated.
b) Immutable - once you run a container it shouldn't be changed. Build a new one and deploy again.

2 Types of storage:
a) Non Persistent - Storage driver/graph driver storage - by Union file system. 
   is at /var/lib/docker in linux and C:\ProgramData\Docker\windowsfilter in windows
   tied to the lifecycle of the container.
  Graph drivers are being replaced in containerD with snapshotters.
b) Persistent - Volume storage.

```
docker volume
docker volume create myvol
docker volume ls
docker volume inspect myvol
docker volume create secvol
ls -l /var/lib/docker/volumes/
docker volume rm myvol secvol
docker volume ls
```

```
docker container run -dit --name voltest --mount source=autovol,target=/vol alpinr:latest
//If no such volume exists then docker creates one automatically.
docker volume ls
ls -l /var/lib/docker/volumes/
//Any data we put in a volume goes in here: which will be empty right now.
ls -l /var/lib/docker/volumes/autovol/_data/
docker container exec -it voltest sh
ls -l /vol/
echo "test data" > /vol/testfile
cat /vol/newfile
exit
//we should be able to see it in the host as well:
ls- l /var/lib/docker/volumes/autovol/_data/
cat /var/lib/docker/volumes/autovol/_data/testfile
//Even if we remove container volume will still exist
docker container rm voltest -f
docker volume ls
//lets create new container with same volume mount
docker container run -dit --name newvoltest --mount source=autovol,target=/app nginx
//going into the container's sh
docker container exec -it newvoltest sh
ls -l /app
cat /app/testfile
echo "added data" >> /app/newfile
exit
//now we are back in host
cat /var/lib/docker/volumes/autovol/_data/testfile
//trying to remove the volume so we cant remove it unless it is being used by some container
docker volume rm autovol 
docker container rm newvoltest -f

ls -l /var/lib/docker/volumes/

## Secrets
anything when exposed to public can cause a security threat,
generally string blob of lenght <= 500K
Requires Swarm-mode.
```docker secret create sec1 ...
docker service create --name service1 --secret sec1 --replicas 2 ...
```
The secrets are encrypted and stored and shared only with worker nodes that have service replicas running.
In worker nodes it is decrypted at 
Linux: /run/secrets/   
In case of Linux it is stored in memory file system and wont be persisted.
Windows: C:/\ProgramData\Docker\Secrets
In case of Windows there are no in memory file systems and it is stored physically.
So we can mount docker root directory using tools like bitlocker etc.
When services are removed 
``` docker service rm service1 ```
the worker nodes are instructed to remove secrets from memory.
It doesn't works on standalone containers even if container is in nodes on swarm mode.
It is available only in services in Swarm-mode.
Example:
```
docker secret create sec-v1 .\secretfile
docker secret ls
docker secret inspect sec-v1 

docker service create -d --name secured-service `
--secret sec-v1 `
microsoft/powershell:windowsservercore `
PowerShell Start-Sleep -s 432000

docker service ls
docker service inspect secured-service

//Go to the node in which service is running or if single node then you are in there itself :P
docker container ls
docker container exec -it containerid PowerShell
ls C:\ProgramData\Docker\secrets\
cat C:\ProgramData\Docker\secrets\sec-v1

docker secret rm sec-v1
//We cannot delete a secret which is being used hence error 
docker service rm secured-service
docker secret rm sec-v1
docker secret ls

docker stack deploy -c yamlfile.yml voter
docker stack ls
docker service ps voter
docker stack services voter
docker service inspect voter_vote --pretty
docker service ps voter_vote
