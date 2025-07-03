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

## Ansible Ad-hoc Commands

Ad-hoc commands are one-time commands you can run against your managed nodes without writing a playbook. These are perfect for quick tasks, testing, and one-off operations.

### Basic Syntax
```bash
ansible <hosts> -i <inventory> -m <module> -a "<arguments>"
```

**Where:**
- `<hosts>` - Target hosts or groups (all, webservers, specific IPs)
- `-i` - Specifies inventory file
- `-m` - Module name to execute
- `-a` - Arguments/parameters for the module (indicates arguments)

### COPY Module

The `copy` module copies files from the controller node to managed nodes.

**Basic syntax:**
```bash
ansible -i <inventory-file> <hosts> -m copy -a "src=<source-path> dest=<destination-path>"
```

**Real-world example:**
```bash
ansible -i ashokinv all -m copy -a "src=/home/ubuntu/india.txt dest=/home/ubuntu"
```

**Common copy module parameters:**
- `src` - Source file path on controller
- `dest` - Destination path on managed nodes
- `mode` - File permissions (e.g., mode=0644)
- `owner` - File owner
- `group` - File group
- `backup` - Create backup if file exists (backup=yes)

**Detailed example output:**
```json
172.31.14.251 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": true,
    "checksum": "ad8a325f94e0b06a3e89b9d1524d9d60c442a9c9",
    "dest": "/home/ubuntu/india.txt",
    "gid": 1000,
    "group": "ubuntu",
    "md5sum": "1f95eaa3834241d6185ed13ac76e7583",
    "mode": "0664",
    "owner": "ubuntu",
    "size": 10,
    "src": "/home/ubuntu/.ansible/tmp/ansible-tmp-1751512085.567984-1228-53413726205935/.source.txt",
    "state": "file",
    "uid": 1000
}
```

**Understanding the output:**
- `CHANGED` - File was successfully copied
- `SUCCESS` - File already exists and is identical
- `checksum` - File integrity verification
- `size` - File size in bytes
- `mode` - File permissions in octal format

### COMMAND Module

The `command` module executes commands on managed nodes. It's the default module and doesn't support shell operators like pipes, redirections, or logical operators.

**Basic syntax:**
```bash
ansible -i <inventory> <hosts> -m command -a "<command>"
```

**Real-world examples with actual outputs:**

**1. List files:**
```bash
ansible -i ashokinv all -m command -a "ls"
```
```
172.31.1.4 | CHANGED | rc=0 >>
ashokinv
india.txt
prudhvi

172.31.14.251 | CHANGED | rc=0 >>
india.txt

172.31.4.249 | CHANGED | rc=0 >>
india.txt
```

**2. Remove files:**
```bash
ansible -i ashokinv all -m command -a "rm india.txt"
```
```
172.31.14.251 | CHANGED | rc=0 >>

172.31.1.4 | CHANGED | rc=0 >>

172.31.4.249 | CHANGED | rc=0 >>
```

**3. Check disk usage:**
```bash
ansible -i ashokinv all -m command -a "df -h"
```
```
172.31.14.251 | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       6.8G  1.9G  4.9G  28% /
tmpfs           479M     0  479M   0% /dev/shm
tmpfs           192M  888K  191M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/xvda16     881M   86M  734M  11% /boot
/dev/xvda15     105M  6.2M   99M   6% /boot/efi
tmpfs            96M   12K   96M   1% /run/user/1000
```

**Additional command module examples:**
```bash
# Find files
ansible all -m command -a "find /tmp -name '*.txt'"

# Search in files  
ansible all -m command -a "grep -r 'pattern' /var/log"

# Change permissions
ansible all -m command -a "chmod 755 /path/to/file"

# Change ownership
ansible all -m command -a "chown user:group /path/to/file"

# Create directories
ansible all -m command -a "mkdir -p /path/to/directory"
```

**Understanding command output:**
- `CHANGED | rc=0` - Command executed successfully (return code 0)
- `rc=0` - Return code 0 means success
- Output appears after `>>` symbol

