(docker install : convenient script )[docs.docker.com]
(userful docker images)[hub.docker.com]

#cheat sheet : Docker

## run command 
- docker run docker/whalesay
- docker run docker/whalesay sleep 5   *will run after 5 secs*
- docker run -d <docker image>         *runs the image in background*
   -- docker logs <docker container name>   *will show logs of containers run in the background*
- docker attach <docker container name>     *attaches to the background process and brings it to foreground*
- docker exec <docker container name> <command to run>   *runs the command inside the docker container*

*NOTE, docker runs in a non interactive mode, does not have a terminal, so cannot take STDIN*
- docker run -i <docker image>    *starts docker in interactive mode*
- docker run -it <docker image>   *starts docker in interactive mode and attaches to host terminal*

### run misc
- docker run  -e <ENVIRONMENT VAR=VAL> <docker image>   *sets the environment variable and runs the container*

### volume mapping : dir mapping to host 
*NOTE, if the volume is not created and docker is run with -v, then the data volume to map gets
 automatically created under /var/lib/docker*

- docker run -v /home/bhaskar:/var/lib/mysql mysql_app   *will map the docker container directory to host dir /home/bhaskar will help in persisting data, even after stopping docker container*
- docker run --mount type=bind source=data_volume1 target=/var/lib/mysql mysql_app *new way of doing it*

### run command with older version : called a TAG
- docker run <docker image name>:<VERSION> *example,  docker run redis:4.0*

### run with network associated 
*NOTE, the docker container name can be used to resolve host IP address*
- docker run <appname> --network=none  *to run with no network*
- docker run <appname> --network=host  *to run with host network, since it uses host port, multiple containers cannot be run on the same port*
- docker run <appname>                 *to run with bridge network, can create multiple containers on the same port (since IP address will be different)*

### port mapping to access ports in docker
- docker run -p 80:8080 kodeklode/simple-webapp  *this command will map the internal docker container port 8080 to host port 80

## create user defined network
- docker network create --driver bridge --subnet 10.1.0.0/16 my-isolated-network
- docker network ls *will list network in docker

## ps command
- docker ps     *shows the running containers
- docker ps -a  *prints all containers that run including the ones that exited
- docker stop <docker container name> *stops the container

## rm command
- docker rm <docker container name>  *removes the container completely
- docker rmi <docker image name>   *removes the image, ensure, you have stopped, rm and then rmi

## inspect command 
- docker inspect <docker container name> *gives all information about the docker container, including IP, mounts, states etc.

## show images 
- docker images

## docker history
- docker history <image name> *shows the history of the image with sizes

## docker build
1. create the Dockerfile  *start with OS : FROM ubuntu for example
- docker build -t sample/mycustom-app .

## Dockerfile : nuances
`CMD *this is to run when container starts. For example, `CMD sleep 10`, will make container sleep for 10 secs*
ENTRYPOINT ["<command>"]    *will ensure that the argument is passed to whatever command is in ENTRYPOINT. For example*
                            *ENTRYPOINT ["sleep"]  and if `docker run sleep-app 10`, then argument 10 is passed to*
                            sleep in ENTRYPOINT. Hence container sleeps for 10 secs*
`
## docker-compose.yml
*if multiple docker containers need to be brought up, it can be done in yml*

*docker-compose.yml*
`
services: 
     web:
         image: "simple-webapp"
     database: 
         image: "mongodb"
     orchestration:
         image: "ansible"
`
*EOF*

*now run command*
- docker-compose up

### linking to another container for resolving by name in the app
- docker run -d --name=vote-app -p 5000:80 --link redis:redis voting-app  *this names the voting-app container as vote-app and links to a container redis named redis and maps port 80 to 5000 in host*
*sample docker-compose.yml*
```
version: 2/3                            *read documentation in understanding difference and features between 2 and 3*
services:
    redis:
        image: redis
        networks:
             - back-end
    db: 
        image: db
        networks:
             - back-end
    vote:
        image: voting-app
        links: 
             - redis
        ports:
             - 5000:80
        networks:
             - front-end
             - back-end
    result:
        build: ./result-app
        depends_on:                       *version 2 of docker compose, does not require links, however can specify order*
             - db
        ports:
             - 5001:80
        networks:
             - front-end
             - back-end
networks:
    front-end:
    back-end:
```
*EoF*
