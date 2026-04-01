# Docker

## Download docker
Follow install instruction under this page to install docker. (https://docs.docker.com/engine/install/)


## Start a container
Follow command below you can pull down ubuntu 20.04 image and you can use docker images to check it. 

Docker run -it command can create a container which can use host gpu, network, can visualize window in host machine and share ~/data folder under host filesystem into /workspace/data under container.
```bash
docker pull ubuntu:20.04
docker run -it \
  --name your_container_name
  --gpus all \
  --privileged \
  --network host \
  --ipc=host \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v ~/data:/workspace/data \
  --restart unless-stopped \
  ubuntu:20.04

docker start container_name # start container
docker stop container_name # stop container
docker exec -it container_name /bin/bash # go into container
```

## Shared Memory

```bash
# Set shared memory at container creation
docker run -it --shm-size="1g" ubuntu

# Change shared memory of a RUNNING container
su                                                    # Need root
systemctl stop docker
cd /var/lib/docker/containers/<container-id>
# Edit hostconfig.json: set "ShmSize" in bytes (e.g. 1073741824 = 1 GB)
systemctl restart docker
# Start your container — shared memory is now updated
```

## commit a container as an image
```bash
docker commit containe_name your_image_name:[tag] # save a container into an docker image
docker save -o your_image_name.tar your_image_name:[tag]  # save image into a tar file
docker load -i your_image_name.tar   # load a docker image tar file into docker images
```
