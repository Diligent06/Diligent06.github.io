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
