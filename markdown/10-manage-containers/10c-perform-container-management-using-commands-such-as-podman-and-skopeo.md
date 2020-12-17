# 10.c Perform container management using commands such as podman and skopeo

### Login to a Container Registry

You can use `podman login [registry]` and `skopeo login [registry]` to login to registries.  

**📌 EXAM TIP:** _If you need to execute `podman {login,inspect,search}` and `skope [login,inspect]` as another user, make sure to use SSH (and not `sudo` or `su`) otherwise it will not work._

### List ports mappings for containers

You can list ports for a running container

    # podman port [container]

Or for all running containers

    # podman port -a

Example:

    $ podman port -a
    3da879abc394    8080/tcp -> 0.0.0.0:8000

**📌 EXAM TIP:** _Anytime you open a port, remember to also open the port with `firewall-cmd`_

### Passing Variables to a Container

As we have seen before, some images can consume variables when you create a container

    # skopeo inspect docker://registry.redhat.io/rhel8/mariadb-103 | egrep '(url|usage)'
        "io.openshift.s2i.scripts-url": "image:///usr/libexec/s2i",
        "io.s2i.scripts-url": "image:///usr/libexec/s2i",
        "url": "https://access.redhat.com/containers/#/registry.access.redhat.com/rhel8/mariadb-103/images/1-116",
        "usage": "podman run -d -e MYSQL_USER=user -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=db -p 3306:3306 rhel8/mariadb-103

We can then create a container by feeding the variable values to `podman run`

    # podman run -d -e MYSQL_USER=admin_user -e MYSQL_PASSWORD=my_secure_passwd -e MYSQL_DATABASE=db1 -p 3306:3306 rhel8/mariadb-103
    Ac7554c6f7f8efd8a9dd93a7f9c05ec84e615c9a4902569afe08ce4fc5ed3905

Here we confirm that the container is running and that the port is up

    # podman ps
    CONTAINER ID  IMAGE                                        COMMAND     CREATED         STATUS             PORTS                   NAMES
    ac7554c6f7f8  registry.redhat.io/rhel8/mariadb-103:latest  run-mysqld  32 seconds ago  Up 29 seconds ago  0.0.0.0:3306->3306/tcp  distracted_chaum

    # podman port -a
    ac7554c6f7f8    3306/tcp -> 0.0.0.0:3306

And successfully connect to the DB (out of scope)

    # mysql -u admin_user -p -h 127.0.0.1  
    Enter password:  
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 8
    Server version: 5.5.5-10.3.17-MariaDB MariaDB Server

    Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | db1                |
    | information_schema |
    | test               |
    +--------------------+
    3 rows in set (0.00 sec)

### Detach from Containers

If the container was run with -i and -t, you can detach from a container and leave it running using the CTRL-p CTRL-q key sequence.

### Rename Container

    # podman rename [container] [new name]

### Running Commands in Container

You can use `podman exec` to run commands inside a running container.

Run `ps –ef | grep sql` inside the 'mysql' container

    # podman exec mysql ps -ef | grep sql
      |  
    mysql          1       0  0 01:22 ?        00:00:00 /usr/libexec/mysqld --defaults-file=/etc/my.cnf
    mysql        131       0  0 01:43 ?        00:00:00 ps -ef

Open an interactive shell (with Bash) inside the container

    [root@localhost ~]# podman exec -ti mysql /bin/bash
    bash-4.4$ exit
    exit
    [root@localhost ~]#

---
[⬅️ Back](10-manage-containers.md)