**Command module limitations (these DON'T work):**
```bash
# ❌ Redirection
ansible all -m command -a "echo hello > file.txt"

# ❌ Append
ansible all -m command -a "echo hello >> file.txt"

# ❌ Pipes
ansible all -m command -a "cat file.txt | grep pattern"

# ❌ Logical operators
ansible all -m command -a "ls && echo done"
ansible all -m command -a "ls || echo failed"

# ❌ Command chaining
ansible all -m command -a "command1; command2"
```

### SHELL Module

The `shell` module executes commands through the shell, supporting pipes, redirections, and other shell operators that the command module cannot handle.

**Basic syntax:**
```bash
ansible -i <inventory> <hosts> -m shell -a "<shell-command>"
```

**Real-world example with output:**
```bash
ansible -i ashokinv all -m shell -a "echo 'india' >> us.txt"
```
```
172.31.14.251 | CHANGED | rc=0 >>

172.31.4.249 | CHANGED | rc=0 >>

172.31.1.4 | CHANGED | rc=0 >>
```

**Verification on controller node:**
```bash
ubuntu@controller-node:~$ cat us.txt
andhra
telanagana
goa
india
```

**Shell module capabilities (these WORK):**
```bash
# ✅ Redirection
ansible all -m shell -a "echo hello > file.txt"

# ✅ Append  
ansible all -m shell -a "echo hello >> file.txt"

# ✅ Pipes
ansible all -m shell -a "cat file.txt | grep pattern"

# ✅ Logical operators
ansible all -m shell -a "ls && echo done"

# ✅ Command chaining
ansible all -m shell -a "command1; command2"

# ✅ Environment variables
ansible all -m shell -a "export VAR=value && echo $VAR"

# ✅ Complex shell operations
ansible all -m shell -a "for i in {1..3}; do echo $i; done"
```

**When to use shell vs command:**
- **Use `command`** for simple commands (faster and more secure)
- **Use `shell`** when you need shell features like pipes, redirections, etc.
- **Security note**: Shell module can be less secure as it processes through shell interpreter

### FETCH Module

The `fetch` module retrieves files from managed nodes to the controller node. It's the opposite of the copy module.

**Basic syntax:**
```bash
ansible -i <inventory> <hosts> -m fetch -a "src=<remote-path> dest=<local-path>"
```

**Real-world example with actual output:**
```bash
ansible -i ashokinv all -m fetch -a "src=/home/ubuntu/uk.txt dest=/home/ubuntu"
```

**Detailed output:**
```json
172.31.4.249 | CHANGED => {
    "changed": true,
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
    "dest": "/home/ubuntu/172.31.4.249/home/ubuntu/uk.txt",
    "md5sum": "d41d8cd98f00b204e9800998ecf8427e",
    "remote_checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
    "remote_md5sum": null
}

172.31.14.251 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": false,
    "msg": "the remote file does not exist, not transferring, ignored"
}

172.31.1.4 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": false,
    "msg": "the remote file does not exist, not transferring, ignored"
}
```

**Result on controller node:**
```bash
ubuntu@controller-node:~$ ls
172.31.4.249  ashokinv  prudhvi  us.txt
```

**Fetch module behavior:**
- **Directory structure**: Creates `dest/hostname/src-path` automatically
- **Only existing files**: Fetches files that exist on managed nodes
- **Graceful failure**: Fails gracefully if file doesn't exist (doesn't stop execution)
- **Preserves path**: Maintains original directory structure from source

**Additional fetch examples:**
```bash
# Fetch with custom destination
ansible all -m fetch -a "src=/var/log/syslog dest=./logs/ flat=yes"

# Fetch and validate checksums
ansible all -m fetch -a "src=/etc/passwd dest=./backups/ validate_checksum=yes"

# Fetch only if file exists (fail_on_missing=no is default)
ansible all -m fetch -a "src=/opt/app/config.txt dest=./configs/ fail_on_missing=no"
```

**Understanding fetch output:**
- `CHANGED` - File successfully fetched
- `FAILED` - File doesn't exist or permission denied
- `dest` - Shows exact local path where file was saved
- `checksum` - File integrity verification

---

## Ansible Playbooks

Playbooks are YAML files that define a series of tasks to be executed on managed nodes. They are the heart of Ansible automation and allow you to define complex workflows.

### Basic Playbook Structure

```yaml
---
- name: Playbook description
  hosts: target_hosts
  become: yes  # Run with sudo privileges
  
  tasks:
    - name: Task description
      module_name:
        parameter1: value1
        parameter2: value2
```

### Example: Hello World Playbook

**File: hello-world.yaml**
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

### Running Playbooks

```bash
ansible-playbook hello-world.yaml -i /home/ubuntu/ashokinv
```

**Complete execution output:**
```
ubuntu@controller-node:~/ansible-playbooks$ ansible-playbook hello-world.yaml -i /home/ubuntu/ashokinv

PLAY [my first ansible playbook] *******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************
[WARNING]: Platform linux on host 172.31.14.251 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of
another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [172.31.14.251]
[WARNING]: Platform linux on host 172.31.4.249 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of
another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [172.31.4.249]
[WARNING]: Platform linux on host 172.31.1.4 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of another
Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [172.31.1.4]

TASK [hello world] *********************************************************************************************************************************
ok: [172.31.1.4] => {
    "msg": "hello world !"
}
ok: [172.31.4.249] => {
    "msg": "hello world !"
}
ok: [172.31.14.251] => {
    "msg": "hello world !"
}

PLAY RECAP *****************************************************************************************************************************************
172.31.1.4                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
172.31.14.251              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
172.31.4.249               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Playbook Execution Flow

**1. Gathering Facts**: Ansible automatically collects system information about managed nodes
- Hardware details, network configuration, OS version, etc.
- Can be disabled with `gather_facts: no`

**2. Task Execution**: Tasks are executed in sequential order on all targeted hosts
- Each task runs on all hosts before moving to the next task
- Failed tasks can stop execution unless handled

**3. Play Recap**: Summary showing results for each host
- `ok` - Tasks that ran successfully
- `changed` - Tasks that made changes
- `unreachable` - Hosts that couldn't be reached
- `failed` - Tasks that failed
- `skipped` - Tasks that were skipped due to conditions
- `rescued` - Tasks that were rescued from failure
- `ignored` - Tasks that failed but were ignored

Happy Automation with Ansible! 
