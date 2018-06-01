# Using Docker EE in Play with Docker

You'll have your own online lab environment at the DockerCon workshop which has [Docker EE](https://www.docker.com/enterprise-edition) already configured, in a multi-node cluster with swarm mode and Kubernetes.

The lab environment is based on [Play with Docker]() (PWD).

## Navigating the PWD Environment

This is the homepage for the lab environment:

![](img/pwd/pwd-home.jpg)

There are three main components to the Play with Docker interface:

- the **left hand navigation** has links to open Universal Control Plane (UCP), and Docker Trusted Registry (DTR). You can also connect the console window to any of the nodes in the cluster.

- the **console window** at the top connects a terminal to one of your cluster nodes. You can type commands in here, use Ctrl-R to search the command history, and copy and paste commands from the workshop.

- the **session information** at the bottom shows you the public URLs where you can access UCP and DTR outside of the lab environment, and also shows the login credentials for your session.

### 1.1 - Clone the lab repo on a worker node

Play with Docker provides access to a multi-node Docker EE cluster:

* A Linux-based Docker EE 18.01 manager node
* Three Linux-based Docker EE 18.01 worker nodes

By clicking a node name on the left, the console window will be connected to that node:

![](/img/pwd/pwd-console.jpg)

Start by connecting to `worker3` and cloning the source for this workshop:

```
mkdir scm && cd scm

https://github.com/sixeyed/docker-networking-workshop.git

cd docker-networking-workshop
```

### 3.3 - Access to Universal Control Plane (UCP) and Docker Trusted Registry (DTR)

PWD also has links to the Universal Control Plane (UCP) management UI as well as the Docker Trusted Registry (DTR) UI. Clicking on either the `UCP` or `DTR` button will bring up the respective server web interface in a new tab.

Click on the UCP link to open Universal Control Plane. The lab uses self-signed HTTPS certificates, so you'll see a connection warning which you can ignore (add an exception to the site in your browser).

Log in to UCP using the username and password from your PWD session information:

![](/img/pwd/ucp-login.jpg)

You'll see the UCP homepage where you can navigate around the cluster. Now switch back to the PWD session and click the link to open DTR. DTR shares authentication with UCP, so you are already logged into DTR with the same user credentials.

## <a name="3"></a> 4 - Manage Kubernetes on Docker EE from Docker for Mac and Docker for Windows

You can remotely manage your Docker EE cluster by downloading a client bundle from UCP. 

Communication between your client and UCP is secured with mutual TLS, and the client bundle contains certificates and setup scripts. Your client certificate identifies your account and enforces access control with the same permissions as in the UI.

In UCP click your username in the left nav and select _My Profile_:

![](/img/pwd/ucp-user-profile.jpg)

Click _New Client Bundle_ and then _Generate Bundle_ to create a client bundle which gets downloaded to your machine:

![](/img/pwd/ucp-user-profile-2.jpg)

On your laptop, expand the ZIP file, open a terminal window and browse to the client bundle folder. You'll see a set of certificate and script files:

```
$ pwd
/Users/elton/Downloads/ucp-bundle-admin
$ tree
.
├── ca.pem
├── cert.pem
├── cert.pub
├── env.cmd
├── env.ps1
├── env.sh
├── key.pem
└── kube.yml

0 directories, 8 files
```

Check the contents of the `env.sh` and `kube.yml` files and you'll see that they set the configuration for the Docker CLI and `kubectl` to securely connect to your UCP instance:

```
cat env.sh

cat kube.yml
```

Run the environment script, and your local CLI is now connected to the remote UCP instance - for both `docker` and `kubectl` commands:

```
eval "$(<env.sh)"
```

> If you're using Docker for Windows, you can do this in a PowerShell session, running `. ./env.ps1`

You can run all the usual Kubernetes commands to work with resources in UCP:

```
$ kubectl get deployment
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
database     1         1         1            1           26m
dotnet-api   1         1         1            1           15m
java-app     1         1         1            1           26m
```

And you can use `docker stack deploy` to deploy and upgrade applications on the Docker EE cluster.

## Up Next

Now you're ready to move onto [networking in Docker swarm](swarm.md). You'll learn how networking works in Docker containers and in Docker swarm mode.