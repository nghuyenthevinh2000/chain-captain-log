# Docker file permission problem for mounted volume

Person: Vinh Nguyen
Original problem link: https://github.com/Stride-Labs/stride/pull/327

## Description

1. Mounted volume on docker is owned by root which is not accessible to docker non - root user due to rights.
2. Running as sudo (root) may lead to different directory (directories for non - root and for root can be different) which make necessary files in non - root directory inaccessible.
3. Docker writes files to mounted volume. These files are owned by root which complicates everything (have to use sudo, …)

I believe that whenever Docker is restricted from executing an action (copy, move, …) due to non - root. Docker will then turn to root (highest permission) to execute actions.

## Case

USER A on host

USER B on container

777 folder FA --> mount FA on docker container --> folder FA in container is owned by non-root. Docker then writes file FB to folder FA in container. Since different user, FB is written as root in host. Now, we have FA (non - root) which contains FB (root).

## Best practices

Eliminate root, make sure that file owner is non - root user (solve description 2)

## Solution

1. Description 1: 
    - change permission to 777 on all folders on host machine to make sure that when mounted to docker, it will be owned by non - root user.
2. Description 3:
    - Make sure that there is the same uid and gid on both host machine and container.
    
    ```bash
    ### Host machine
    # 1000 for simplicity
    # 1000 is default non - root user for all linux - based system
    id -un 1000 # vinh
    id -ug 1000 # vinh
    ```
    
    ```docker
    ### Container
    RUN adduser -h /home/workdir -D user -u 1000
    
    USER 1000
    WORKDIR /home/workdir
    ```
    
    - If it is more difficult, then have a look at this two links:
        - [https://gist.github.com/utkuozdemir/3380c32dfee472d35b9c3e39bc72ff01](https://gist.github.com/utkuozdemir/3380c32dfee472d35b9c3e39bc72ff01)
        - [https://github.com/dockcross/dockcross/blob/master/imagefiles/entrypoint.sh#L21](https://github.com/dockcross/dockcross/blob/master/imagefiles/entrypoint.sh#L21)