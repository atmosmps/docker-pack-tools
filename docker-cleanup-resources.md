# Docker - How to cleanup (unused) resources

Once in a while, you may need to cleanup resources (containers, volumes, images, networks) ...
These are some simple ways in which this can be done.

## To fast clean

    // see: https://docs.docker.com/engine/reference/commandline/system_prune/#examples

    ```
    $ docker system prune

    WARNING! This will remove:
            - all stopped containers
            - all networks not used by at least one container
            - all dangling images
            - all build cache
    Are you sure you want to continue? [y/N] y
    ```

    NOTE: This does not seem to include removal of unused/dangling volumes. Try: `docker system prune -a --volumes`

### Using in composer

// see: https://github.com/moby/moby/issues/31254#issuecomment-464668235

```
system-prune:
    image: docker
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    command: docker system prune --all --force
    deploy:
      mode: global
      restart_policy:
        delay: 24h
```

## delete volumes
    
    // see: https://github.com/chadoe/docker-cleanup-volumes
    
    $ docker volume rm $(docker volume ls -f dangling=true -q)
    $ docker volume ls -qf dangling=true | xargs -r docker volume rm
    
## delete networks

    $ docker network ls  
    $ docker network ls | grep "bridge"   
    $ docker network ls | awk '$3 == "bridge" && $2 != "bridge" { print $1 }'

    You can also see:

    https://docs.docker.com/engine/reference/commandline/network_prune/

    https://docs.docker.com/engine/reference/commandline/system_prune/
    
## remove docker images
    
    // see: http://stackoverflow.com/questions/32723111/how-to-remove-old-and-unused-docker-images
    
    $ docker images
    $ docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
    
    $ docker images | grep "none"
    $ docker rmi $(docker images | grep "none" | awk '/ / { print $3 }')

## remove docker containers

    // see: http://stackoverflow.com/questions/32723111/how-to-remove-old-and-unused-docker-images
    
    $ docker ps
    $ docker ps -a
    $ docker rm $(docker ps -qa --no-trunc --filter "status=exited")
    
## Resize disk space for docker vm
    
    $ docker-machine create --driver virtualbox --virtualbox-disk-size "40000" default
