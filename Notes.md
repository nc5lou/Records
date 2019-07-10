# My Notes
 - Docker 
 - Etherum
 - Python-Java-Go-NodeJS-Solidity
 - Rasperberry PI

# Docker 
----
### Install Docker
```
$ uname -r  查看内核版本
$ sudo apt-get remove docker docker-engine docker.io containerd runc
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
$ sudo groupadd docker
$ sudo groupadd $user
```

### Docker Commands
```
$ docker --version
$ docker info
$ docker image ls
$ docker container ls
```

### build container by docker
```
prepare the dockerfile
$ docker build --tag=friendlyhello .
$ docker image ls
$ docker run -d -p 4000:80 friendlyhello
$ curl http://localhost:4000
$ docker container stop $container_ID
```

### share personal image by docker
```
$ docker login
$ docker tag image username/repository:tag
   # for example: docker tag friendlyhello gordon/get-started:part2
$ docker image ls
$ docker push username/repository:tag
$ docker run -p 4000:80 username/repository:tag  
   # you can use docker run and run your app on any machine with this command
```

### scale app on the swarm mode in docker 
```
docker-compose.yml is a YAML file that defines how Docker containers should behave in production
$ docker swarm init
$ docker stack deploy -c docker-compose.yml getstartedlab
$ docker service ls
$ docker service ps getstartedlab_web
$ docker container ls -q
$ docker stack rm getstartedlab
$ docker swarm leave --force
```


#### Install docker-machine
```
 $ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
     curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
     sudo install /tmp/docker-machine /usr/local/bin/docker-machine
 $ sudo apt-get install virtualbox
```
### delete container or image in docker

- docker rm container_ID 是删除某个特定ID的container
- docker rmi image_ID 是删除某个特定ID的image
- docker kill $(docker ps -a -q) 杀死所有正在运行的容器
- docker rm $(docker ps -a -q)   删除所有已经停止的容器
- docker rmi $(docker images -q -f dangling=true)  删除所有未打 dangling 标签的镜像
- docker rmi $(docker images -q)   删除所有镜像
- docker rmi --force $(docker images -q) 强制删除所有镜像
- docker rmi --force $(docker images | grep doss-api | awk '{print $3}') 强制删除镜像名称中包含“doss-api”的镜像

----

# Etherum 


# Python-Java-Go-NodeJS-Solidity 
## Python 
   
## Java

## Go

## NodeJS

## Solidity


## Rasperberry PI
