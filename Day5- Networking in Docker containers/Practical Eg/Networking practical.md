

# Container C1 & C2 on same network

1. We will create 2 containers - C1 & C2 and run them in detached state in background

- docker run -d --name login nginx:latest
- docker run -d --name logout nginx:latest

```
docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                    NAMES
d05a511841dd   nginx:latest             "/docker-entrypoint.…"   4 seconds ago    Up 3 seconds    80/tcp                   logout

437b66709bf1   nginx:latest             "/docker-entrypoint.…"   52 seconds ago   Up 51 seconds   80/tcp                   login

```
2. Login to the container and install ping on login container. 

- docker exec -it login /bin/bash

    apt-get update -y
    apt-get install -y iputils-ping

3. We will check the IP OF OUR CONTAINERS

- docker ps
  docker inspect <name of container>

  docker inspect login
    "IPAddress": "172.17.0.3",

  docker inspect logout
    "IPAddress": "172.17.0.4",


Since we have not mentioned any custom bridge network while running the containers, they are using the same default bridge network, and must be in the same subnet.

4. We will ping the iP of logout container from login contaienr. We will see if they will be able to access the other container which are on same host.

```
docker exec -it login /bin/bash          
root@437b66709bf1:/# ping 172.17.0.4
PING 172.17.0.4 (172.17.0.4) 56(84) bytes of data.
64 bytes from 172.17.0.4: icmp_seq=1 ttl=64 time=7.30 ms
64 bytes from 172.17.0.4: icmp_seq=2 ttl=64 time=1.31 ms
64 bytes from 172.17.0.4: icmp_seq=3 ttl=64 time=0.200 ms
64 bytes from 172.17.0.4: icmp_seq=4 ttl=64 time=0.206 ms

```

Yes, since they are using default OOB bridge networking, which allows one container to talk to other container.

- docker network ls - will list all the network on the host.


```
docker network ls
NETWORK ID     NAME       DRIVER    SCOPE
499be84d1e4f   bridge     bridge    local
494cde14a147   host       host      local
30052472ff0f   kind       bridge    local
c0bce477aec1   minikube   bridge    local
d332effe0544   none       null      local

```

# Container C3 on diff network than C1 & C2

We will create a finance container that has to be logically isolated from finance container. We will create a Bridge Network for our finance container.

The below will create a custom bridge network for us - 

`docker network create secure-network`

`dokcer network ls`

```
NETWORK ID     NAME             DRIVER    SCOPE
499be84d1e4f   bridge           bridge    local
494cde14a147   host             host      local
30052472ff0f   kind             bridge    local
c0bce477aec1   minikube         bridge    local
d332effe0544   none             null      local
6f7028a8eb31   secure-network   bridge    local

```
Now we can see that we have a custom bridge network and a bridge network by default. We will connect this network to our finance docker container, and then try to ping container c1 from c3 and vice versa

### How to attach a custom network to a docker container?

To attach a custome bridge network to a docker container, we make use of --network parameter

`docker run -d --name finance --network secure-network nginx:latest `

```
docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS         PORTS                    NAMES
cb1aa1f24392   nginx:latest             "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   80/tcp                   finance
d05a511841dd   nginx:latest             "/docker-entrypoint.…"   3 hours ago     Up 3 hours     80/tcp                   logout
437b66709bf1   nginx:latest             "/docker-entrypoint.…"   3 hours ago     Up 3 hours     80/tcp                   login

```

We will look for the IP using docker inspect command - "IPAddress": "172.19.0.2"

`docker exec -it finance /bin/bash`

Below is the ping result of trying to ping Container C1 & C2. Both are not reachable.

```
root@cb1aa1f24392:/# ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
^C
--- 172.17.0.3 ping statistics ---
6 packets transmitted, 0 received, 100% packet loss, time 5106ms

root@cb1aa1f24392:/# ping 172.17.0.4 
PING 172.17.0.4 (172.17.0.4) 56(84) bytes of data.
^C
--- 172.17.0.4 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2075ms

root@cb1aa1f24392:/# 

```

If we try to ping C3 from C1 login container, it also fails

```
root@437b66709bf1:/# ping 172.19.0.2
PING 172.19.0.2 (172.19.0.2) 56(84) bytes of data.
^C
--- 172.19.0.2 ping statistics ---
5 packets transmitted, 0 received, 100% packet loss, time 4134ms

root@437b66709bf1:/# 

```
# What are the differences?

1. The Network section when you do docker inspect has Bridge for C1 & C2 while for C3, its "secure-network".
2. The IP Address for C1 & C2 belong to same subset, but for C3 its different.

C3 -"172.19.0.2"
C2 - 172.17.0.3
C1 - 172.17.0.4 


## Delete all the resources

terminal ~ % docker stop login
login
terminal ~ % docker stop finance
finance
terminal ~ % docker stop logout
logout
terminal ~ % docker rm login
login
terminal ~ % docker rm logout
logout
terminal ~ % docker rm finance
finance
terminal ~ % docker ps
CONTAINER ID   IMAGE                    COMMAND    CREATED       STATUS       PORTS                    NAMES
49e461e49df6   asds                      "./main"   6 weeks ago   Up 13 days   sd
terminal ~ % docker network rm secure-network
secure-network
