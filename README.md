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
Troubleshooting Tips
