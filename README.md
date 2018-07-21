faqs relating to udemy docker course: https://www.udemy.com/docker-mastery/learn/v4/t/lecture/10091380?start=0

### Docker Images and Containers

- `docker container run --publish 80:80 nginx`. 
This command can also be executed as docker run and then all the options and the container name. However the new way of doing it would be to use docker management commands and therefore to use docker container run. The `--publish` option publishes the container port 80 on the host port 80. The port on the left side of the colon is the host port and the port on the right side is the container port.

nginx is the name of the container. If the container is not found in the machine, it will be pulled from docker hub.

- `docker container run --publish 80:80 --detach nginx`.
`--detach` tells docker to run the container in detached mode or in the background. We get back the container id.

- `docker container ls` lists the containers that are running.

- `docker container stop <container id>` stops the container with the container id. For the container id, simply the first few numbers of the container id will do simply to make the number unique.

- `docker container ls -a` lists the history of all containers that were started. 

- `docker container run --publish 80:80 --detach --name webhost nginx` This will start the nginx container but it will start the container with the name webhost.

- `docker container logs <container name>` This will show the logs of the container with the container name.

- `docker container top <container name>` This will show all the running processes inside a container  

- `docker container rm <container id1> <container id2> <container id3> ...` This will remove the containers having the listed ids. Please note that you cannot remove a running container. In case of the container ids as well we need to only specify only the first few digits of the container id just so that it is unique. There is a `-f` option for force to remove even running containers. 

- `docker container start <name of the container>` This will start a container that was stopped previously

- `docker container inspect <name of the container>` returns a json array of all the configurations that was used in order to start the container.

- `docker container stats` gives real time data of the containers' cpu usage, ram usage and the like

#### Getting inside a docker container

- `docker container run -it --name <container name> nginx <command>` Runs a docker container and attaches a pseudo terminal to it. Note that when a container is started it automatically runs a command or a set of commands based on how it was configured. However if we want to run our own command at the start of the container we can do so by writing the command at the end of the run command. 

For instance the following command starts an nginx container and starts a bash shell inside the terminal `docker container run -it --name nginx nginx bash`

- `docker container start -ai <name of the container>` starts an existing container and attaches a pseudo terminal in it. 

### Docker Networks

When we start a container what we are actually doing is in the background connecting to a particular docker network called the `bridge` network. For containers to talk to each other we are not actually required to expose their ports to the physical network. For instance if we were using a mongodb container and a node container, we would not have to expose their ports for them to talk to each other. The best practice is generally to create a virtual network for each app that we are running. 

Along the way its easy to notice that most parts of docker use many default configurations that work right out of the box, but they are easily configurable. Hence the phrase `Batteries included, But Removable`. 

Its possible to attach containers to more than one virtual network and it is also possible to skip virtual networks and simply use the host IP using `--net=host`   

#### IP Address
 
The ip address of a container when we start it is not the same as the ip address of the host we are in. In order to inspect the ip address of the container we can use the `inspect` command.

For instance `docker container inspect --format '{{ .NetworkSettings.IPAddress }}' <name of the container>`. The `--format` option comes with the `inspect` command and it is used to filter the json response that we get from the `inspect` command. The `--format` option takes a `go-template` as the template for filtering the json response. 

When we start docker service, a default docker network is always started called the `docker0` or `bridge` network. This is essentially a virtual network and any container by default becomes a part of this network unless otherwise specified. All docker virtual networks are protected from the outer internet by a NAT firewall. When we publish ports on a docker container, the corresponding host machine port is opened in the docker firewall and all traffic from there is routed to the port inside the container. 

Now any container that is using the same virtual network will be able to communicate without them having to publish their ports. Whenever we are publishing the port, it means we are actually opening up traffic from the NAT firewall to the virtual networks.

- `docker network ls`: for listing all networks
- `docker network inspect <network>`: for inspecting a particular network
- `docker network create --driver`: The driver flag is optional and can be used to create a network with a built in or 3rd party driver.
- `docker network connect`: attach a network to a container
- `docker network disconnect`: detach a network from a container

The `host` network is a special network that skips any virtual networks and attaches the container directly to the host's physical network. 

- `docker container run --network <virtual_net> <container>`: run a container with a specific network. 

It is possible to attach a container to 2 networks. The way to do that would be to connect a container to a specific network in the following way

- `docker network connect <network_id> <container_id>`: essentially attach a container to a network. 

#### Docker Networks: DNS

Docker uses the container names as the host names for containers talking to each other. Lets say we run a container with a name and specify a network for that container. When the container is attached with a network, the DNS of that container will automatically be the name of that container. 

```bash
docker network create test_net
docker container run -d --network test_net --name nginx1 nginx
docker container run -d --network test_net --name nginx2 nginx
docker container exec -it nginx1 ping nginx2
```
When we execute the last command we find that it is possible to ping container nginx2 simply by using the name of the container as the name becomes the host name of the container as well

Here we can see how important it is to specify names of our containers. 

For the default bridge network there is no default DNS service built into it. So in order to link different containers we can use the `--link` option while creating a container.

When we create multiple containers running the same piece of software, lets say an elasticsearch server, it is possible to have an alias to access those containers such that we have kind of a load balancer between the containers. This can be achieved in the following way:

