# NodeTweetAnsible

The setup instructions below assume generic linux servers running Ubuntu-14.04 and managed through Open Nebula.

## Setting up a public Master Load Balancer (MLB)
This section details the setup for an MLB server which will communicate with and control an array of private app servers.
### Setup Ansible
- Update cache to be able to find repositories: `apt-get update`
- Get the add-apt repository: `sudo apt-get install -y software-properties-common`
- Add Ansible's official repository: `sudo add-apt-repository -y ppa:ansible/ansible`
- Update repositories: `apt-get update`
- Install Ansible: `sudo apt-get install -y ansible`

### Setup Git
- `sudo apt-get update`
- `sudo apt-get install git`
- `git config --global user.name “My Name"`
- `git config --global user.email “my.email@domain.com"`

### Get the TweetMap Repo
- Git clone the repo into `/etc/ansible`:
    - `git clone https://github.com/scherroman/TweetMap /etc/ansible/TweetMap`
- Copy config & hosts files from the repo into `/etc/ansible`:
    - `cp -i hosts ..`
    - `cp -i ansible.cfg ..`

### Generate an SSH Key
- Generate a new SSH key on the MLB and add its public key to this repository's deploy keys.
    - `ssh-keygen -t rsa`
    - `cat /root/.ssh/id_rsa.pub`
- Also add this key to open nebula so you can use it to ssh to any newly created private machines.

## Setting up the distributed server infrastructure
This section explains how to use this repository's ansible scripts to setup and maintain an array of generic private servers to build our distributed server infrastructure for TweetMap.
### Update Hosts File
- Update the `hosts` file now in `/etc/ansible` to contain a list of your current private App Servers.
    - [localServer]
    - [frontEndMasterServer]
    - [frontEndSlaveServers]
    - [frontEndServers]

### Setup SSH Agent
Used for Ansible's SSH forwarding
- Startup ssh-agent: `eval $(ssh-agent -s)`
- Add ssh keys to agent: `ssh-add`

**Warning**: Every time you ssh back into the MLB, to allow Ansible to ssh-forward properly to its hosts, you must start up ssh-agent again with: 
- Startup ssh-agent: `eval $(ssh-agent -s)`
- Add ssh keys to agent: `ssh-add`

**Note**: It sometimes takes a couple of minutes for ssh-agent forwarding to start working properly.

### Ansible Deployment Scripts
These scripts are used to deploy and maintain the web app infrastructure for NodeTweet across an arbitrary array of private servers defined in the `hosts` file located in `/etc/ansible`.

- **Test hosts for connectivity**:
    - `ansible all -m ping`
- **Setup All Servers**:
    - `ansible-playbook setupAllServers.yml`
    - Used for initial setup of the MLB and all app servers.
- **Update All Servers**:
    - `ansible-playbook updateAllServers.yml`
    - Used for deploying new updates to the web app.
- **Restart Web App**:
    - `ansible-playbook restartWebApp.yml`
    - Used to restart all node.js web app servers.
- **Restart Web App Hard**:
    - `ansible-playbook restartWebAppHard.yml1
    - Used to restart all infrastructure services, including Nginx servers, rethinkDB instances, and node.js web app servers.

**Note**: Unlike Bash scipts, Ansible scripts are idempotent, meaning they may safely be run as many times as needed to reach the desired state.

### Testing Nginx Setup
- `sudo service nginx configtest`
    - if [FAIL], check nginx error logs
- Checking nginx server access & error logs:
    - `nano /var/log/nginx/access.log`
    - `nano /var/log/nginx/error.log`

### Updating System & Nginx File Limits (optional, not included in scripts)

Can be used to increase traffic load capabilities.

Paste the text below into the bottom of `/etc/security/limits.conf`.

```
\# This is added for Open File Limit Increase
*               hard    nofile          199680
*               soft    nofile          65535

root            hard    nofile          65536
root            soft    nofile          32768

\# This is added for Nginx User
nginx           hard    nofile          199680
nginx           soft    nofile          65535
```

## Misc Resources

###Accessing a remote server's RethinkDB web interface

`ssh -L 9000:10.0.0.17:8080 130.245.168.239 -l root`

Where `10.0.0.17` is the private IP of the final destination machine, `130.245.168.146` is the public IP of the MLB, and `8080` is the default port by which to access RethinkDB's web interace.

Afterwards, enter `localhost:9000` into your browser's search, click on `tables`, select the table to shard, click on `reconfigure` and change as pleased.

###Directly accessing an app server's web interface

- `ssh -L 9000:10.0.0.3:xxxx 130.245.168.239 -l root`

Where `10.0.0.17` is the private IP of the final destination machine, `130.245.168.146` is the public IP of the MLB, and `xxxx` is the port to connect to (generally this can be changed to 80 or something like 3000, depending on whether or not you are using a reverse proxy and what port your web app is actually running out of).

### Ansible Tips
- Ansible Syntax Checking 
    - `ansible-playbook --syntax-check nginx.yml`

- Creating an Ansible Role
    - `mkdir -p roles/xxxx`
    - `cd roles/xxxx`
    - `mkdir files handlers meta templates tasks vars`
    - `mkdir handlers/main.yml meta/main.yml tasks/main.yml vars/main.yml`
 
**Note**: Use `connection: local` in Ansible playbooks when intending to run tasks locally on the current server.

###Helps RethinkDB write more inputs/sec
- durability soft helps
- increasing CPUs per machine helps
- more shards helps

### System Commands
- Check performance of CPUs
    - `top`
- Check # of CPUs
    - `lscpu`
- Check space available on disk
    - `df -h`

## Other Resources
### Regular Expression Helper

https://regex101.com/#javascript

### cURL to Node.js converter (not 100% accurate but almost there)

http://curl.trillworks.com/#node
