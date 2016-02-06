# NodeTweetAnsible

The below setup assumes a server running Ubuntu-14.04, as well as Open Nebula for cloud management.

## Setting up the public Master Load Balancer (MLB):
### Setup Ansible
- Update cache to be able to find repositories: `sudo apt-get update`
- Get the "add-apt-repository”: `sudo apt-get install -y software-properties-common`
- Add Ansible's official repository: `sudo add-apt-repository -y ppa:ansible/ansible`
- Update repositories: `apt-get update`
- Install Ansible: `sudo apt-get install -y ansible`
- Find ansible hosts & config file: `cd /etc/ansible`

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
- Generate a new SSH key on the MLB and add its public key to the github repo's deploy keys.
- Also add this key to open nebula so you can use it to ssh to any newly created private machines.
    - ssh-keygen -t rsa
    - cat /root/.ssh/id_rsa.pub

## Setting up the distributed web app infrastricture
### Update Hosts File
- Update the `hosts` file now in `/etc/ansible` to contain a list of your current App Servers.
    - [localServer]
    - [frontEndServers]
    - [databaseServers]

### Setup SSH Agent
- Startup ssh-agent: `eval $(ssh-agent -s)`
- Add ssh keys to agent: `ssh-add`

### Ansible Deployment Scripts:
These scripts are used to deploy and maintain the web app infrastructure for NodeTweet across an arbitrary array of (usually private) servers defined in the `hosts` file located in `/etc/ansible`.

- **Test hosts for Connectability**:
    - `ansible all -m ping`
- **Setup All Servers**:
    - `ansible-playbook setupAllServers.yml`
    - Initial setup scripts for  
- **Update All Servers**:
    - `ansible-playbook updateAllServers.yml`
- **Restart Web App**:
    - `ansible-playbook restartWebApp.yml`
- **Restart Web App Hard**:
    - `ansible-playbook restartWebAppHard.yml1

**Warning**: Every time you ssh back into the MLB, to allow Ansible to ssh-forward properly to its hosts, you must start up ssh-agent again with: 
- Startup ssh-agent: `eval $(ssh-agent -s)`
- Add ssh keys to agent: `ssh-add`

**Note**: It sometimes takes a couple of minutes for ssh-agent forwarding to start working properly.

**Note**: Unlike Bash scipts, Ansible scripts are idempotent, meaning they may safely be run as many times as needed to reach the desired state.

### Testing Nginx Setup:
- `sudo service nginx configtest`
    - if [FAIL], check nginx error logs

### Updating System & Nginx File Limits (optional)
cd /etc/security/limits.conf copy text below into bottom of file
\# This is added for Open File Limit Increase
*               hard    nofile          199680
*               soft    nofile          65535

root            hard    nofile          65536
root            soft    nofile          32768

\# This is added for Nginx User
nginx           hard    nofile          199680
nginx           soft    nofile          65535

***USE ‘connection: local' IN PLAYBOOKS WHEN RUNNING TASKS ON LOCAL SERVER

CHECKING LOAD BALANCER NGINX SERVER ACCESS & ERROR LOGS:
nano /var/log/nginx/access.log
nano /var/log/nginx/error.log

CHECK PERFORMANCE OF CPUS ON MACHINES
top
CHECK NUMBER OF CPUs
lscpu
CHECK SPACE AVAILABLE ON DISK
df -h

WHAT HELPS RETHINKDB WRITE MORE INPUTS/SEC
- durability soft helps
- increasing CPUs per machine helps
- more shards helps
- having noreply in input can possibly cause problems with submission script

"Tunnelception":

ssh -L 9000:10.0.0.17:8080 130.245.168.239 -l root

where 10.0.0.17 is the private IP of the final destination machine and 130.245.168.146 is the IP of the load balancer.

reference: https://piazza.com/class/ic2s17jp8v4is?cid=35

Afterwards, go to localhost:9000, click on "tables", select table to shard, click on "reconfigure" and change as pleased.

Useful Commands:
Ansible Syntax Checking: ansible-playbook --syntax-check nginx.yml

Creating an Ansible Role:
    mkdir -p roles/xxxx
   cd roles/xxxx
  mkdir files handlers meta templates tasks vars
  mkdir handlers/main.yml meta/main.yml tasks/main.yml vars/main.yml

Directly tunneling to localhost:80 or localhost:3000

ssh -L 9000:10.0.0.3:8080 130.245.168.239 -l root

where 10.0.0.17 is the private IP of the final destination machine, 130.245.168.146 is the IP of the load balancer and 8080 is the port to connect to (this can be changed to either 80 or 3000)

Regular Expression Helper:

https://regex101.com/#javascript

cURL to Node.js converter (not 100% accurate but almost there):

http://curl.trillworks.com/#node
