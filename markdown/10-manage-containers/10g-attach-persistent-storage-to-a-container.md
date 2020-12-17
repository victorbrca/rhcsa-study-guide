# 10.g Attach persistent storage to a container

## Privileged VS Labeling

Many instructions advise you to use the `--privileged` option to allow mounts to work with podman. This however may not be the best approach because it gives full access to the host (based on the access of the user that started it).  

The reason that mounts do not work is due to SELinux (see below):

> Labeling systems like SELinux require that proper labels are placed on volume content mounted into a container. Without a label, the security system might prevent the processes running inside the container from using the content.  By default, Podman does not change the labels set by the OS.  
>
> To change a label in the container context, you can add either of two suffixes :z or :Z to the volume mount. These suffixes tell Podman to relabel file objects on the shared volumes. The z option tells Podman that two containers share the volume content. As a result, Podman labels the content with a shared content label. Shared volume labels allow all containers to read/write content.  The Z option tells Podman to label the content with a private unshared label. Only the current container can use a private volume.

**The '--privileged' option**

> --privileged=true|false

> Give extended privileges to this container. The default is false.

> By default, Podman containers are unprivileged (=false) and cannot, for example, modify parts of the operating system.  This is because by default a container is only allowed limited access to devices.  A "privileged" container is given the same access to devices as the user launching the container.
>
> A privileged container turns off the security features that isolate the container from the host. Dropped Capabilities, limited devices, read-only mount points, Apparmor/SELinux separation, and Seccomp filters are all disabled.
>
> Rootless containers cannot have more privileges than the account that launched them.

For more reading on this, see Privileged Docker containers—do you really need them?

## Creating the Container with the Volume and Mount

Create the folder that we will use

    # mkdir container_mount

Create the container (with the '-v' option, the first field the host folder, the second field is the mount point on the container and the third the SELinux label 'Z')

    # podman run -ti -v /root/container_mount:/mnt/container_mnt:Z --name devil docker.io/library/httpd /bin/bash

Once in the container, you can check that the mount point is working

    root@bcf532e9194c:/usr/local/apache2# ls -ld /mnt/container_mnt  
    drwxr-xr-x. 2 root root 6 Dec  6 01:42 /mnt/container_mnt

And that the SELinux label has been added

    # ls -ldZ /root/container_mount
    drwxr-xr-x. 2 root root system_u:object_r:container_file_t:s0:c427,c973 35 Dec 11 21:27 /root/container_mount

---
[⬅️ Back](10-manage-containers.md)
