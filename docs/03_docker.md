# Docker

## Container Lifecycle

```bash
# Create a new container with a volume mount
docker run -it \
  -v /home/diligent/Desktop/BIGAI-EAI-RoboVerse:/home/diligent/RoboVerse_local \
  --name metasim_local \
  bigai-eai-roboverse-metasim

# Connect as a specific user
docker exec -u your_user_name <container_name> bash
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

## Display (GUI) Inside Docker

```bash
# Fix "cannot open display" error
export DISPLAY=:1
```

## Proxy for Container Network

Configure Squid on the **host machine** so containers can use the host's network proxy:

```bash
# 1. Install Squid on host
sudo apt install squid

# 2. Edit config
sudo vi /etc/squid/squid.conf
# Add these lines:
acl localnet src 10.1.110.0/23
http_access allow localnet
http_port 3128

# 3. Restart Squid
sudo systemctl restart squid
sudo systemctl status squid
```

Then inside the container:

```bash
export http_proxy=http://<host-ip>:3128
export https_proxy=http://<host-ip>:3128
```

> **Tip:** Set the proxy before starting IsaacLab — otherwise startup may hang.
