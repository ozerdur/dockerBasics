# Security 

## Root User
Containers can access to host files as root if mounted.\
In order to prevent container to access host root files:

    sudo dockerd --userns-remap=default 


## Docker Sock
Mounting **/var/run/docker.sock** into the container is dangerous. Since this socket can be exposed over the network on a specific port -HTTP API. This socket is used to interact Docker Daemon by docker cli client.

- A malicious user can access to container and install docker. 
- Run a container using the sock and gain access to root files on host.
    
    docker -H:///var/run/docker.sock run -it -v /:/test:ro -t alpine sh 

## --privileged flag

When --privileged flag is used with a container, it will give all Linux capabilities to the container.

If an attacker gains access to the container he can take advantage of these capabilities.

cap_sys_admin, cap_sys_ptrace and cap_sys_module are some of the dangerous capabilities to name.

When an attacker gains a shell on the container and if it has cap_sys_module enabled, it is possible to load a kernel module directly onto the host's kernel from within the container.




## Static Analysis 
-   Trivy 
    -   It is a simple vulnerability scanner for containers. 
    -   It is an open-source project
    -   Fits well in CI/CD pipelines, so it can also be used in DevSecOps pipelines
    -   To perform a static analysis against Docker images

    Ex: docker run --rm -v `pwd`:/root/.cache/ aquasec/trivy getcapsule9/shellshock:test


- Docker Bench Security
    -  It is a script that checks for dozens of common best-practises around deploying Docker containers in production
    -  Uses CIS benchmarks
    -  https://github.com/docker/docker-bench-security

    Ex: 

        docker run --rm --net host --pid host --userns host --cap-add audit_control \
        -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
        -v /etc:/etc:ro \
        -v /usr/bin/containerd:/usr/bin/containerd:ro \
        -v /usr/bin/runc:/usr/bin/runc:ro \
        -v /usr/lib/systemd:/usr/lib/systemd:ro \
        -v /var/lib:/var/lib:ro \
        -v /var/run/docker.sock:/var/run/docker.sock:ro \
        --label docker_bench_security \
        docker/docker-bench-security 


## Defense

These are useless with --privileged flag

- AppArmor
  
    AppArmor or Application Armor is a Linux security module \
    Allows us to restrict programs capabilities with apparmor profiles \
    It can be used to protect Docker containers from security threats \
    To use it with Docker, we need to associate an AppArmor security profile with each container

    When we start a container, we must provide a custom AppArmor profile to it and Docker expects to find an AppArmor policy loaded and enforced

    Used to deny file access etc.

    Ex: 
        
        sudo apparmor_parser -r -W <apparmor-profile> 
        docker run -it --security-opt apparmor=<apparmor-profile> alpine

- Seccomp

    Seccomp is another Linux kernel feature \
    This can be used for filtering system calls issued by a program. \
    This acts as a firewall for system calls
    We can write seccomp profiles, to filter what system calls can be run from within the container.

    We need to load these profiles on each container

    
    Ex: 
        
        docker run -it --security-opt seccomp=<seccomp-profile.json> alpine

- Capabilities

    root users in Linux are very special \
    They have superpowers - more privileges \
    If you break all these superpowers into distinct units, they become capabilities. \
    Almost all the special powers associated with the Linux root user are broken down into individual capabilities.

    Being able to break down these permissions allows you to:
    - Have granular control over controlling what root user can do - less powerful.
    - Provide more powers to standard use at a granular level.

    Ex:
    
        #Even if the container runs with root user, the user does not have CHOWN capability

        docker run -it --cap-drop CHOWN alpine 


        #Even if the container runs with root user, the user has only   CHOWN capability

        docker run -it --cap-drop ALL --cap-add chown alpine


### Docker Content Trust

A feature to force the docker client to download only signed images.\
Not guaranteed to be sage as anyone can sign an image and push it to docker hub.\
It is recommended to pull
- official images
- images with Verified Publisher tag
- Docker Certified images
  

export DOCKER_CONTENT_TRUST=1


