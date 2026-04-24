# Ubuntu System Administration

## User Management

```bash
# Add a new user (run as root)
useradd your_user_name
usermod -aG sudo your_user_name   # Grant sudo privileges
```

## Start a web page server
```bash
python3 -m http.server your_port
```

## List devices connected to current device
```bash
sudo apt install v4l-utils
v4l2-ctl --list-devices
```

## Process Management

```bash
# Check background jobs and kill by job number
jobs -p # get all processes id started by current terminal
kill -9 $(jobs -p) # kill all processes started by current terminal
```

## SSH & Remote File System

```bash
# Mount remote directory via SSHFS
sshfs remote_user@remote_ip:remote_directory your_local_path
```

## Github repository construction
```bash
# First you need to create an empty github repository under your github account and copy its http or ssh url
git init
git config --global user.email "your_email"
git config --global user.name "your_name"
git remote add origin "your_remote_repository_url"
git add .
git commit -m 'your commit log'
# push your local files into 'origin' github repository's master branch
# your local branch corresponding to github repository's branch
git push origin master 

# change branch
git checkout -b you_branch_name # create a branch if it is not existing else change to it
git branch # check current branch
```

## GitHub connect by SSH key

```bash
# ~/.ssh/config
Host Host_alias # for example github_ssh
  HostName Host_ip # for example github.com
  User User_name # for example git
  IdentityFile your_ssh_private_key 

```

## rsync copy files
```bash
sudo apt install rsync
rsync -av --exclude='file_to_skip' /source/folder/ /target/folder/
```

## check how many space current directory occupy
```bash
du -h --max-depth=1 your_directory_path
```

## add a ntp service in host machine
```bash
sudo apt install chrony
sudo systemctl enable chrony
sudo systemctl start chrony
systemctl status chrony
chronyc tracking

sudo vi /etc/chrony/chrony.conf
# add allow 192.168.0.0/24 into file above (you need to change ip subnet)
sudo systemctl restart chrony
```
If you want to use ntp service in python script, you can edit your code like below and you need to pip install ntplib firstly.
```python
import ntplib
client = ntplib.NTPClient()
response = client.request("192.168.50.208")
ntp_time = response.tx_time
```

## connect to a wifi in ubuntu terminal
```bash
sudo nmcli dev wifi list
sudo nmcli device wifi connect "your_wifi_name" password "your_wifi_password"
```

## python package manager tools
```bash
# conda
conda env list
conda env create -n your_env_name python=your_python_version
conda env remove -n your_env_name

# uv
pip install uv # install into your user python interpreter

# pixi
pixi info # print pixi information
pixi add your_package_name # install a python package for pixi environment

```

## docker container display setting
```bash
docker run -it \
  --gpus all \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix

echo $DISPLAY # run this command in host
echo $DISPLAY # run this command in container and check this number is the same with host
```

## find history command in terminal
```bash
history # print last command line history
history | grep your_keyword
```