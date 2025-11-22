# 🚀 Ansible Multi-Node Setup Guide

[![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kali-linux&logoColor=white)](https://www.kali.org/)
[![CentOS](https://img.shields.io/badge/CentOS-262577?style=for-the-badge&logo=centos&logoColor=white)](https://www.centos.org/)
[![Parrot OS](https://img.shields.io/badge/Parrot_OS-33CCCC?style=for-the-badge&logo=linux&logoColor=white)](https://www.parrotsec.org/)

> A comprehensive guide to setting up Ansible with Kali Linux as the master node, managing both Parrot OS and CentOS nodes for multi-distribution infrastructure automation.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
  - [Step 1: Install Ansible on Kali Linux](#step-1-install-ansible-on-kali-linux-master)
  - [Step 2: Configure SSH Access](#step-2-configure-ssh-access)
  - [Step 3: Configure Ansible Inventory](#step-3-configure-ansible-inventory)
  - [Step 4: Setup Python on Nodes](#step-4-setup-python-on-nodes)
  - [Step 5: Test Connections](#step-5-test-connections)
- [Multi-Node Configuration](#multi-node-configuration)
- [Playbook Examples](#playbook-examples)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Resources](#resources)

---

## 🎯 Overview

This project demonstrates setting up an Ansible automation environment with:
- **Master Node**: Kali Linux (192.168.0.150)
- **Managed Nodes**:
  - Parrot OS (192.168.0.122)
  - CentOS (192.168.0.202)

Ansible enables infrastructure automation, configuration management, and application deployment across heterogeneous Linux environments.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│          Ansible Master (Kali Linux)            │
│              IP: 192.168.0.150                  │
│                                                 │
│  • Ansible Control Node                        │
│  • SSH Key Management                           │
│  • Playbook Execution                           │
└────────────┬────────────────────┬───────────────┘
             │                    │
             │ SSH                │ SSH
             │                    │
    ┌────────▼─────────┐  ┌──────▼──────────┐
    │   Parrot OS      │  │    CentOS       │
    │  192.168.0.122   │  │  192.168.0.202  │
    │                  │  │                 │
    │  Debian-based    │  │  RedHat-based   │
    │  Uses: apt       │  │  Uses: yum/dnf  │
    └──────────────────┘  └─────────────────┘
```

---

## ✅ Prerequisites

### On All Machines:
- Linux operating system installed and running
- Network connectivity between all nodes
- SSH service enabled
- Python 3 installed

### Required Information:
- IP addresses of all nodes
- Username and password for each node
- Sudo privileges on managed nodes

---

## 🛠️ Installation Steps

### Step 1: Install Ansible on Kali Linux (Master)

On your Kali Linux machine:

```bash
# Update package lists
sudo apt update

# Install Ansible
sudo apt install ansible -y

# Verify installation
ansible --version
```

**Expected Output:**
```
ansible [core 2.x.x]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/kali/.ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.x.x
```

---

### Step 2: Configure SSH Access

Ansible uses SSH for communication. Set up passwordless SSH authentication.

#### Generate SSH Key on Master

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096

# Press Enter to accept default location
# You can set a passphrase or leave it empty
```

#### Copy SSH Key to Managed Nodes

**For Parrot OS:**
```bash
ssh-copy-id user@192.168.0.122
```

**For CentOS:**
```bash
ssh-copy-id saleem@192.168.0.202
```

#### Test SSH Connection

```bash
# Test Parrot OS
ssh user@192.168.0.122

# Test CentOS
ssh saleem@192.168.0.202
```

If you can log in without a password, SSH is configured correctly.

#### Troubleshooting SSH on CentOS (SELinux)

If passwordless SSH doesn't work on CentOS:

```bash
# On CentOS node, fix permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Fix SELinux context
restorecon -R -v ~/.ssh

# Verify SSH server configuration
sudo nano /etc/ssh/sshd_config
# Ensure these lines are uncommented:
# PubkeyAuthentication yes
# AuthorizedKeysFile .ssh/authorized_keys

# Restart SSH service
sudo systemctl restart sshd
```

---

### Step 3: Configure Ansible Inventory

Create an inventory file to define your managed nodes.

```bash
# Create project directory
mkdir -p ~/ansible-project
cd ~/ansible-project

# Create inventory file
nano inventory.ini
```

**Add the following content:**

```ini
# Parrot OS Node
[parrot_nodes]
parrot-node1 ansible_host=192.168.0.122 ansible_user=user

# CentOS Node
[centos_nodes]
centos-node1 ansible_host=192.168.0.202 ansible_user=saleem

# Group all Debian-based systems
[debian_based]
parrot-node1

# Group all RedHat-based systems
[redhat_based]
centos-node1

# Group all nodes together
[all_nodes:children]
parrot_nodes
centos_nodes

# Variables for Parrot OS
[parrot_nodes:vars]
ansible_python_interpreter=/usr/bin/python3

# Variables for CentOS
[centos_nodes:vars]
ansible_python_interpreter=/usr/bin/python3
```

**Replace:**
- `user` with your actual Parrot OS username
- `saleem` with your actual CentOS username

---

### Step 4: Setup Python on Nodes

Ensure Python 3 is installed on all managed nodes.

**On Parrot OS:**
```bash
sudo apt update
sudo apt install python3 -y
python3 --version
```

**On CentOS:**
```bash
sudo yum install python3 -y
python3 --version
```

---

### Step 5: Test Connections

#### Test with Ansible Ping Module

```bash
# Test all nodes
ansible -i inventory.ini all_nodes -m ping

# Test only Parrot OS
ansible -i inventory.ini parrot_nodes -m ping

# Test only CentOS
ansible -i inventory.ini centos_nodes -m ping
```

**Expected Output:**
```
parrot-node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
centos-node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

#### Run Ad-Hoc Commands

```bash
# Check uptime on all nodes
ansible -i inventory.ini all_nodes -m command -a "uptime"

# Get disk space
ansible -i inventory.ini all_nodes -m command -a "df -h"

# Check OS information
ansible -i inventory.ini all_nodes -m command -a "uname -a"
```

---

## 🌐 Multi-Node Configuration

### Understanding Groups

Ansible allows you to group nodes for targeted management:

- `parrot_nodes` - Only Parrot OS systems
- `centos_nodes` - Only CentOS systems
- `debian_based` - All Debian-family distributions
- `redhat_based` - All RedHat-family distributions
- `all_nodes` - All managed systems

### Target Specific Groups

```bash
# Run commands on Debian-based systems only
ansible -i inventory.ini debian_based -m command -a "apt --version"

# Run commands on RedHat-based systems only
ansible -i inventory.ini redhat_based -m command -a "yum --version"
```

---

## 📝 Playbook Examples

### Basic Multi-Distribution Playbook

Create a playbook that handles both Debian and RedHat systems:

```bash
nano multi-node-playbook.yml
```

```yaml
---
- name: Configure Debian-based systems (Parrot OS)
  hosts: debian_based
  become: yes
  tasks:
    - name: Install packages on Debian-based systems
      apt:
        name:
          - vim
          - curl
          - wget
          - htop
        state: present
        update_cache: no

- name: Configure RedHat-based systems (CentOS)
  hosts: redhat_based
  become: yes
  tasks:
    - name: Enable EPEL repository
      yum:
        name: epel-release
        state: present
    
    - name: Update yum cache
      yum:
        update_cache: yes
      
    - name: Install packages on RedHat-based systems
      yum:
        name:
          - vim
          - curl
          - wget
          - htop
        state: present

- name: Common tasks for all nodes
  hosts: all_nodes
  become: yes
  tasks:
    - name: Create a test directory
      file:
        path: /tmp/ansible-test
        state: directory
        mode: '0755'
        
    - name: Create a test file
      copy:
        content: "Managed by Ansible from Kali Linux\n"
        dest: /tmp/ansible-test/info.txt
        mode: '0644'
        
    - name: Get system information
      command: uname -a
      register: system_info
      
    - name: Display system info
      debug:
        msg: "{{ inventory_hostname }}: {{ system_info.stdout }}"
```

**Run the playbook:**

```bash
ansible-playbook -i inventory.ini multi-node-playbook.yml -K
```

The `-K` flag prompts for the sudo password.

---

### Distribution-Agnostic Playbook

Using the `package` module for automatic package manager detection:

```yaml
---
- name: Configure all nodes (distribution-agnostic)
  hosts: all_nodes
  become: yes
  tasks:
    - name: Install common packages
      package:
        name:
          - vim
          - curl
          - wget
        state: present
        
    - name: Create test file
      file:
        path: /tmp/ansible-managed.txt
        state: touch
        mode: '0644'
```

---

### OS Information Gathering Playbook

```yaml
---
- name: Check OS Information on All Nodes
  hosts: all
  gather_facts: yes
  tasks:
    - name: Display OS Information
      debug:
        msg: |
          Hostname: {{ inventory_hostname }}
          OS Family: {{ ansible_os_family }}
          Distribution: {{ ansible_distribution }}
          Distribution Version: {{ ansible_distribution_version }}
          Kernel: {{ ansible_kernel }}
          Architecture: {{ ansible_architecture }}
```

---

## 🔧 Troubleshooting

### Common Issues and Solutions

#### 1. SSH Connection Issues

**Problem:** `Permission denied (publickey,password)`

**Solution:**
```bash
# Verify SSH key is copied
ssh user@192.168.0.122

# Re-copy SSH key if needed
ssh-copy-id -i ~/.ssh/id_rsa.pub user@192.168.0.122
```

---

#### 2. Missing Sudo Password

**Problem:** `Missing sudo password`

**Solution:**
```bash
# Use -K flag when running playbooks
ansible-playbook -i inventory.ini playbook.yml -K

# Or configure passwordless sudo on nodes
sudo visudo
# Add: username ALL=(ALL) NOPASSWD: ALL
```

---

#### 3. Package Not Found (htop on CentOS)

**Problem:** `No package htop available`

**Solution:**
```bash
# Enable EPEL repository first
ansible -i inventory.ini centos_nodes -m yum -a "name=epel-release state=present" --become -K
```

---

#### 4. Apt Cache Update Failure (Parrot OS)

**Problem:** `Failed to update apt cache`

**Solution:**
```bash
# On Parrot OS node
sudo killall apt apt-get
sudo rm -f /var/lib/apt/lists/lock
sudo rm -f /var/cache/apt/archives/lock
sudo rm -f /var/lib/dpkg/lock*
sudo dpkg --configure -a
sudo apt clean
sudo apt update

# Or skip cache update in playbook
update_cache: no
```

---

#### 5. SELinux Blocking SSH Keys (CentOS)

**Problem:** SSH key authentication not working on CentOS

**Solution:**
```bash
# On CentOS node
restorecon -R -v ~/.ssh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

### Verbose Mode for Debugging

```bash
# Run with verbose output
ansible -i inventory.ini all -m ping -v
ansible -i inventory.ini all -m ping -vv
ansible -i inventory.ini all -m ping -vvv

# Debug SSH connection
ssh -v user@192.168.0.122
```

---

## 🎓 Best Practices

### 1. Security

- ✅ Use SSH keys instead of passwords
- ✅ Use Ansible Vault for sensitive data
- ✅ Limit sudo access appropriately
- ✅ Keep your master node secure

### 2. Inventory Management

- ✅ Organize hosts into logical groups
- ✅ Use descriptive hostnames
- ✅ Document your inventory structure
- ✅ Use variables for repeated values

### 3. Playbook Design

- ✅ Make playbooks idempotent (safe to run multiple times)
- ✅ Use roles for complex configurations
- ✅ Handle errors gracefully with `ignore_errors` or `failed_when`
- ✅ Test playbooks in a staging environment first

### 4. Version Control

- ✅ Store playbooks and inventory in Git
- ✅ Use meaningful commit messages
- ✅ Create branches for experimental changes
- ✅ Document changes in README

---

## 📚 Useful Commands Cheat Sheet

### Ad-Hoc Commands

```bash
# Ping all nodes
ansible -i inventory.ini all -m ping

# Run shell command
ansible -i inventory.ini all -m shell -a "uptime"

# Check disk space
ansible -i inventory.ini all -m command -a "df -h"

# Install package (requires sudo)
ansible -i inventory.ini all -m package -a "name=git state=present" --become -K

# Copy file to nodes
ansible -i inventory.ini all -m copy -a "src=/local/file dest=/remote/file" --become

# Gather facts
ansible -i inventory.ini all -m setup

# Filter specific facts
ansible -i inventory.ini all -m setup -a "filter=ansible_distribution*"
```

### Playbook Commands

```bash
# Run playbook
ansible-playbook -i inventory.ini playbook.yml

# With sudo password prompt
ansible-playbook -i inventory.ini playbook.yml -K

# Check syntax
ansible-playbook -i inventory.ini playbook.yml --syntax-check

# Dry run (check mode)
ansible-playbook -i inventory.ini playbook.yml --check

# Limit to specific hosts
ansible-playbook -i inventory.ini playbook.yml --limit centos_nodes

# Start at specific task
ansible-playbook -i inventory.ini playbook.yml --start-at-task="task name"

# Run with tags
ansible-playbook -i inventory.ini playbook.yml --tags "install,configure"
```

---

## 🔗 Resources

### Official Documentation
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Getting Started](https://docs.ansible.com/ansible/latest/getting_started/index.html)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html)

### Learning Resources
- [Ansible for DevOps](https://www.ansiblefordevops.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/) - Community roles and collections
- [Ansible Examples](https://github.com/ansible/ansible-examples)

### Community
- [Ansible Community](https://www.ansible.com/community)
- [Reddit r/ansible](https://www.reddit.com/r/ansible/)
- [Stack Overflow - Ansible](https://stackoverflow.com/questions/tagged/ansible)

---

## 👨‍💻 Author

**Saleem Ali**
- LinkedIn: [linkedin.com/in/saleem-ali-189719325](https://www.linkedin.com/in/saleem-ali-189719325/)
- Currently studying AIOps at Al-Nafi International College
- Tech enthusiast passionate about automation and infrastructure management

---

## 📄 License

This project is open source and available for educational purposes.

---

## 🙏 Acknowledgments

- Ansible community for excellent documentation
- Linux community for robust distributions
- Open source contributors

---

## 📝 Version History

- **v1.0** (November 2025) - Initial setup with Kali Linux master, Parrot OS and CentOS nodes

---

## 🤝 Contributing

Feel free to fork this repository and submit pull requests for improvements!

---

## 📧 Contact

For questions or feedback, connect with me on [LinkedIn](https://www.linkedin.com/in/saleem-ali-189719325/)

---

**Happy Automating! 🚀**
