Docker Based Development
=======================

###1. Docker Basics

**1.1 Docker Volumes**
What is a volume?
Volumes decouple the life of the data being stored in them from the life of the container that created them. This makes it so you can docker rm my_container and your data will not be removed.
A volume can be created in two ways:
```
     Specifying VOLUME /some/dir in a Dockerfile
     Specifying it as part of your run command as docker run -v /some/dir
```
You can tell docker to remove a volume along with the container:
```docker rm -v my_container```
Sometimes you've already got a directory on your host that you want to use in the container, so the CLI has an extra option for specifying this:
```docker run -v /host/path:/some/path ...  ```

This tells docker to use the specified host path specifically, instead of creating one itself within the docker root, and mount that to the specified path within the container (/some/path above). Note, that this can also be a file instead of a directory. This is commonly referred to as a bind-mount within docker terminology (though technically speaking, all volumes are bind-mounts in the sense of what is actually happening). If the path on the host does not exist, a directory will be automatically be created at the given path.

**Bind-mount** volumes are treated a little differently than a "normal" volume, with the preference of not modifying things on the host that Docker did not itself create:

With a "normal" volume, docker will automatically copy data at the specified volume path (e.g. /some/path, above) into the new directory that was created by docker, with a "bind-mount" volume this does not happen.
When you docker ```rm -v my_container``` a container with "bind-mount" volumes, the "bind-mount" volumes will not be removed.

You can share volumes with another container.
```
   docker run --name my_container -v /some/path ...
   docker run --volumes-from my_container --name my_container2 ...
```

The command above will tell docker to mount the same volumes from the first container into the 2nd container. This effectively allows you to share data between two containers.

If you docker ```rm -v my_container```, *if the 2nd container above still exists, the volumes will not be removed, and indeed will not ever be removed unless you remove the second container with the same docker* 
```rm -v my_container2.```


 **2. Development Environment**
 **3. Development Bases**
 **4. Building and maintaining development containers**
 **5. Tools and scripts**
 **6. Conclusion**

 **7. References**

7.1 [Building good Docker images](http://jonathan.bergknoff.com/journal/building-good-docker-images)

7.2 [Docker Explained](https://www.digitalocean.com/community/tutorials/docker-explained-using-dockerfiles-to-automate-building-of-images)

7.3 [Docker Container42](http://container42.com/)

7.4 [A better Dev/Test Experience with Docker](https://medium.com/aws-activate-startup-blog/a-better-dev-test-experience-docker-and-aws-291da5ab1238)

7.5 [Big cheat sheet](http://bigocheatsheet.com/#comments)

7.6 [A Docker environment in 24 hours](https://blog.relateiq.com/a-docker-dev-environment-in-24-hours-part-1-of-2/) 



 
 
> Written with [StackEdit](https://stackedit.io/).
