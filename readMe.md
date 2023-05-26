# Docker
Contains Docker Daemon/ Rest API, CLI

Linux OS contains Kernel, applications and sometimes a user interface
Kernel is not included in image


## Installation

For Linux:
    We need to install Docker Engine CE \
    if you add your user to docker user group, you will not have to use sudo for docker

        sudo usermod -aG docker <username>

For Mac:
    Docker Desktop For Mac

Mac and Windows installs a hidden Linux machine with Docker Community Edition 

If you have:

    - Windows 10 Enterprise/ Education or higher, it requires hyperV which also requires BIOS-level hardware virtualization support to be enabled in the BIOS settings
  
    - Windows 10 Home Edition requires WSL2 for Docker Desktop,
            - x64 systems or ARM64 systems
            - Enable Microsoft-Windows-Subsystem-Linux
            - Enable VirtualMachinePlatform
            - update linux kernel to 2
            - set default version to 2
            - BIOS-level hardware virtualization support must be enabled in the BIOS settings

        But you can also follow instructions for Windows 7-8.
    - Other versions like Windows 7-8 you can NOT use Docker Desktop for Windows. Instead you need to use Docker Toolbox with a 64 Bit OS after enabling Virtualization from BIOS. Docker Toolbox installs a Oracle Virtual Box Virtualization tool and then installs a linux vm with Docker Community Edition. It will also install some terminals for our access

    

Docker Community Edition 

    - free
    - no support
    - get patches for only 7 months
    - do not use in prod
    - Released in each month

Docker Enterprise Edition

    - not free
    - get support
    - get patches for 2 years
    - use in prod
    - Released quarterly
    - Supported for 1 year



## Union File System
    The files are stored in layers
    The images are not copied into containers. The container just adds a R/W Layer and makes changes in that layer. The original image is used as read only by multiple containers. If we change any file within the image, the file copied to that layer and modified in that layer.


## Volume
    Volumes can be bind as read only like ilkvolume:/deneme3:ro
    if the folder is empty, we will see the files in the volume
    if the folder is not empty while the volume is empty, the files in the folder are copied to the volume
    If the volume is not empty, we will see the files in the volume no matter the folder has files or not

    To create an empty volume:
        docker volume create ilkvolume (It will be created in linux (vm for mac and windows))
        docker volume inspect ilkvolume 

## Bind Mount
    Do not use in production
    Bind your files or folders in host to a container
    We need to give permission to local disc in Mac and Windows in file Sharing


## Network
used with --net option
Port fwd is needed to access to containers from outside. Default is tcp but by adding /udp we can use udp as well

    --publish <host port>:<container port> or - p <host port>:<container port>
    --publish <host port>:<container port> or - p <host port>:<container port>/udp ( for udp)



### Bridge Driver 
Default Network Driver. The container connects to that driver unless told not to do so. The containers in the same bridge can communicate each other
Unlike default bridge driver, with user defined bridges containers can communicate each other with their names. Also we can connect running containers to user defined bridges

    docker network connect <bridge name> <container name>
    docker network disconnect <bridge name> <container name>  
    docker network create [OPTIONS] firstbridge 

    OPTIONS: 
            --driver bridge 
            --subnet=10.10.0.0/16 
            --ip-range=10.10.10.0/24 
            --gateway=10.10.10.10

### Host Driver
No Network Isolation. Containers work as if they are working on host directly

### MacVlan Driver
Containers behave like they have their own physical network adapter with mac address.
No NAT or IP Routing

### None Driver
No connection

### Overlay Driver
Used to make containers on different host work as if they are in the same network
Containers in the same overlay network can communicate each other without any port limit

    SWARM:
    By default Main management layer communicates encrypted, but to encrypt the communication between containers
    we need create the network with --opt option. But it will slow down the communication. For every service a VIP is created for load balancer so that the containers can be reached by their service name.

## Tips

Ctrl + PQ to exit from container shell without stopping it.

docker commit \<container name> \<image name with tag> : to create an image from a running container. Additional steps can be added by -c option.

    Ex: docker commit  -c 'CMD ["java", "application"]'  con1 ozerdur/con:1.1

docker container run -rm \<image name>: with -rm option the container is removed just after it is stopped

docker container run --restart always \<image name>: to restart container if exits. Other options are "no", "on-failure", "unless-stopped"

docker cp \<container name>:\<path> \<target directory>: to copy files from a container even it is stopped

docker run options for resource management to avoid over resource consumption in nodes

    --memory=100m to limit (not limited unless)
    --memory-swap=200m to give swap memory in hd 
    --cpus="1.5" or --cpuset-cpus="<core number>"

Docker logs options

     -t to add time in front of each log
    --until <time> 
    --since <time>
    --tails <number> to see last few lines specified
    -f to continue to watch logs

docker top \<container name> to run ps to see running processes

