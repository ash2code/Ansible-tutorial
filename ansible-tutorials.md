Ansible tutorial

What is Ansible ?
Ansible is an open-source automation tool.

what is the use of ansible ?
1) configuration management
2) application deployment
3) server provisioning

It helps automate repetitive tasks and manage multiple systems from a central location.

to set the hostname => sudo hostnamectl set-hostname <new-hostname>

SSH passwordless authentication:
from your controller node => run command 
ssh-keygen

then go to .ssh directory , there we can see 3 files authorised_keys , .pub file and pem file 

cat the .pub file and copy the content , now in another terminal connect to managed node 1 and go to .ssh then open authorised_keys and paste the key which is copied from controller node to authorised_keys of managed node 1
follow same process again for managed node 2

once done , we can do ssh ubuntu@<ip of managed node 1> if it is successful our setup is correct 

Installing Ansible on Ubuntu:

$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible

=> After installing ansible ccheck for ansible --version

ubuntu@controller-node:~$ ansible --version
ansible [core 2.18.6]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/ubuntu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/ubuntu/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.12.3 (main, Feb  4 2025, 14:48:35) [GCC 13.3.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True

now go to the path /etc/ansible

there we can see ansible.cfg  hosts  roles

now open hosts file

update hosts file like below 

  GNU nano 7.2                                                            hosts                                                                     [webservers]
172.31.1.4
172.31.4.249
172.31.14.251


ubuntu@controller-node:~$ ansible all -m ping
[WARNING]: Platform linux on host 172.31.4.249 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of
another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
172.31.4.249 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": false,
    "ping": "pong"
}
[WARNING]: Platform linux on host 172.31.14.251 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of
another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
172.31.14.251 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": false,
    "ping": "pong"
}
[WARNING]: Platform linux on host 172.31.1.4 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of another
Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
172.31.1.4 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": false,
    "ping": "pong"
}


if we do not want to use hosts file we can create our own inventory file and update the ip details

for example we have created a file testinventory and updated the ip details as below 

[webservers]
172.31.1.4

[dbserver]
172.31.4.249

[backendserver]
172.31.14.251

if we want to run any ansible command we shouuld mention the inventory file as well like below

ansible -i testinventory all -m ping

all indicates => all IP addresses which are mentioned in inventory files

m indicates => module

ping => ping is module here


