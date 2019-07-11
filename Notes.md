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

### docker plus Etherum
```
$ docker pull ethereum/client-go
$ docker run -d --name eth-ropsten-node -v          HOME/geth/ropsten:/root \
  -p 8545:8545 -p 30303:30303 \
  ethereum/client-go \
  --testnet \
  --syncmode "fast" \
  --cache=1024
 $ docker exec -it eth-ropsten-node geth attach ipc:/root/.ethereum/testnet/geth.ipc 查看区块同步情况
 $ eth 查看支持的function
 $ eth.getBlock(99)
```
稍微說明一下這段指令：
- run：執行一個新的 Container
- -d：在背景執行 Container 並且列印出 Container 的 ID
- --name eth-ropsten-node：Container 的名稱
- -v：綁定 Volume 的位置，$HOME/get/ropsten 是本機端的目錄，對映到 Container 的 root 目錄
- -p 8545:8545 -p 30303:30303：要 expose 的 port
- ethereum/client-go：指定映像檔名稱，使用我們剛剛拉取回來的映像檔
- --testnet、--syncmode "fast"、--cache=1024：當容器起來後，會執行 ENTRYPOINT 的指令，可以參考上面的 Dockerfile
- 使用測試鏈的資料，加上 --testnet（使用 Ropsten 測試鏈），預設會同步主鏈
- 設定同步模式：--syncmode "fast"（模式有： fast、full、light）
- 設定區塊的 cache 大小為 1024（預設就是為 1024，可以省略不打）


### build private etherum blockchain
```


```














# Python-Java-Go-NodeJS-Solidity 
## Python 
   
## Java

## Go

## NodeJS

## Solidity


## Rasperberry PI