docker stats \<container name> to see cpu and memory usage


## Image & Registry

Build image:

    docker image build -t <image with tag> -f <docker file path> <build context>

Tag image:

    docker image tag <old image> <new image with tag>

Push image to registry with tag:

    docker image push <image with tag>

To see each step of image

    docker image history <image with tag>

### Dockerfile Commands

**FROM** \<image:tag>: To define the base image. Only must command in Dockerfile

    Ex: FROM ubuntu:18.04

**RUN** \<command>: to run commands

    Ex: RUN apt-get update -y


**WORKDIR** \<directory>: to change directory in image. Creates the directory if does not exist


    Ex: WORKDIR /app

**COPY** \<source> \<target>: copies sources to image


    Ex: COPY /src /app

**EXPOSE** \<port>:  expose port to outside. Just informational does not have an actual effect


    EX: EXPOSE 80/tcp

**ARG** \<parametername>: to use a variable and give its value in image build state. Can not be accessed in container

    EX: ...
        # Can also have default value  ARG VERSION=1.0.0
        ARG VERSION 
        ADD https://www.python.org/ftp/python/${VERSION}/Python-${VERSION}.tgz .
        ...

        docker image build -t argsampleimage --build-arg VERSION=3.8.1 


**HEALTHCHECK** \<command>: By default Docker monitors the first process to decide whether the container works properly or not. But does not check if the process is working properly. By Healtcheck we can check it.s

    Ex: HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1

**LABEL** \<key>=\<value>:  To store key value pair in image's metadata. Not accessible inside container

    Ex: LABEL version="1.0"

**ENV** \<key>=\<value>: to define env variable

    Ex: ENV USER="ozer"

**ADD** \<source> \<target>: copies sources to image similar to CMD command but source can be a url. Also tar files are copied after extraction if it is a local file
    
    Ex: ADD https://wordpress.org/latest.tar.gz /temp


**CMD** \<command>: to specify default command for run time

    EX: CMD ["java", "merhaba"]

**ENTRYPOINT** \<command>: to specify default command for run time but can NOT be changed. If used with CMD, CMD command is processed as parameters to ENTRYPOINT command

    EX: ENTRYPOINT ["java", "merhaba"]


### CMD  Exec Form and Shell Form 
Exec Form: CMD ["java", "application"]
Shell Form: CMD java application

- If shell form is used, the command is run inside default shell as first process
- If exec form is used, no shell is run and the cmd is run directly as first process
- If exec form is used, since no shell is run, we may not access to Env variables 
- If shell form is used with ENTRYPOINT, the CMD commands are not passed as inputs. Therefore if ENTRYPOINT is used exec form should be used 


### Multi Stage Build
To compile source code and copy them to another image. Every Stage starts with FROM and can be tagged using as. Result image just comes from the latest stage.

    EX: FROM <any sdk> as compiler
        COPY <source> <target>
        RUN <compile command>

        FROM <runtime image>
        # to copy files from compiler image
        COPY --from=compiler <source> <target>
        CMD <run command>

### Docker Save Load
To copy an image and load it to an isolated environment

    docker save <image name> -o <target tar file name>.tar
    # After copying the result tar file manually
    docker load -i <tar file path>

 ### Linux Shell

 ">" : to write an output a file
 
    Ex: echo $SHELL > deneme.txt

 "&": to run a command async
    
    Ex: ./longRunningScript.sh &

 "|": to give previous cmd's output to next cmd as an input 

    Ex: cat abc.txt | grep 3 

 ";": to run multiple cmds in the same line

    Ex: ls ; date

 "&&": to run multiple cmds just after each other if the previous one completes successfully

    Ex: cat abc.txt && echo "file found"

 "||": to run multiple cmds just after each other if the previous one fails

    Ex: cat.abc.txt || echo "file not found"

 "\": to run a long single cmd in multiple lines.


### Docker Registry
Private Image Registry Options:

    - docker registry: 
      - Free
      - No RBAC and image scan

    - docker trusted registry

      - Not Free
      - RBAC and image scan

Public Image Registry Options:

    - dockerhub
    - Azure Container Registry


### Docker Compose

Used to create images and run multiple containers in a declarative way. Not used in production. Used for test.
    
    docker-compose up -d
    docker-compose down (deletes all resources except volumes and images)
    docker-compose exec <service name> <command>
    docker-compose build (to regenerate images after update)


### Windows Container
To run containers that are based on Windows. Requires Docker Enterprise Edition and Windows Server 2019/2016( no additional fee for Docker EE) 

Images are based on OS (2016 or 2019)
Base Images

  - Windows Server Core: Support Traditional .Net Framework Applications
  - Nano Server: Built for .Net Core Applications
  - Windows: Provides the full Windows Api set
  - Windows IoT Core: Purpose-built for IoT applications

