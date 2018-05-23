# Docker Networking Workshop

Hi, welcome to the Docker Networking Workshop!

You will get your hands dirty by going through examples of a few basic networking concepts, learn about Bridge and Overlay networking, and finally learning about the Swarm Routing Mesh.

> **Difficulty**: Beginner to Intermediate

> **Time**: Approximately 45 minutes

> **Tasks**:
>
> * [Section #1 - Networking Basics](#task1)
> * [Section #2 - Bridge Networking](#task2)
> * [Section #3 - Overlay Networking](#task3)

# <a name="task1"></a>Section #1 - Networking Basics

## <a name="list_networks"></a>Step 1: The Docker Network Command

Connect to the **manager1** node in your PWD session to explore the existing Docker networks.

The `docker network` command is the main command for configuring and managing container networks. Run the `docker network` command from **manager1**.

```
$ docker network

Usage:    docker network COMMAND

Manage networks

Options:
      --help   Print usage

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
```

The command output shows how to use the command as well as all of the `docker network` sub-commands. As you can see from the output, the `docker network` command allows you to create new networks, list existing networks, inspect networks, and remove networks. It also allows you to connect and disconnect containers from networks.

## <a name="list_networks"></a>Step 2: List networks

Run a `docker network ls` command on **manager1** to view existing container networks on the current Docker host.

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
bf49ba724655        bridge              bridge              local
f88f42dbcd4c        docker_gwbridge     bridge              local
qcagnwr8f6xh        dtr-ol              overlay             swarm
f34066c16e6a        host                host                local
86zgov3fztu8        ingress             overlay             swarm
69826fd119d8        none                null                local
```

The output above shows the container networks that are created as part of a standard installation of Docker EE.

> These networks are cluster-wide in Docker EE. You can run the `docker network ls` command on any other node and you will see the same output.

New networks that you create will also show up in the output of the `docker network ls` command.

You can see that each network gets a unique `ID` and `NAME`. Each network is also associated with a single driver. Notice that the "bridge" network and the "host" network have the same name as their respective drivers.

## <a name="inspect"></a>Step 3: Inspect a network

The `docker network inspect` command is used to view network configuration details. These details include; name, ID, driver, IPAM driver, subnet info, connected containers, and more.

Use `docker network inspect <network>` on **manager1** to view configuration details of the container networks on your Docker host. The command below shows the details of the network called `bridge`.

```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "3430ad6f20bf1486df2e5f64ddc93cc4ff95d81f59b6baea8a510ad500df2e57",
        "Created": "2017-04-03T16:49:58.6536278Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

> **NOTE:** The syntax of the `docker network inspect` command is `docker network inspect <network>`, where `<network>` can be either network name or network ID. In the example above we are showing the configuration details for the network called "bridge". Do not confuse this with the "bridge" driver.

## <a name="list_drivers"></a>Step 4: List network driver plugins

Docker has a plugin system to enable different network topologies using different network drivers.

You can see all the installed plugins (for volumes, networks and logging) from the `docker info` command.

Run `docker info` on **manager1** and locate the list of network plugins:

```
$ docker info
Containers: 31
 Running: 27
 Paused: 0
 Stopped: 4
Images: 24
Server Version: 17.06.2-ee-11
...
Plugins:
 Volume: local
 Network: bridge host ipvlan macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
...
```

The output above shows the **bridge**, **host**, **ipvlan**, **macvlan**, **null**, and **overlay** drivers are installed.

# <a name="task2"></a>Section #2 - Bridge Networking

==TODO - intro==

## <a name="connect-container"></a>Step 1: The Basics

Every clean installation of Docker comes with a pre-built network called **bridge**. Verify this with the `docker network ls` command on **manager1**. filtering on the network driver:

```
$ docker network ls --filter driver=bridge
NETWORK ID          NAME                DRIVER              SCOPE
bf49ba724655        bridge              bridge              local
f88f42dbcd4c        docker_gwbridge     bridge              local
```

The output above shows two networks associated with the *bridge* driver: 

* **bridge** is a default network created when you install Docker, even in a single-node installation

* **docker_gwbridge** is a default network created when you deploy UCP in Docker EE

The output above also shows that the bridge networks are scoped locally. This means that the network only exists on this Docker host. This is true of all networks using the *bridge* driver - the *bridge* driver provides single-host networking.

> In a Docker EE cluster, these networks are replicated on all the nodes. Every node has its own **bridge** and **docker_gwbridge** network, but each is local to the node and there is no traffic between them.

All networks created with the *bridge* driver are based on a Linux bridge (a.k.a. a virtual switch).

## <a name="connect-container"></a>Step 2: Connect a container

The **bridge** network is the default network for new containers. This means that unless you specify a different network, all new containers will be connected to the **bridge** network.

Create a new bridge network on **manager1** and call it `br`.

```
docker network create -d bridge br
```

> This custom network only exists on node **manager1**. If you list the networks on another node, you will not see the new **br** network.

Now create a container called `c1` and attach it to your new `br` network on **manager1**.

```
docker container run -itd --net br --name c1 alpine sh
```

This command will create a new container based on the `alpine:latest` image. 

Running `docker network inspect br` will show the containers on that network.

```
$ docker network inspect br

[
    {
        "Name": "br",
        "Id": "056b27c1488e56bda4e11ca0937166f0512031cddd4f4b68ff5bb2a1d11b136a",
        "Created": "2018-05-23T10:53:43.594987763Z",
        "Scope": "local",
        "Driver": "bridge",
        ...
        "Containers": {
            "f849e0f45933da5f2324f132ed6224511fc6c8ea95532858ee6753a2a1054746": {
                "Name": "c1",
                "EndpointID": "f43ccc304a907615fd341b195d2bcce5761c0ab4b9f19ae52a24e56b25b29eb0",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            }
        },
        ...
    }
]
```


## <a name="ping_local"></a>Step 3: Test network connectivity

The output to the previous `docker network inspect` command shows the IP address of the new container. In the previous example it is `172.20.0.2` but yours might be different.

> You can get the container's IP address for the **br** network by inspecting the container and formatting the response: `docker container inspect c1 --format "{{ .NetworkSettings.Networks.br.IPAddress }}"`

Ping the IP address of the container from the shell prompt of your Docker host by running `ping -c 3 <IPv4 Address>` on **node0**. Remember to use the IP of the container in **your** environment.

You can get the IP address of the **c1** container directly from the Docker engine by running `docker inspect --format "{{ .NetworkSettings.Networks.br.IPAddress }}" c1`.

```
$ ping -c 3 172.20.0.2
PING 172.20.0.2 (172.20.0.2) 56(84) bytes of data.
64 bytes from 172.20.0.2: icmp_seq=1 ttl=64 time=0.087 ms
64 bytes from 172.20.0.2: icmp_seq=2 ttl=64 time=0.067 ms
64 bytes from 172.20.0.2: icmp_seq=3 ttl=64 time=0.048 ms
...
```

The replies above show that the Docker host can ping the container over the **bridge** network. But, we can also verify the container can connect to the outside world too. 

Enter in to the **c1** container that you created using the command `docker container exec`. We will pass the `sh` command to `container exec` which puts us in to an interactive shell inside the container.

Enter in to the container and inspect the interfaces of the container:

```
$ docker exec -it c1 sh

 # ip addr show | grep inet
    inet 127.0.0.1/8 scope host lo
    inet 172.20.0.2/16 scope global eth
```

The IP address of the container is accessible from the host. Now prove that containers can gain outside access by pinging `www.docker.com`.

```
 # ping -c 3 www.docker.com

PING www.docker.com (52.85.101.235): 56 data bytes
64 bytes from 52.85.101.235: seq=0 ttl=237 time=7.428 ms
64 bytes from 52.85.101.235: seq=1 ttl=237 time=7.451 ms
64 bytes from 52.85.101.235: seq=2 ttl=237 time=7.406 ms
```

Exit out of the container.

```
 # exit
```

Now you will create a second container on this bridge so you can test connectivity between them.

```
docker container run -itd --net br --name c2 alpine sh
```

Containers **c1** and **c2** are connected to the same bridge network, so they are visible to each other and to the Docker host.

Use `docker container exec` to ping the IP address of the **c1** container from **c2**:

```
$ docker container exec -it c2 ping -c 2 172.20.0.2
PING 172.20.0.2 (172.20.0.2): 56 data bytes
64 bytes from 172.20.0.2: seq=0 ttl=64 time=0.072 ms
64 bytes from 172.20.0.2: seq=1 ttl=64 time=0.070 ms
...
```

Now ping container **c1** using it's name:

```
$ docker container exec -it c2 ping -c 2 c1
PING c1 (172.20.0.2): 56 data bytes
64 bytes from 172.20.0.2: seq=0 ttl=64 time=0.095 ms
64 bytes from 172.20.0.2: seq=1 ttl=64 time=0.108 ms
...
```

The Docker engine provides DNS resolution automatically for all container names and service names.

We've been running containers directly on the **manager1** node, using a private network on that node. Connect to **worker1** and verify that the container isn't accessible on that node:

```
$ ping -c 2 -W 2 172.20.0.2
PING 172.20.0.2 (172.20.0.2) 56(84) bytes of data.

--- 172.20.0.2 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1003ms
```

> The container IP address is private to the host when you're using bridge networking.

Now switch back to **manager1** and remove the containers:

```
docker rm -f c1
docker rm -f c2
```

## <a name="nat"></a>Step 4: Configure NAT for external connectivity

==TODO - intro==

In this step we'll start a new **NGINX** container and map port 8000 on the Docker host to port 80 inside of the container. This means that traffic that hits the Docker host on port 8000 will be passed on to port 80 inside the container.

> **NOTE:** If you start a new container from the official NGINX image without specifying a command to run, the container will run a basic web server on port 80.

Start a new container based off the official NGINX image by running a container on **manager1**.

```
docker container run --name web1 -d -p 8000:80 nginx:alpine
```

Review the container status and port mappings by running `docker container ls` on **manager1**, filtering fot the web container name:

```
$ docker container ls --filter name=web1
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
62782dc5f612        nginx:alpine        "nginx -g 'daemon ..."   30 seconds ago      Up 29 seconds       0.0.0.0:8000->80/tcp   web1
```

The top line shows the new **web1** container running NGINX. Take note of the command the container is running as well as the port mapping - `0.0.0.0:8000->80/tcp` maps port 8000 on all host interfaces to port 80 inside the **web1** container. This port mapping is what effectively makes the containers web service accessible from external sources (via the Docker hosts IP address on port 8000).

Now that the container is running and mapped to a port on a host interface you can test connectivity to the NGINX web server, by browsing to the master node.

IN the PWD session information, you will see the **UCP Hostname** listed - it will be a long domain, something like `ip172-18-0-6-bc2m2d8qitt0008vqor0.direct.beta-hybrid.play-with-docker.com`. Browse to port `8000` at that address and you will see Nginx running - Docker receives the request and routes it into the container.

> **NOTE:** The port mapping is actually port address translation (PAT).

# <a name="task3"></a>Section #3 - Overlay Networking

Overlay networks span all the nodes in a Docker swarm mode cluster. Docker EE is built on top of swarm mode. You have multiple nodes in your lab environment so you can verify that containers in an overlay network can reach it other, even if they are running on different hosts.

## <a name="connect-container"></a>Step 1: The Basics

Run `docker node ls` on **manager1** to verify that you have multiple active nodes in the swarm:

```
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
lo6fasr4izuz6okhm411vmyu3 *   manager1            Ready               Active              Leader
o0b82a6jwp3jfibixttb06fk8     worker3             Ready               Active
u4xc7yyu1j8hrea4dudr51ncx     worker2             Ready               Active
wuy5h23ly2y4wx5ccrjiscohv     worker1             Ready               Active
```

The `ID` and `HOSTNAME` values may be different in your lab. The important thing to check is that both nodes have joined the Swarm and are *ready* and *active*.

## <a name="create_network"></a>Step 2: Create an overlay network

Now that you have a Swarm initialized it's time to create an **overlay** network.

Create a new overlay network called **overnet** by running `docker network create -d overlay overnet` on **manager1**.

```
$ docker network create -d overlay overnet
wlqnvajmmzskn84bqbdi1ytuy
```

Use the `docker network ls` command to verify the network was created successfully:

```
$ docker network ls --filter name=overnet
NETWORK ID          NAME                DRIVER              SCOPE
2s4a0n00wdhd        overnet             overlay             swarm
```

The new "overnet" network is associated with the **overlay** driver and is scoped to the entire swarm - compare this to the **bridge** and **nat** networks which are locally scoped to a single node.

Run the `docker network ls` command on **worker1** to show all the overlay networks:

```
$ docker network ls --filter driver=overlay
NETWORK ID          NAME                DRIVER              SCOPE
55f10b3fb8ed        bridge              bridge              local
b7b30433a639        docker_gwbridge     bridge              local
a7449465c379        host                host                local
8hq1n8nak54x        ingress             overlay             swarm
06c349b9cc77        none                null                local
```

> The **overnet** network is not in the list. This is because Docker only extends overlay networks to hosts when they are needed. This is usually when a host runs a task from a service that is created on the network. We will see this shortly.

Use the `docker network inspect` command to view more detailed information about the "overnet" network. You will need to run this command from **manager1**.

```
$ docker network inspect overnet
[
    {
        "Name": "overnet",
        "Id": "wlqnvajmmzskn84bqbdi1ytuy",
        "Created": "0001-01-01T00:00:00Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": false,
        "Containers": null,
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": null
    }
]
```

## <a name="create_service"></a>Step 3: Create a service

Now that we have an overlay network in our Docker EE cluster, it's time to create a service that uses the network.

Execute the following command from **manager1** to create a new service called *myservice* on the *overnet* network with two tasks/replicas.

```
docker service create --name ubuntu \
--network overnet \
--replicas 6 \
sixeyed/ubuntu-with-utils sleep infinity
```

Verify that the service is created and both replicas are up by running `docker service ls` and filtering on the service name:

```
$ docker service ls --filter name=ubuntu
ID                  NAME                MODE                REPLICAS            IMAGE                              PORTS
yxt9w3573qb5        ubuntu              replicated          6/6                 sixeyed/ubuntu-with-utils:latest                ubuntu:latest
```

The `6/6` in the `REPLICAS` column shows that all the tasks in the service are up and running.

Verify that the tasks (replicas) are running on different swarm nodes `docker service ps`:

```
$ docker service ps ubuntu
ID                  NAME                IMAGE                              NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
5z0u26v62qof        ubuntu.1            sixeyed/ubuntu-with-utils:latest   worker2             Running             Running about a minute ago
ph7ps3273w5m        ubuntu.2            sixeyed/ubuntu-with-utils:latest   manager1            Running             Running about a minute ago
tueyl7niinsb        ubuntu.3            sixeyed/ubuntu-with-utils:latest   worker3             Running             Running about a minute ago
...
```

The `ID` and `NODE` values might be different in your output. The important thing to note is that every node is running at least one replica, which means multiple containers across the swarm, all connected to the same overlay network.

Now that all the nodes are running tasks on the "overnet" network, you will be able to see that network from every node. Connect to **worker2** and verify that by running `docker network ls` and filtering for overlay networks:

```
$ docker network ls --filter driver=overlay
NETWORK ID          NAME                DRIVER              SCOPE
jfy2cbcbw567        ingress             overlay             swarm
j20kcrvwhqw4        overnet             overlay             swarm
```

> You'll learn about the **ingress** network later in this workshop.

We can also run `docker network inspect overnet` on **worker2** to get more detailed information about the "overnet" network and obtain the IP address of the task:

```
$ docker network inspect overnet
[
    {
        "Name": "overnet",
        "Id": "j20kcrvwhqw45ujxay2m7q1af",
        "Created": "2018-05-23T15:19:36.669608649Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1"
                }
            ]
        },
        ...
        "Containers": {
            "67264d255b873567c30375e8d021146cbae7c0a162c9297d143471fd14e0ce54": {
                "Name": "ubuntu.5.4rrqbneiay7uau63m07blofh5",
                "EndpointID": "a1530106c077896f4c24ae7054039a9ea728ae72831955802e92ac8c3bf31075",
                "MacAddress": "02:42:0a:00:00:0b",
                "IPv4Address": "10.0.0.11/24",
                "IPv6Address": ""
            }
            ...
```

> The subnet is specific to the **overnet** network in the swarm. In this case it is **10.0.0.0/24**, which means any containers attached to the network will have an IP address in the range **10.0.0.1** to **10.0.0.254**.

You should note that `docker network inspect` only shows containers/tasks running on the local node. This means that **10.0.0.11** is the IPv4 address of the container running on **worker2**. Make a note of this IP address for the next step (the IP address in your lab might be different than the one shown here in the lab guide).

## <a name="test"></a>Step 4: Test the network

To complete this step you will need the IP address of the service task running on the worker that you saw in the previous step (in this example it was **10.0.0.11**).

Execute the following commands from **worker3**, to verify that containers on this node can connect to containers on **worker2**.


>> DONE TO HERE


```
$ docker network inspect overnet
[
    {
        "Name": "overnet",
        "Id": "wlqnvajmmzskn84bqbdi1ytuy",
        "Created": "2017-04-04T09:35:47.362263887Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.10.10.0/24",
                    "Gateway": "10.10.10.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "d676496d18f76c34d3b79fbf6573a5672a81d725d7c8704b49b4f797f6426454": {
                "Name": "myservice.2.nlozn82wsttv75cs9vs3ju7vs",
                "EndpointID": "36638a55fcf4ada2989650d0dde193bc2f81e0e9e3c153d3e9d1d85e89a642e6",
                "MacAddress": "02:42:0a:00:00:04",
                "IPv4Address": "10.10.10.4/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": {},
        "Peers": [
            {
                "Name": "node0-f6a6f8e18a9d",
                "IP": "10.10.10.5"
            },
            {
                "Name": "node1-507a763bed93",
                "IP": "10.10.10.6"
            }
        ]
    }
]
```

Notice that the IP address listed for the service task (container) running on **node0** is different to the IP address for the service task running on **node1**. Note also that they are one the sane "overnet" network.

Run a `docker ps` command to get the ID of the service task on **node0** so that you can log in to it in the next step.

```
$ docker ps
CONTAINER ID        IMAGE                                                                            COMMAND                  CREATED             STATUS              PORTS                           NAMES
d676496d18f7        ubuntu@sha256:dd7808d8792c9841d0b460122f1acf0a2dd1f56404f8d1e56298048885e45535   "sleep infinity"         10 minutes ago      Up 10 minutes                                       myservice.2.nlozn82wsttv75cs9vs3ju7vs
<Snip>
```

Log on to the service task. Be sure to use the container `ID` from your environment as it will be different from the example shown below. We can do this by running `docker exec -it <CONTAINER ID> /bin/bash`.

```
$ docker exec -it d676496d18f7 /bin/bash
root@d676496d18f7:/#
```

Install the ping command and ping the service task running on **node1** where it had a IP address of `10.10.10.3` from the `docker network inspect overnet` command.

```
root@d676496d18f7:/# apt-get update && apt-get install -y iputils-ping
```

Now, lets ping `10.10.10.3`.

```
root@d676496d18f7:/# ping -c5 10.10.10.3
PING 10.10.10.3 (10.10.10.3) 56(84) bytes of data.
^C
--- 10.10.10.3 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 2998ms
```

The output above shows that both tasks from the **myservice** service are on the same overlay network spanning both nodes and that they can use this network to communicate.

## <a name="discover"></a>Step 5: Test service discovery

Now that you have a working service using an overlay network, let's test service discovery.

If you are not still inside of the container on **node0**, log back into it with the `docker exec -it <CONTAINER ID> /bin/bash` command.

Run `cat /etc/resolv.conf` form inside of the container on **node0**.

```
$ docker exec -it d676496d18f7 /bin/bash
root@d676496d18f7:/# cat /etc/resolv.conf
search ivaf2i2atqouppoxund0tvddsa.jx.internal.cloudapp.net
nameserver 127.0.0.11
options ndots:0
```

The value that we are interested in is the `nameserver 127.0.0.11`. This value sends all DNS queries from the container to an embedded DNS resolver running inside the container listening on 127.0.0.11:53. All Docker container run an embedded DNS server at this address.

> **NOTE:** Some of the other values in your file may be different to those shown in this guide.

Try and ping the "myservice" name from within the container by running `ping -c5 myservice`.

```
root@d676496d18f7:/# ping -c5 myservice
PING myservice (10.10.10.2) 56(84) bytes of data.
64 bytes from 10.10.10.2: icmp_seq=1 ttl=64 time=0.020 ms
64 bytes from 10.10.10.2: icmp_seq=2 ttl=64 time=0.052 ms
64 bytes from 10.10.10.2: icmp_seq=3 ttl=64 time=0.044 ms
64 bytes from 10.10.10.2: icmp_seq=4 ttl=64 time=0.042 ms
64 bytes from 10.10.10.2: icmp_seq=5 ttl=64 time=0.056 ms

--- myservice ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4001ms
rtt min/avg/max/mdev = 0.020/0.042/0.056/0.015 ms
```

The output clearly shows that the container can ping the `myservice` service by name. Notice that the IP address returned is `10.10.10.2`. In the next few steps we'll verify that this address is the virtual IP (VIP) assigned to the `myservice` service.

Type the `exit` command to leave the `exec` container session and return to the shell prompt of your **node0** Docker host.

```
root@d676496d18f7:/# exit
```

Inspect the configuration of the "myservice" service by running `docker service inspect myservice`. Lets verify that the VIP value matches the value returned by the previous `ping -c5 myservice` command.

```
$ docker service inspect myservice
[
    {
        "ID": "ov30itv6t2n7axy2goqbfqt5e",
        "Version": {
            "Index": 19
        },
        "CreatedAt": "2017-04-04T09:35:47.009730798Z",
        "UpdatedAt": "2017-04-04T09:35:47.05475096Z",
        "Spec": {
            "Name": "myservice",
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "ubuntu:latest@sha256:dd7808d8792c9841d0b460122f1acf0a2dd1f56404f8d1e56298048885e45535",
                    "Args": [
                        "sleep",
                        "infinity"
                    ],
<Snip>
        "Endpoint": {
            "Spec": {
                "Mode": "vip"
            },
            "VirtualIPs": [
                {
                    "NetworkID": "wlqnvajmmzskn84bqbdi1ytuy",
                    "Addr": "10.10.10.2/24"
                }
            ]
        },
<Snip>
```

Towards the bottom of the output you will see the VIP of the service listed. The VIP in the output above is `10.10.10.2` but the value may be different in your setup. The important point to note is that the VIP listed here matches the value returned by the `ping -c5 myservice` command.

Feel free to create a new `docker exec` session to the service task (container) running on **node1** and perform the same `ping -c5 service` command. You will get a response form the same VIP.

## <a name="routingmesh"></a>Step 6: Test Routing Mesh

Now let's create a service that utilizes Routing Mesh and the ingress network. Here you'll be creating a single task service that exposes port 5000 on the ingress network.

```
docker service create -p 5000:5000 --name pets --replicas=1 nicolaka/pets_web:1.0
```

Check which nodes did the task run.

```
ubuntu@node-0:~$ docker service ps pets
ID            NAME    IMAGE                  NODE    DESIRED STATE  CURRENT STATE          ERROR  PORTS
sqaa61qcepuh  pets.1  nicolaka/pets_web:1.0  node-0  Running        Running 4 minutes ago
```

You can see that the task is running on `node-0`, it could be `node-1` in your case. Regardless which node the task is running on, routing mesh make sure that you can connect to port `5000` on all cluster nodes and it will take care of forwarding the traffic to a healthy task. 

Using your browser, go to the node where the task is NOT running on ( e.g `52.23.23.1:5000` where `52.23.23.1` is the IP of the node that the task is NOT running on).

You still can see the app right? That's the power of Routing Mesh!


## Wrap Up

Thank you for taking the time to complete this lab! Feel free to try any of the other labs.
