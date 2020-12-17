# 10.e Run a service inside a container

## Pre-Setup

Before we start, we need to enable systemd to be able to write to cgroups when running containers (by default blocked by SELinux). 'cgroups' (or Control Groups) are used for resource management.  

This can be done with 'setsebool':

    # setsebool container_manage_cgroup 1

**💡 TIP:** _If forget what boolean you need to change, just run 'getsebool -a | grep container'. The result will be small and should help you._

## Creating a Custom Image and Creating a Container

### Dockerfile

A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using docker build users can create an automated build that executes several command-line instructions in succession.

Create a file named 'Dockerfile' in an empty folder with the following content (without the line numbers)

    1 FROM registry.access.redhat.com/ubi8/ubi-init
    2 RUN dnf install -y httpd ; dnf clean all  
    3 RUN systemctl enable httpd.service
    4 RUN echo "This is a test server" > /var/www/html/index.html
    5 RUN mkdir /etc/systemd/system/http.service.d/; echo -e '[Service]\nRestart=always' > /etc/systemd/system/http.service.d/override.conf
    6 EXPOSE 80
    7 CMD ["/sbin/init"]

**Break down of the Dockerfile **
Line 1 (FROM): We specify what image will be used
Line 2 (RUN): We install httpd and remove all temporary files used to install httpd
Line 3 (RUN): We enable the httpd.service
Line 4 (RUN): We create an index file to help us test
Line 5 (RUN): We create an override file (drop-in unit) for systemd (see more about it here)
Line 6 (EXPOSE): We expose the default httpd port
Line 7 (CMD): We provide the default command for executing the container

**💡 TIP1:** _If you can't remember all the Dockerfile instructions, try to remember the task. Then you can use 'podman history' with other images to try and figure out the instructions._

**💡 TIP2:** _Remember you want an image with systemd. So search for images with 'init'
Building the image._

Run 'podman build' from the same directory

    # podman build -t httpd_systemd .

Image created

    # podman images
    REPOSITORY                                TAG     IMAGE ID      CREATED         SIZE
    localhost/httpd_systemd                   latest  6c1db7fe4e73  39 minutes ago  247 MB

### Create the container

Create the container with attached output. You can disconnect from the container with 'Ctl+p+q'

    # podman run -ti --name httpd_systemd -p 8080:80 localhost/httpd_systemd

Confirm that it's working

    # podman ps
    CONTAINER ID  IMAGE                                 COMMAND               CREATED        STATUS            PORTS                 NAMES
    443cbcf1ec10  localhost/httpd_systemd:latest        /sbin/init            6 minutes ago  Up 2 minutes ago  0.0.0.0:8080->80/tcp  httpd_systemd

Optionally, add it to systemd (not required)

---
[⬅️ Back](10-manage-containers.md)
