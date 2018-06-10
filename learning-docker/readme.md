faqs relating to udemy docker course: https://www.udemy.com/docker-mastery/learn/v4/t/lecture/10091380?start=0

## Docker Commands

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
