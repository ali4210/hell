how to setup ansible in kali linux as a master(192.168.0.150) and in ParrotOS as a node (192.168.0.122)

I'll guide you through setting up Ansible with Kali Linux as the master (control node) and Parrot OS as the managed node.

Step 1: Install Ansible on Kali Linux (Master)
On your Kali Linux machine (192.168.0.150):

bash
# Update package lists
sudo apt update

# Install Ansible
sudo apt install ansible -y

# Verify installation
ansible --version
Step 2: Configure SSH Access
Ansible uses SSH to communicate with nodes. You need passwordless SSH access from master to node.

On Kali Linux (Master):

bash
# Generate SSH key pair (if you don't have one)
ssh-keygen -t rsa -b 4096

# Press Enter to accept default location
# You can set a passphrase or leave it empty for passwordless access
Copy the SSH key to Parrot OS:

bash
# Replace 'user' with your actual username on Parrot OS
ssh-copy-id user@192.168.0.122
You'll be prompted for the Parrot OS user's password. After this, test the connection:

bash
ssh user@192.168.0.122
If you can log in without a password, you're good to go.

Step 3: Configure Ansible Inventory
Create an inventory file to define your managed nodes.

bash
# Create a directory for your Ansible project
mkdir -p ~/ansible-project
cd ~/ansible-project

# Create inventory file
nano inventory.ini
Add the following content:

ini
[parrot_nodes]
parrot-node1 ansible_host=192.168.0.122 ansible_user=user

[parrot_nodes:vars]
ansible_python_interpreter=/usr/bin/python3
Replace user with your actual Parrot OS username.

Step 4: Test the Connection
Run a ping test to verify connectivity:

bash
ansible -i inventory.ini all -m ping
```

You should see a SUCCESS response like:
```
parrot-node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
Step 5: Ensure Python is Installed on Parrot OS
On Parrot OS (192.168.0.122):

bash
# Check if Python 3 is installed
python3 --version

# If not installed, install it
sudo apt update
sudo apt install python3 -y
Step 6: Run Your First Ansible Command
Try some basic Ansible ad-hoc commands:

bash
# Check uptime
ansible -i inventory.ini all -m command -a "uptime"

# Get disk space
ansible -i inventory.ini all -m command -a "df -h"

# Install a package (requires sudo privileges)
ansible -i inventory.ini all -m apt -a "name=htop state=present" --become
Step 7: Create a Simple Playbook (Optional)
Create a playbook to automate tasks:

bash
nano test-playbook.yml
Add this content:

yaml
---
- name: Test Playbook for Parrot OS
  hosts: parrot_nodes
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install basic packages
      apt:
        name:
          - vim
          - curl
          - wget
        state: present

    - name: Create a test file
      file:
        path: /tmp/ansible-test.txt
        state: touch
        mode: '0644'
Run the playbook:

bash
ansible-playbook -i inventory.ini test-playbook.yml















---------------------------------------------------------------------











Is it possible to include another node in my inventory.ini so that I can also run Ansible as a master from Kali Linux to that another Linux machine and that is CentOS which will be working as my node too. Is it possible to connect both machines via Ansible?

Yes, absolutely! You can manage multiple nodes with different Linux distributions from a single Ansible master. Ansible is distribution-agnostic and can manage various Linux flavors simultaneously.

Step 1: Set Up SSH Access to CentOS Node
On your Kali Linux (Master):

bash
# Copy SSH key to CentOS node (replace with your CentOS IP and username)
ssh-copy-id user@192.168.0.XXX

# Test SSH connection
ssh user@192.168.0.XXX
Step 2: Update Your Inventory File
Edit your inventory.ini:

bash
nano ~/ansible-project/inventory.ini
Update it to include both nodes:

ini
# Parrot OS Node
[parrot_nodes]
parrot-node1 ansible_host=192.168.0.122 ansible_user=user

# CentOS Node
[centos_nodes]
centos-node1 ansible_host=192.168.0.XXX ansible_user=centos_username

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
Replace:

192.168.0.XXX with your CentOS IP address
centos_username with your actual CentOS username
Step 3: Test Connection to Both Nodes
bash
# Test all nodes
ansible -i inventory.ini all_nodes -m ping -K

# Test only Parrot OS
ansible -i inventory.ini parrot_nodes -m ping -K

# Test only CentOS
ansible -i inventory.ini centos_nodes -m ping -K
Step 4: Create a Multi-Distribution Playbook
Since CentOS uses yum/dnf instead of apt, you need distribution-aware playbooks:

bash
nano multi-node-playbook.yml
Add this content:

yaml
---
- name: Configure Debian-based systems (Parrot OS)
  hosts: debian_based
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install packages on Debian-based systems
      apt:
        name:
          - vim
          - curl
          - wget
          - htop
        state: present

- name: Configure RedHat-based systems (CentOS)
  hosts: redhat_based
  become: yes
  tasks:
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
Step 5: Run the Multi-Node Playbook
bash
ansible-playbook -i inventory.ini multi-node-playbook.yml -K
Alternative: Using Package Module (Distribution-Agnostic)
You can also use Ansible's package module which automatically detects the package manager:

bash
nano simple-playbook.yml
yaml
---
- name: Configure all nodes (distribution-agnostic)
  hosts: all_nodes
  become: yes
  tasks:
    - name: Install common packages (works on both apt and yum)
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
Run it:

bash
ansible-playbook -i inventory.ini simple-playbook.yml -K
Step 6: Run Ad-Hoc Commands on Specific Nodes
bash
# Run command on all nodes
ansible -i inventory.ini all_nodes -m command -a "uptime" -K

# Run only on CentOS
ansible -i inventory.ini centos_nodes -m command -a "cat /etc/redhat-release" -K

# Run only on Parrot OS
ansible -i inventory.ini parrot_nodes -m command -a "lsb_release -a" -K

# Get disk space from all nodes
ansible -i inventory.ini all_nodes -m command -a "df -h" -K
Pro Tips
Different sudo passwords: If nodes have different sudo passwords, you can specify them individually in inventory:
ini
[centos_nodes]
centos-node1 ansible_host=192.168.0.XXX ansible_user=centos_user ansible_become_pass=centos_password

[parrot_nodes]
parrot-node1 ansible_host=192.168.0.122 ansible_user=parrot_user ansible_become_pass=parrot_password








----------------------------------------------------------------------------------


Okay brother, but the thing is I cannot access my CentOS node without a password or by passwordless entry because after SSH-copy-ID space Saleem@192.168.0.202, doing this it is still requiring password.

No problem! Let's troubleshoot the SSH passwordless login issue. This is a common problem with specific causes.

Step 1: Verify SSH Key Was Copied
On your CentOS node (192.168.0.202), check if the key exists:

bash
# Check if authorized_keys file exists and has content
cat ~/.ssh/authorized_keys

# Check permissions
ls -la ~/.ssh/
ls -la ~/.ssh/authorized_keys
The file should contain your public key from Kali Linux.

Step 2: Fix Permissions on CentOS
SSH is very strict about permissions. On your CentOS node:

bash
# Set correct permissions for .ssh directory
chmod 700 ~/.ssh

# Set correct permissions for authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Ensure your home directory isn't too open
chmod 755 ~

# Check ownership (should be your username)
ls -la ~/.ssh/



ANd Try Your Passwordless SSH Entry Again...........DONE>>>>>>>>>>>>>>>