```bash
docker network create dude
docker container run -d --net dude --net-alias search elasticsearch:2 
docker container run -d --net dude --net-alias search elasticsearch:2 # 2 containers started with the same alias
docker container run --rm --net dude alpine nslookup search

# nslookup obviously is nameserver lookup. So when we lookup the name search we actually get 2 different container ip addresses

docker container run --rm --net dude centos curl -s search:9200

# this gives us back the elasticsearch response on 9200
```
If we actually run that last command over and over again we will be able to access the 2 containers not necessarily in a round robin fashion. This behaves like a load balancer that balances the load rather randomly.

### Docker Images
 
Images are stored as fs layers. Each layer is stored only once for the entire system which means that if there are 2 images that have the same base layer say then the 2 base layers are not stored twice for the 2 images. It is stored only once and each image actually uses that same layer.

When a container is launched of an image, docker actually adds another file system layer on top of the topmost image layer. If we modify an image layer that is shared by another image, docker actually copies that image layer on top of the existing layer and makes a new layer only for that image, so that the original image layer is unmodified for other images to share. 

```bash
docker image tag <source image with an optional tag> <destination image with an optional tag> # used for tagging images
docker image push <image name with tag>
```

### The Dockerfile

The Dockerfile has instructions for building docker images. Each directive in the dockerfile serves as a layer therefore the order in which the directives are written actually matters. So lets say if there are 2 `RUN` directives then it means that those will be 2 layers in the file system.

Note: In case we want to log something from one of our apps that is running inside our container, the best way to log it is to redirect all logs to stdout and stderr. This can be done in the following way:
```dockerfile
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log
```
We are simply linking `/dev/stdout` to `/var/log/nginx/access.log` and `/dev/stderr` to `/var/log/nginx/error.log`

When we are building a dockerfile each layer that gets built actually gets cached as well, so the next time we build the dockerfile, the layers that did not change are not rebuilt. If we change one line in the dockerfile, the step at which it got changed will be rebuilt along with all the steps succeeding it. It is therefore very important that we get the order of the steps right so that the lines are less prone to change are at the top and the ones that are more prone to change are at the bottom. 

`WORKDIR` is the working directory in the dockerfile

We can potentially get away with the `CMD` command in a dockerfile if our base image already has some `CMD` in it. When we use a base image we inherit everything except the `ENV`s from the base image. 

### Container Lifetime and Persistent Volumes

Containers are in general **immutable and ephemeral**. These are important buzz words in the container industry. They are just fancy ways of saying that containers are designed to be immutable and disposable. That means that once a container is running, we should not be able to change stuff inside the container, if we need to do so we should throw away the existing container and build a new image for a new container. 

However we should be able to have all our data separate from all our container related settings and configs. This is called **separation of concerns** and the data that we are trying to separate is called persistent data. 

There are actually 2 ways in which docker handles persistent data:
- volumes
- bind mounts

**Volumes** actually make special locations outside the container's union file system. 
**Bind Mounts** link container paths to host paths.

#### Unnamed volumes or simply volumes

volumes are for storage of persistent data. One way by which we can configure a volume for persistent data is by stating it in the dockerfile, in the following way:

```dockerfile
VOLUME /var/lib/mysql
```
This creates an unnamed volume that points to `/var/lib/mysql` in the container. During runtime of the container, anything that is stored in this location will remain persistent. Even if the container is stopped or even removed, any data that was stored in that volume remains persistent. If we inspect the container that has the volume, we would be able to see the actual location where the volume data is stored. On a linux machine we would be able to actually navigate to this location however on a windows or a mac machine, that runs docker on a linux vm, the source location is actually present in that volume and not on the real machine. 
 
Another way to state the volume would be to use the `-v` option along with the `container run` command, in the following way

```bash
docker container run -v /var/log/nginx nginx
``` 
This would have the same impact as the `VOLUME` directive in the dockerfile. Once a container is stopped, if we spin another container with the same image, a new location will be set up. However if we start the previously stopped container that same location will persist. 

#### Named volumes

Another way to have persistent data is to have a named volume. A named volume can be configured in a dockerfile in the following way:

```dockerfile
VOLUME logs-vol:/var/log/nginx/access.log
```
When a named volume is created, if there are 2 containers created having the same named volume, the persistent data is shared across the containers.

At container run time it can be done in the following way:

```bash
docker container run -v logs-vol:/var/log/nginx/access.log nginx
```

#### Bind Mounts

With bind mounts, a directory in the host file system is actually mapped into a directory in the container. This means that bind mounts cannot actually be written inside dockerfiles as binding is done during runtime. 

```bash
docker container run -v /var/log:/var/log nginx
```

The left side is the host machine and the right side is the container. 

### Docker Compose

Main sections in docker-compose.yml file
- services
- volumes
- networks

Sample docker-compose.yml

```yml
version: '3.1'  # if no version is specificed then v1 is assumed. Recommend v2 minimum

services:  # containers. same as docker run
  servicename: # a friendly name. this is also DNS name inside network
    image: # Optional if you use build:
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker run
    volumes: # Optional, same as -v in docker run
  servicename2:

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create
```