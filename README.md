# DockerCompleteReference By Mithun Ashok

## Installation

### Linux

Install script provided by Docker:

```
curl -sSL https://get.docker.com/ | sh
```

Or see [Installation](https://docs.docker.com/engine/installation/linux/) instructions for your Linux distribution.


### Mac OS X

Download and install [Docker For Mac](https://docs.docker.com/docker-for-mac/)

### Create Docker VM with Docker Machine

You can use [Docker Machine](https://docs.docker.com/machine/) to:
- Install and run Docker on Mac or Windows
- Provision and manage multiple remote Docker hosts
- Provision Swarm clusters

A simple example to create a local Docker VM with VirtualBox: 

```
docker-machine create --driver=virtualbox default
docker-machine ls
eval "$(docker-machine env default)"
```

Then start up a container:

```
docker run alpine echo "hello-world"
```

That's it, you have a running Docker container.


## Container lifecycle

* Create a container: [`docker create imageName`](https://docs.docker.com/engine/reference/commandline/create).  
* Create and start a container in one operation: [`docker run imageName`](https://docs.docker.com/engine/reference/commandline/run)
  -  Remove the container after it stops `--rm`: `docker run  --rm alpine ls /usr/lib`
  -  Attach the container stdin/stdout to the current terminal use `-it`: `docker run -it ubuntu bash`
  -  To mount a directory on the host to a container add `-v` option, e.g. `docker run -v $HOSTDIR:$DOCKERDIR imageName` 
  -  To map a container port use `-p $HOSTPORT:$CONTAINERPORT`: `docker run -p 8080:80 nginx`
  -  To run the container in background use `-d` switch: `docker run -d -p 8080:80 nginx`
  -  Set container name: `docker run --name myContainerName imageName` 
  -  Example to run nginx web server to serve files from html directory on port 8080: 

```
    docker run -d -v $(pwd)/html:/usr/share/nginx/html -p 8080:80 --name myNginx nginx
    # access the webserver
    curl http://localhost:8080
``` 
* In some cases containers need [extended privileges](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities). Add privileges with the `--cap_add` switch, e.g. `docker run --cap-add SYS_ADMIN imageName`. The flag `--cap_drop` is used to remove privileges. 
* Rename a container: [`docker rename name newName`](https://docs.docker.com/engine/reference/commandline/rename/) 
* Delete a container: [`docker rm containerID`](https://docs.docker.com/engine/reference/commandline/rm)
  - Delete all unused containers: `docker ps -q -a | xargs docker rm`
  - Remove the volumes associated with a container: `docker rm -v containerName`
 
* Update container resource limits:
[`docker update --cpu-shares 512 -m 300M`](https://docs.docker.com/engine/reference/commandline/update/)  
* List running containers: [`docker ps`](https://docs.docker.com/engine/reference/commandline/ps/)
* List all containers: [`docker ps -a`](https://docs.docker.com/engine/reference/commandline/ps/)
* List all container IDs: [`docker ps -q -a`](https://docs.docker.com/engine/reference/commandline/ps/)

##### Issues with docker update for memory
You may get into issues while executing "docker update --cpu-shares 512 -m 300M" like below.
![Memorylimit](/images/memorylimit.jpg)

""Error response from daemon: Cannot update container c3e36910c1c9a6cb1496ab16021eab08d61437dfaf31b97588f64838758b4f2a: Memory limit should be smaller than already set memoryswap limit, update the memoryswap at the same time""

To solve this issue execute the command below,

![updatememory](/images/updatememory.jpg)

docker update --cpu-shares 512 -m 10M --memory-swap -1 <containername/id>


#### CPU Constraints

You can limit CPU, either using a percentage of all CPUs, or by using specific cores.  

For example, you can tell the [`cpu-shares`](https://docs.docker.com/engine/reference/run/#/cpu-share-constraint) setting.  The setting is a bit strange -- 1024 means 100% of the CPU, so if you want the container to take 50% of all CPU cores, you should specify 512.  See <https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/#_cpu> for more:

```
docker run -it -c 512 agileek/cpuset-test
```

You can also only use some CPU cores using [`cpuset-cpus`](https://docs.docker.com/engine/reference/run/#/cpuset-constraint).  See <https://agileek.github.io/docker/2014/08/06/docker-cpuset/> for details and some nice videos:

```
docker run -it --cpuset-cpus=0,4,6 agileek/cpuset-test
```

Note that Docker can still **see** all of the CPUs inside the container -- it just isn't using all of them.  See <https://github.com/docker/docker/issues/20770> for more details.

#### Memory Constraints

You can also set [memory constraints](https://docs.docker.com/engine/reference/run/#/user-memory-constraints) on Docker:

```
docker run -it -m 300M ubuntu:14.04 /bin/bash
```

##### By default it is all ulimitied for both CPU and Memory
It is usually ulimited resources allocated to each of the containers when it gets created unless otherwise configured. 
You can refer to [official Docker Documentation](https://docs.docker.com/config/containers/resource_constraints/) for the same. 

![CpuMemoryDefaults](/images/cpumemorydefaults.jpg)

## Starting and stopping containers

* Start a container:
[`docker start containerName`](https://docs.docker.com/engine/reference/commandline/start) * [`docker stop`](https://docs.docker.com/engine/reference/commandline/stop) 
* Restart a container: [`docker restart containerName`](https://docs.docker.com/engine/reference/commandline/restart)
* Pause a container ("freeze"): [`docker pause containerName`](https://docs.docker.com/engine/reference/commandline/pause/)
* Unpause a container: [`docker unpause containerName `](https://docs.docker.com/engine/reference/commandline/unpause/) 
* Stop and wait for termination: [`docker wait containerName`](https://docs.docker.com/engine/reference/commandline/wait) 
* Kill a container (sends SIGKILL): [`docker kill containerName`](https://docs.docker.com/engine/reference/commandline/kill) 

## Executing commands in containers and apply changes

* Execute a command in a container: [`docker exec -it containerName command`](https://docs.docker.com/engine/reference/commandline/exec) 
* Copy files or folders between a container and the local filesystem: [`docker cp containerName:path localFile`](https://docs.docker.com/engine/reference/commandline/cp) or `docker cp localPath containerName:path`
* Apply changes in container file systems: [`docker commit containerName`](https://docs.docker.com/engine/reference/commandline/commit)


## Logging and monitoring

Logging on Docker could be challenging - check [Top 10 Docker Logging Gotchas](https://sematext.com/blog/top-10-docker-logging-gotchas/).

* Show container logs: [`docker logs containerName`](https://docs.docker.com/engine/reference/commandline/logs)
* Show only new logs: [`docker logs -f containerName`](https://docs.docker.com/engine/reference/commandline/logs)
* Show CPU and memory usage: [`docker stats`](https://docs.docker.com/engine/reference/commandline/stats)
* Show CPU and memory usage for specific containers: [`docker stats containerName1 containerName2`](https://docs.docker.com/engine/reference/commandline/stats)
* Show running processes in a container: [`docker top containerName`](https://docs.docker.com/engine/reference/commandline/top) 
* Show Docker events: [`docker events`](https://docs.docker.com/engine/reference/commandline/events) 
* Show storage usage: [`docker system df`](https://docs.docker.com/engine/reference/commandline/system_df) 
* To run a container with a custom [logging driver](https://docs.docker.com/engine/admin/logging/overview/) (i.e., to syslog), use: 
```
docker run -–log-driver syslog –-log-opt syslog-address=udp://syslog-server:514 \
alpine echo hello world
```
* Use [Logagent](https://sematext.com/logagent/) for log collection and [Sematext Agent](https://www.sematext.com/docker) for metrics and event monitoring. Create Docker Monitoring App & Logs App in [Sematext Cloud](https://sematext.com/cloud) to get required tokens and use them instead the placeholders `YourContainerToken`, `YourInfraToken` and `YourLogsToken` in commands below. Start collecting all container metrics, host metrics and Docker events:

```
docker run -d  --restart always --privileged -P --name st-agent \
-v /sys/kernel/debug:/sys/kernel/debug \
-v /var/run/:/var/run/ \
-v /proc:/host/proc:ro \
-v /etc:/host/etc:ro \
-v /sys:/host/sys:ro \
-v /usr/lib:/host/usr/lib:ro \
-e CONTAINER_TOKEN=YourContainerToken \
-e INFRA_TOKEN=YourInfraToken \
-e JOURNAL_DIR=/var/run/st-agent \
-e LOGGING_WRITE_EVENTS=false \
-e LOGGING_REQUEST_TRACKING=false \
-e LOGGING_LEVEL=info \
-e NODE_NAME=`hostname` \
-e CONTAINER_SKIP_BY_IMAGE=sematext \
sematext/agent:latest
```

Collect all container logs:

```
docker run -d --name st-logagent \
-e LOGS_TOKEN=YourLogsToken \
-v /var/run/docker.sock:/var/run/docker.sock \
sematext/logagent
```

The commands above will collect all container metrics, host metrics, Docker events, and container logs. 

### Monitoring/Removing Images

| task | command |
|:-----|:--------|
| list images | `docker images` |
| remove specific image(s) | `docker rmi <image_id> <image_id> ...` |
| remove dangling images | `docker image prune` |
| remove all unused images | `docker image prune --all` |
| remove all images | `docker rmi -f $(docker images -q)` |

### Monitoring/Killing Containers

| task | command | notes|
|:-----|:--------|:-----|
| see running containers | `docker ps` | like `ps` in bash! |
| see most recently launched container | `docker ps -l` | `-l` for last |
| see all containers | `docker ps -a` | `-a` for all |
| see hash of running containers | `docker ps -q` | `-q` for hash |
| see hash of most recent container | `docker ps -ql` | mix `-q` and `-l` for hash of last |
| see hash of all containers | `docker ps -aq` | ditto for `-a` and `-q` |
| see running processes | `docker top id container` | use `id container` |
| kill all running containers | `docker kill $(docker ps -q)` | `kill` only stops running containers |
| kill most recent container | `docker kill $(docker ps -ql)` |
| remove all exited containers | `docker container prune` |
| remove all containers | `docker rm -f $(docker ps -qa)` | `-f` forces `rm` to kill and remove |
| remove old containers | `docker ps -a \| grep 'weeks ago' \| awk '{print $1}' \| xargs docker rm` | removes containers that are created weeks ago |

## Exploring Docker information

* Show Docker info: [`docker info`](https://docs.docker.com/engine/reference/commandline/info)
* List all containers: [`docker ps -a`](https://docs.docker.com/engine/reference/commandline/ps) 
* List all images: [`docker image ls`](https://docs.docker.com/engine/reference/commandline/image_ls) 
* Show all container details: [`docker inspect containerName`](https://docs.docker.com/engine/reference/commandline/inspect) 
* Show changes in the container's files:[`docker diff containerName`](https://docs.docker.com/engine/reference/commandline/diff) 


## Manage Docker images

* List all images: [`docker images`](https://docs.docker.com/engine/reference/commandline/images) 
* Search in registry for an image:
[`docker search searchTerm`](https://docs.docker.com/engine/reference/commandline/search) 
* Pull image from a registry: [`docker pull imageName`](https://docs.docker.com/engine/reference/commandline/pull) pulls an image from registry to local machine
* Create image from Dockerfile: [`docker build`](https://docs.docker.com/engine/reference/commandline/build)
* Remove image: [`docker rmi imageName`](https://docs.docker.com/engine/reference/commandline/rmi) 
* Export container into tgz file: [`docker export myContainerName -o myContainerName `](https://docs.docker.com/engine/reference/commandline/export) 
* Create an image from a tgz file:[`docker import file`](https://docs.docker.com/engine/reference/commandline/import) 

## Docker networks

* List existing networks: [`docker network ls`](https://docs.docker.com/engine/reference/commandline/network_ls/)
* Create a network: [`docker network create netName`](https://docs.docker.com/engine/reference/commandline/network_create/)
* Remove network: [`docker network rm netName`](https://docs.docker.com/engine/reference/commandline/network_rm/)
* Show network details: [`docker network inspect`](https://docs.docker.com/engine/reference/commandline/network_inspect/)
* Connect container to a network: [`docker network connect networkName containerName`](https://docs.docker.com/engine/reference/commandline/network_connect/)
* Disconnect network from container: [`docker network disconnect networkName containerName`](https://docs.docker.com/engine/reference/commandline/network_disconnect/)

## Data cleanup 

Data management commands:

* Remove unused resources: [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/)
* Remove unused volumes: [`docker volume prune`](https://docs.docker.com/engine/reference/commandline/volume_prune/)
* Remove unused networks: [`docker network prune`](https://docs.docker.com/engine/reference/commandline/network_prune/)
* Remove unused containers: [`docker container prune`](https://docs.docker.com/engine/reference/commandline/network_prune/)
* Remove unused images: [`docker image prune`](https://docs.docker.com/engine/reference/commandline/network_prune/)


## Security

This is where security tips about Docker go. The Docker [security](https://docs.docker.com/engine/security/security/) page goes into more detail.

First things first: Docker runs as root. If you are in the `docker` group, you effectively [have root access](https://web.archive.org/web/20161226211755/http://reventlov.com/advisories/using-the-docker-command-to-root-the-host). If you expose the docker unix socket to a container, you are giving the container [root access to the host](https://www.lvh.io/posts/dont-expose-the-docker-socket-not-even-to-a-container/).

Docker should not be your only defense. You should secure and harden it.

For an understanding of what containers leave exposed, you should read [Understanding and Hardening Linux Containers](https://www.nccgroup.trust/globalassets/our-research/us/whitepapers/2016/april/ncc_group_understanding_hardening_linux_containers-1-1.pdf) by [Aaron Grattafiori](https://twitter.com/dyn___). This is a complete and comprehensive guide to the issues involved with containers, with a plethora of links and footnotes leading on to yet more useful content. The security tips following are useful if you've already hardened containers in the past, but are not a substitute for understanding.

### Security Tips

For greatest security, you want to run Docker inside a virtual machine. This is straight from the Docker Security Team Lead -- [slides](http://www.slideshare.net/jpetazzo/linux-containers-lxc-docker-and-security) / [notes](http://www.projectatomic.io/blog/2014/08/is-it-safe-a-look-at-docker-and-security-from-linuxcon/). Then, run with AppArmor / seccomp / SELinux / grsec etc to [limit the container permissions](http://linux-audit.com/docker-security-best-practices-for-your-vessel-and-containers/). See the [Docker 1.10 security features](https://blog.docker.com/2016/02/docker-engine-1-10-security/) for more details.

Docker image ids are [sensitive information](https://medium.com/@quayio/your-docker-image-ids-are-secrets-and-its-time-you-treated-them-that-way-f55e9f14c1a4) and should not be exposed to the outside world. Treat them like passwords.

See the [Docker Security Cheat Sheet](https://github.com/konstruktoid/Docker/blob/master/Security/CheatSheet.adoc) by [Thomas Sjögren](https://github.com/konstruktoid): some good stuff about container hardening in there.

Check out the [docker bench security script](https://github.com/docker/docker-bench-security), download the [white papers](https://blog.docker.com/2015/05/understanding-docker-security-and-best-practices/).

Snyk's [10 Docker Image Security Best Practices cheat sheet](https://snyk.io/blog/10-docker-image-security-best-practices/)

You should start off by using a kernel with unstable patches for grsecurity / pax compiled in, such as [Alpine Linux](https://en.wikipedia.org/wiki/Alpine_Linux). If you are using grsecurity in production, you should spring for [commercial support](https://grsecurity.net/business_support.php) for the [stable patches](https://grsecurity.net/announce.php), same as you would do for RedHat. It's $200 a month, which is nothing to your devops budget.

Since docker 1.11 you can easily limit the number of active processes running inside a container to prevent fork bombs. This requires a linux kernel >= 4.3 with CGROUP_PIDS=y to be in the kernel configuration.

```
docker run --pids-limit=64
```

Also available since docker 1.11 is the ability to prevent processes from gaining new privileges. This feature have been in the linux kernel since version 3.5. You can read more about it in [this](http://www.projectatomic.io/blog/2016/03/no-new-privs-docker/) blog post.

```
docker run --security-opt=no-new-privileges
```

From the [Docker Security Cheat Sheet](http://container-solutions.com/content/uploads/2015/06/15.06.15_DockerCheatSheet_A2.pdf) (it's in PDF which makes it hard to use, so copying below) by [Container Solutions](http://container-solutions.com/is-docker-safe-for-production/):

Turn off interprocess communication with:

```
docker -d --icc=false --iptables
```

Set the container to be read-only:

```
docker run --read-only
```

Verify images with a hashsum:

```
docker pull debian@sha256:a25306f3850e1bd44541976aa7b5fd0a29be
```

Set volumes to be read only:

```
docker run -v $(pwd)/secrets:/secrets:ro debian
```

Define and run a user in your Dockerfile so you don't run as root inside the container:

```
RUN groupadd -r user && useradd -r -g user user
USER user
```

### User Namespaces

There's also work on [user namespaces](https://s3hh.wordpress.com/2013/07/19/creating-and-using-containers-without-privilege/) -- it is in 1.10 but is not enabled by default.

To enable user namespaces ("remap the userns") in Ubuntu 15.10, [follow the blog example](https://raesene.github.io/blog/2016/02/04/Docker-User-Namespaces/).

### Security Videos

* [Using Docker Safely](https://youtu.be/04LOuMgNj9U)
* [Securing your applications using Docker](https://youtu.be/KmxOXmPhZbk)
* [Container security: Do containers actually contain?](https://youtu.be/a9lE9Urr6AQ)
* [Linux Containers: Future or Fantasy?](https://www.youtube.com/watch?v=iN6QbszB1R8)

### Security Roadmap

The Docker roadmap talks about [seccomp support](https://github.com/docker/docker/blob/master/ROADMAP.md#11-security).
There is an AppArmor policy generator called [bane](https://github.com/jfrazelle/bane), and they're working on [security profiles](https://github.com/docker/docker/issues/17142).

## Guidelines for building secure Docker images

1. Prefer minimal base images
2. Dedicated user on the image as the least privileged user
3. Sign and verify images to mitigate MITM attacks
4. Find, fix and monitor for open source vulnerabilities
5. Don’t leak sensitive information to docker images
6. Use fixed tags for immutability
7. Use COPY instead of ADD
8. Use labels for metadata
9. Use multi-stage builds for small secure images
10. Use a linter


### Run MongoDB

#### Run MongoDB Using Named Volume

To run a new MongoDB container, execute the following command from the CLI:

```docker
docker run --rm --name mongo-dev -v mongo-dev-db:/data/db -d mongo
```

CLI Command | Description
--- | ---
--rm | remove container when stopped
--name mongo-dev | give container a custom name
-v mongo-dev-db/data/db | map the container volume 'data/db' to a custom name 'mongo-dev-db'
-d mongo | run mongo container as a daemon in the background

#### Run MongoDB Using Bind Mount

```bash
cd
mkdir -p mongodb/data/db
docker run --rm --name mongo-dev -v ~/mongodb/data/db:/data/db -d mongo
```

CLI Command | Description
--- | ---
--rm | remove container when stopped
--name mongo-dev | give container a custom name
-v ~/mongodb/data/db/data/db | map the container volume 'data/db' to a bind mount '~/mongodb/data/db'
-d mongo | run mongo container as a daemon in the background

### Access MongoDB

#### Connect to MongoDb

There are 2 steps to accessing the MongoDB shell.

1. Firstly, access the MongoDB container shell by executing the following command:

   ```bash
   docker exec -it mongo-dev bash
   ```

   This will open an interactive shell (bash) on the MongoDB container.

1. Secondly, once inside the container shell, access the MongoDB shell by executing the following command:

   ```bash
   mongo localhost
   ```

#### Show Databases

Once connected to MongoDB shell, run the following command to show a list of databases:

```bash
show dbs
```

#### Create New Database

Create a new database and collection

```javascript
use test
db.messages.insert({"message": "Hello World!"})
db.messages.find()
```

---
# Important Commands

### How to start Docker service automatically
#### Option1
You can refer to the [link](https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot) from Docker Documentation and the same is available below.

![DockerStartonBoot](/images/dockeronboot.jpg)

#### Option2
You can use this command to enable automatic start of the docker service after startup.

```linuxcommand
systemctl enable /usr/lib/systemd/system/docker.service
```

### How to change default image directory in Linux
#### Option1
You can refer to [IBM Documentation](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.2.x/installing/docker_dir.html) to make this change at Linux level.

![IBMChangeImageDirforDocker](/images/ibmchangeimagedir.jpg)

#### Option2

With recent versions of Docker, you would set the value of the data-root parameter to your custom path, in /etc/docker/daemon.json according to [Docker Documentation](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file).

![DockerConfigFile](/images/dockerconfigfile.jpg)

```text
By default daemon.json does not exist, if you want to specify some configuration then go ahead and create the file. 
The default config file path on Linux is /etc/docker/daemon.json like you said, but it doesn't exist by default. You can write one yourself and put additional docker daemon configuration stuff in there instead of passing in those configuration options into the command line. You don't even have to do dockerd --config-file /etc/docker/daemon.json since that's the default path, but it can be useful to make it explicit for others who are inspecting the system.

Also ensure that any configuration you set in /etc/docker/daemon.json doesn't conflict with options passed into the command line evocation of dockerd.
```

With older versions, you can change Docker's storage base directory (where container and images go) using the -goption when starting the Docker daemon. (check docker --help). You can have this setting applied automatically when Docker starts by adding it to /etc/default/docker


### How to change the configuration of an already existing container

You can change the configuration of an existing container by directly editing the hostconfig.json file at /var/lib/docker/containers/[hash_of_the_container]/hostconfig.json or /var/snap/docker/common/var-lib-docker/containers/[hash_of_the_container]/hostconfig.json, I believe, if You installed Docker as a snap.

You can determine the [hash_of_the_container] via the docker inspect <container_name> command and the value of the "Id" field is the hash.

Stop the container (docker stop <container_name>).
Stop docker service (per Tacsiazuma's comment)
Change the file.
Restart your docker engine (to flush/clear config caches).
Start the container (docker start <container_name>).
So you don't need to create an image with this approach. You can also change the restart flag here.
