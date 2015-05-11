
Running GUI apps in a Docker Container
======================================
There are a few different options to run GUI applications inside a Docker container like using SSH with X11 forwarding, or VNC but the simplest one that I figured out was to share my X11 socket with the container and use it directly.

The idea is pretty simple and you can easily it give a try by running a Firefox container using the following Dockerfile as a starting point:

```shell
FROM ubuntu:14.04

RUN apt-get update && apt-get install -y firefox

# Replace 1000 with your user / group id
RUN export uid=1000 gid=1000 && \
    mkdir -p /home/developer && \
    echo "developer:x:${uid}:${gid}:Developer,,,:/home/developer:/bin/bash" >> /etc/passwd && \
    echo "developer:x:${uid}:" >> /etc/group && \
    echo "developer ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/developer && \
    chmod 0440 /etc/sudoers.d/developer && \
    chown ${uid}:${gid} -R /home/developer

USER developer
ENV HOME /home/developer
CMD /usr/bin/firefox
```


> Goal is to be able to run GUI apps running inside a docker container. Eventually we will create a complete
> java development environment inside a container.

###Docker Basics

###Docker tips

**Getting the id of the last-run container You could do this:**
```shell 
  $ ID=$(docker run ubuntu echo hello world)
  hello world
$ docker commit $ID hello world
fd08a884dc79
```
But you have to keep assigning IDs. Try this instead:
```shell
$ alias dl='docker ps -l -q'
$ docker run ubuntu echo hello world
hello world
$ dl
1904cf045887
$ docker commit `dl` hello world
fd08a884dc79
```
**Why you always want a Docker file, even if you install everything in the shell?**

Docker commit commands can get unwieldy:
```shell 
$ docker run -i -t ubuntu bash
root@db0c3978abf8:/# apt-get install postgresql -y
root@db0c3978abf8:/# exit
$ docker commit -run='{"Cmd":["postgres", "-too -many -opts"]}' `dl` postgres # ugly long command, easier to do within a Dockerfile
5061b88e4cf0
```
Instead, make a wee little Dockerfile that is FROM the image you made interactively. There you can set CMD, ENTRYPOINT, VOLUME, etc.

**How to run Docker without root ?**
```shell
 $ sudo groupadd docker
 $ sudo gpasswd -a myusername docker
 $ sudo service docker restart
 $ exit
```
**How to delete all running containers?**
```shell
 $ docker rm $(docker ps -a -q)
```
**Easy way to parse Docker's inspect command.**
Here is how to grab the IP address in one line of unix:
```shell
$ docker inspect `dl` | grep IPAddress | cut -d '"' -f 4
$ 172.17.0.52
```
**What environments variables does an image have?**
This is especially nice when using docker run --link to connect containers:
```shell
$ docker run ubuntu env
```
 **Difference between RUN and CMD in the Docker file:**

**Does a Docker container have its own IP address?**
Yes, it is like a process with an internal IP address
```shell
  $ ip -4 -o addr show eth0

  $ $ docker run ubuntu ip -4 -o addr show eth0
```
###Setup Phases 1 & 2 
```shell
$ google-chrome --no-sandbox &
$ ssh -X docker@192.168.33.10 -p 49153

```

###Conclusion


###References

 1. [Panamax](http://panamax.io/) - Docker management tools
 2. [Docker desktop project](https://github.com/rogaha/docker-desktop) 

