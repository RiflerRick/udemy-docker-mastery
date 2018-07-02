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


