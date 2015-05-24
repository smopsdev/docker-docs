Building a Bitnami Tomcat Image using Docker
=============================================

####Step 0 - Create a VM and install Docker
I did this in a single step using Digital Ocean's ability to select OS / application combos - in this case Docker 1.3.1 on Ubuntu 14.04 (64 bit). To keep Bitnami's installer from complaining about memory (in Step 4) you are going to need at least a 2 GB VM. If you want to run multiple stacks side-by-side on the same VM, you are going to need at least 4GB.

#####Step 1 - Download the Bitnami Tomcat Installer onto Your VM
The easiest way to do this is use 'wget' on the VM:
```bash
root@vm:~# mkdir bitnami; cd bitnami
root@vm:~# wget https://bitnami.com/redirect/to/45854/bitnami-tomcatstack-7.0.62-0-linux-x64-installer.run
root@vm:~# chmod +x *.run
```

Also note the I've saved the installer under a new directory (which we will reference in Step 3) 
and made it executable.

#####Step 2 - Download/Pull the Base Docker Image
Working with Docker is like baking sourdough bread; you need a little something to start with. 
I chose to use Docker's base Ubuntu image because (a) I really don't care which OS I'm running Tomcat on, 
and (b) I've used Bitnami's Tomcat stack on Ubuntu before and never had any problems.

```bash
root@vm:~# docker pull ubuntu
```
You should see a brief flurry of activity ending with:
Status: Downloaded newer image for ubuntu:latest

#####Step 3 - Start a Container
First I'll show you the command, then I'll explain the options:

```bash
root@vm:~# docker run --cap-add=ALL -i -p 80:80 -t -v /root/bitnami:/bitnami ubuntu /bin/bash
```

--cap-add=ALL: When it starts, Tomcat tries to set some capabilities (i.e. establish the privilege 
to do one or more "superuser like" things). By default Docker does not allow processes within a container
to do this. This option allows processes within the container to set any capability they want. This is a 
sloppy and dangerous thing to do. I should dig into the Tomcat code and figure out exactly which 
capabilities it is requesting and grant only those capabilities (see the "principle of least privilege").
-v /root/bitnami:/bitnami: This option bind mounts "/root/bitnami" on the VM to "/bitnami" in the container. 
This will allow us to access the installer file from inside the container.

-p 80:80: By default the Apache web server listens on port 80. This option maps port 80 of the container
to port 80 on our VM. Obviously you can map the container port to any free port on your VM 
(e.g 8080 using "-p 8080:80").

-i, -t: These two options connect you to the shell running inside the container.

ubuntu: This option specifies the image to run in the container. In this case it is the default Ubuntu 
image that we pulled in Step 2.

/bin/bash: This option tells Docker to run a bash shell inside the container.

At this point you should find yourself at a container-level prompt like:

root@d10f70897ce3:/# 


#####Step 4 - Run the Bitnami Installer
Next we want to run the Tomcat installer to install Apache, Tomcat, and MySQL into our container:

/bitnami/bitnami-tomcatstack-7.0.57-0-linux-x64-installer.run --mode unattended

This command will take a couple of minutes to complete, so be patient. If all goes well you should return to the container-level prompt where you can poke around a bit to check things out. A "ps -ef" should show you the Apache, MySQL, and Tomcat processes, there should be an "/opt/tomcatstack-7.0.57-0 directory", etc. You can test whether apache is up and accessible by browsing to "http://<your VM address>/". You should see the welcome page for the Bitnami Tomcat stack.

Note that the way in which we installed Apache, MySQL, and Tomcat is extremely unsafe. For example, there is no password for the Tomcat manager application. Under this configuration it should only be a matter of minutes before someone installs something unpleasant onto Tomcat. The Bitnami installer supports a number of command-line options for setting the MySQL password, the Tomcat manager password, etc. You can play around with these to get the configuration you want. This is where Docker shines; you can quickly re-run Steps 3 and 4 to experiment with different configurations. One thing to be aware of is that Docker saves containers after you exit them so, to avoid confusion, you should probably "docker rm <container-id>" on any containers you are no longer interested in.

#####Step 5 - Snapshot the Container
Now that you have a container running a configuration of the Tomcat stack that you are happy with, it is time to snapshot that container and create a Docker image. Since we started the Apache, MySQL, and Tomcat processes from the bash shell that we launched on container startup, exiting the shell will cause these processes to terminate. I confess to being somewhat superstitious, however, so I prefer to shut down these processes in the "proper" manner:

root@d10f70897ce3:/# /opt/tomcatstack-7.0.57-0/ctlscript.sh stop

After this completes you can simply exit the bash shell to exit the container and return to your VM-level shell. At this point we can snapshot the container and create a new image using the "docker commit" command like so:

root@vm:~# docker commit -m="Some pithy comment." d10f70897ce3 mybitnami/tomcat:v1

The resulting image should be viewable through the "docker images" command.

#####Step 6 - Launching the Image
Launching our newly created image is simply a matter of starting a container using that image:

root@vm:~# docker run --cap-add=ALL -d -p 80:80 mybitnami/tomcat:v1 /bin/sh -c "/opt/tomcatstack-7.0.57-0/ctlscript.sh start; tail -F /opt/tomcatstack-7.0.57-0/apache-tomcat/logs/catalina-daemon.out"

This looks a little intimidating, so let's break it down. The "--cap-add=ALL" option was covered in Step 3. We still need this because Tomcat still sets the same capabilities. The "-d" option simply tells Docker to run the container in the background. We've eliminated the "-i" and "-t" options because we don't need to interact directly with the container. The "-p 80:80" options specifies the same port mapping and we've eliminated the "-v" option because we no longer need to access any host files from the container. What makes this step look complicated is the in-line shell script at the end. What we are telling Docker to do is run the following commands in a shell:

/opt/tomcatstack-7.0.57-0/ctlscript.sh start
tail -F /opt/tomcatstack-7.0.57-0/apache-tomcat/logs/catalina-daemon.out

Docker will run a shell that executes "ctlscript.sh start" thus starting Apache, MySQL, and Tomcat. It will then run the "tail" command on the main Tomcat log file, blocking on additional writes to this file. What this means is that the shell process that is the parent or grandparent of all the Apache, MySQL, and Tomcat processes will continue to run, thus keeping the whole tree of processes alive.

There are a number of ways we can monitor our container at this point. We can view a top-like display of the processes in the container via:

root@vm:~# docker top <container ID>

We can look at containers STDOUT and STDERR using:

root@vm:~# docker logs <container ID>

#####Step 7 - Stopping the Container
To stop the container running our tomcat stack we can send the SIGTERM signal to the root process of the container (our shell running "tail") via:

root@vm:~# docker stop <container ID>

#####References
[Bitnami DockerImages](http://recursivedigressions.blogspot.com/2014/11/building-bitnami-tomcat-image-using.html)
