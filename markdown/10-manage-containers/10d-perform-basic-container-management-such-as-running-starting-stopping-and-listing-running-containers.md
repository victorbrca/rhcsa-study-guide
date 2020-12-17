# 10.d Perform basic container management such as running, starting, stopping, and listing running containers

## Listing Images

You can use 'podman images'

    # podman images
    REPOSITORY                 TAG     IMAGE ID      CREATED        SIZE
    docker.io/sameersbn/squid  latest  a68a19f689c3  17 months ago  168 MB

Or 'podman image ls'

    # podman image ls
      REPOSITORY                 TAG     IMAGE ID      CREATED        SIZE
      docker.io/sameersbn/squid  latest  a68a19f689c3  17 months ago  168 MB

## Starting/Creating a Container

Create a container from image without starting it

    # podman create docker.io/sameersbn/squid
    d1e9a571aeda3e6dccbd2b12932ff7ba4a2530e0a7664d92e64d2a70a63decdb

Create a container from image and start attached

    # podman run [image]

**📝 NOTE:** _When using 'podman run', If the image doesn't exist it will try to pull it from the registry._

Create a container from image and start detached

    # podman run –d [image]

Start a container

    # podman start [container ID]

## Listing Running Containers

List only running containers

    # podman ps
    CONTAINER ID  IMAGE                             COMMAND  CREATED         STATUS            PORTS   NAMES
    41cc3053faa0  docker.io/sameersbn/squid:latest           9 minutes ago   Up 6 minutes ago          awesome_meninsky
    ff12876812fe  docker.io/sameersbn/squid:latest           10 seconds ago  Up 9 seconds ago          nervous_kepler

List all containers (even stopped containers)

    # podman ps -a
    CONTAINER ID  IMAGE                             COMMAND  CREATED             STATUS                       PORTS   NAMES
    41cc3053faa0  docker.io/sameersbn/squid:latest           10 minutes ago      Up 7 minutes ago                     awesome_meninsky
    ff12876812fe  docker.io/sameersbn/squid:latest           About a minute ago  Exited (137) 24 seconds ago          nervous_kepler

List containers that have stopped

    # podman ps -f "status=exited"
    CONTAINER ID  IMAGE                             COMMAND  CREATED        STATUS                           PORTS   NAMES
    ff12876812fe  docker.io/sameersbn/squid:latest           2 minutes ago  Exited (137) About a minute ago          nervous_kepler

## Stopping a Container

    # podman stop ff12876812fe
    ff12876812fe12919f227b2fcab596f2948097d6f8b1ae94f014976e06a86c36

**📝 NOTE:** _The stop commands attempts to stop the container, and if the container does not stop after 10s it will kill the container process._

---
[⬅️ Back](10-manage-containers.md)
