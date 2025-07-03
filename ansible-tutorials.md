# Ansible Tutorial

## What is Ansible?

Ansible is an open-source automation tool used for:

- Configuration management  
- Application deployment  
- Server provisioning  

It helps automate repetitive tasks and manage multiple systems from a central location.

---

## Set the Hostname

To set the hostname:

```bash
sudo hostnamectl set-hostname <new-hostname>
```

---

## SSH Passwordless Authentication

From your **controller node**, run:

```bash
ssh-keygen
```

Navigate to the `.ssh` directory where you will find:

- `id_rsa` (private key)  
- `id_rsa.pub` (public key)  
- `authorized_keys` (on managed node)

To configure passwordless authentication:

1. Copy the contents of the `id_rsa.pub` file from the controller node:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```
2. SSH into the **managed node**:
   ```bash
   ssh ubuntu@<managed-node-ip>
   ```
3. On the managed node, go to `~/.ssh/` and paste the copied key into the `authorized_keys` file.

Repeat for all managed nodes.

To verify:

```bash
ssh ubuntu@<managed-node-ip>
```

If successful, your setup is correct.

---

## Installing Ansible on Ubuntu

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

After installation, verify Ansible version:

```bash
ansible --version
```

Example output:

```
ansible [core 2.18.6]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/ubuntu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/ubuntu/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.12.3
  jinja version = 3.1.2
  libyaml = True
```

---

## Inventory and Hosts Configuration

Navigate to:

```bash
cd /etc/ansible/
```

Files present:

- `ansible.cfg`  
- `hosts`  
- `roles/`  

### Editing the Default Hosts File

```ini
[webservers]
172.31.1.4
172.31.4.249
172.31.14.251
```

### Testing Connection

```bash
ansible all -m ping
```

Example Output:

```json
172.31.1.4 | SUCCESS => {
    "ping": "pong"
}
```

Warnings regarding Python interpreter can be safely ignored or resolved by defining interpreter path in config.

---

## Using a Custom Inventory File

You can create your own inventory file, e.g., `testinventory`:

```ini
[webservers]
172.31.1.4

[dbserver]
172.31.4.249

[backendserver]
172.31.14.251
```

### Running with a Custom Inventory

```bash
ansible -i testinventory all -m ping
```

- `-i` specifies the inventory file.  
- `all` targets all groups.  
- `-m` specifies the module to run.  
- `ping` is the module used to check connectivity.

---

## Ansible Architecture

![Ansible Architecture](https://github.com/user-attachments/assets/a9fdcbce-7711-4b50-a4af-7d3f180bebb9)

### Controller Node:
- Machine where Ansible is installed.
- Controls the automation process.
- Pushes configurations/commands to managed nodes.

### Inventory File:
- Lists all managed nodes.
- Can contain IPs, hostnames, and groups.

### Managed Nodes:
- Remote systems managed by Ansible.
- Ansible **does not need to be installed** on them.
- Communication is **push-based** using SSH.

---

## Ad-Hoc Commands

### Copy File

```bash
ansible -i <inventory> all -m copy -a "src=/path/to/file dest=/destination"
```

### Run Command

```bash
ansible -i <inventory> all -m command -a "ls"
```

### Common `command` Examples

```bash
ansible all -m command -a "find /tmp -name '*.txt'"
ansible all -m command -a "grep -r 'pattern' /var/log"
ansible all -m command -a "chmod 755 /path/to/file"
ansible all -m command -a "chown user:group /path/to/file"
ansible all -m command -a "mkdir -p /path/to/directory"
```

### Use `shell` for Redirection and Pipes

```bash
ansible -i <inventory> all -m shell -a "echo 'hello' >> file.txt"
```

### Fetch Files

```bash
ansible -i <inventory> all -m fetch -a "src=/remote/path/file.txt dest=/local/path"
```

---

## Ansible Playbooks

Create a YAML playbook file:

```yaml
---
- name: my first ansible playbook
  hosts: all
  become: yes

  tasks:
    - name: hello world
      debug:
        msg: "hello world !"
```

Run the playbook:

```bash
ansible-playbook hello-world.yaml -i /path/to/inventory
```


Happy Automation with Ansible! 
