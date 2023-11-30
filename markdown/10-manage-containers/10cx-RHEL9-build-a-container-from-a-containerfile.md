# 10c. Build a container from a Containerfile

*This topic includes:*
- Building a pre-written container
- Writing a simple Containerfile
- Buildah explanation

*Commands:*
- podman build
- podman run

## Build commands
To build a container from an existing Containerfile, all that is needed is specifying the directory and a tag, `-t`, to name the build by. In the following example, we assume the container runs a server on port 80.

```
[user@localhost ~]$ ls 
# myServer

[user@localhost ~]$ podman build -t serverImage myServer/

[user@localhost ~]$ podman images
# REPOSITORY                                   TAG         IMAGE ID      CREATED         SIZE
# localhost/serverImag                         latest      a463eb6e3e4f  19 minutes ago  249 MB
```

This image can then be run through a `docker run` command. It is advisable to specify a name, though the same name cannot be used twice.

```
[user@localhost ~]$ podman run -d -p 8080:80 --name serverInstance serverImage:latest

[user@localhost ~]$ curl localhost:8080
# <h1>Server response</h1>
```

## Writing a Containerfile
Red Hat provides a [_universal basic image_](https://access.redhat.com/articles/4238681) to base your containers from. It is recommended to use it for any container-related activities within RHEL and RHCSA.
Below is a simple containerfile which pulls a standard UBI and installs a httpd server to it. 

Environment:
```
ls ~/myServer/
index.html  Containerfile

cat ~/myServer/index.html
# Hello World
```

```Dockerfile
FROM registry.access.redhat.com/ubi9/ubi
RUN dnf install -y httpd
EXPOSE 80

WORKDIR /var/www/html
COPY index.html /var/www/html/index.html

CMD httpd -D FOREGROUND
```

You can now build and run the Containerfile as shown above.

**📝 NOTE:** *In a way, a Containerfile is just a glorified shell script. This flexibility allows developers to build different containers which achieve the same thing. For instance, instead of copying index.html to the workdir it could be printed directly; or the httpd execution command could be written in a different way. Whichever syntax you find most comfortable is likely fine to use.*


## A word on Buildah
`buildah` is a utility which pre-dates Docker and Containerfiles. It works on a lower-level than `podman build`, providing finer-grained control. You can experiment with it, but for the levels of detail needed in the RHCSA learning podman and Containerfiles is likely enough. 

An interesting apsect of buildah is that it can interactively build a container line-by-line - allowing for trial and error. In production, however, these buildah command are written into a shell script and look similar to a Containerfile anyway. A section of [this article](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users#) is dedicated to buildah and explains it well.