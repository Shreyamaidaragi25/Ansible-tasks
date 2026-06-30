# Ansible Task 1 - Install Nginx Using a Playbook

## Overview

This project demonstrates how to use Ansible to automate the installation of Nginx on remote servers using a playbook.

You need to use Ansible to install Nginx automatically on another server.

Think of it like this:

Control Node = The machine where Ansible is installed.
Managed Node = The machine where Nginx will be installed.


The playbook performs the following actions:

* Targets the `webservers` group from the inventory file
* Uses privilege escalation (`become: yes`)
* Updates the APT package cache
* Installs the Nginx package

---

## Architecture

```text
+-------------------+
|   Control Node    |
|  (Ansible Server) |
+---------+---------+
          |
          | SSH
          |
          v
+-------------------+
|   Managed Node    |
|   Ubuntu Server   |
|      Nginx        |
+-------------------+
```

---

## Prerequisites

* Ubuntu Control Node
* Ubuntu Managed Node
* Ansible installed on the Control Node
* SSH access configured between Control Node and Managed Node
* Security Group allowing:

  * SSH (Port 22)
  * HTTP (Port 80)

---

## Step 1: Install Ansible

Update package repositories and install Ansible:

```bash
sudo apt update
sudo apt install ansible -y
```

Verify installation:

```bash
ansible --version
```

---

## Step 2: Create Inventory File

Create an inventory file named `inventory`.

```ini
[webservers]
<Managed-Node-Public-IP> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ubuntu.pem
```

Example:

```ini
[webservers]
65.1.128.218 ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ubuntu.pem
```

---

## Step 3: Test Connectivity

Verify communication between the Control Node and Managed Node.

```bash
ansible webservers -i inventory -m ping
```

Expected Output:

```bash
65.1.128.218 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

<img width="1092" height="737" alt="Screenshot 2026-07-01 012911" src="https://github.com/user-attachments/assets/6b471dd4-9c05-420c-8950-9ba3012251d7" />

```

---

## Step 4: Create the Playbook

Create a file named:

```bash
task1_install_nginx.yml
```

Add the following content:

```yaml
---
- name: Install Nginx
  hosts: webservers
  become: yes

  tasks:
    - name: Install nginx package
      apt:
        name: nginx
        state: present
        update_cache: yes
```

---

## Step 5: Run the Playbook

Execute the playbook:

```bash
ansible-playbook -i inventory task1_install_nginx.yml
```

Expected Output:

```bash
PLAY [Install Nginx]

TASK [Install nginx package]
changed: [65.1.128.218]

PLAY RECAP
65.1.128.218 : ok=1 changed=1 failed=0
```
<img width="1391" height="467" alt="Screenshot 2026-07-01 013724" src="https://github.com/user-attachments/assets/83cea171-e8aa-41ba-bcd3-3361465d8b24" />

---

## Step 6: Verify Nginx Installation

Login to the managed node:

```bash
ssh -i ubuntu.pem ubuntu@<Managed-Node-Public-IP>
```

Check Nginx status:

```bash
sudo systemctl status nginx
```

Expected Output:

```bash
Active: active (running)
```

---

## Step 7: Access Nginx Web Page

Open a browser and navigate to:

```text
http://<Managed-Node-Public-IP>
```

Example:

```text
http://65.1.128.218
```

Expected Result:

```text
Welcome to nginx!
```
<img width="1323" height="455" alt="Screenshot 2026-07-01 013027" src="https://github.com/user-attachments/assets/b04deec6-a0af-455d-91f8-e6280286f34c" />

---


## Key Ansible Concepts Used

### Inventory

Defines the target hosts managed by Ansible.

### Playbook

A YAML file that contains automation tasks.

### Become

Allows tasks to run with elevated privileges.

```yaml
become: yes
```

### Apt Module

Used to manage packages on Ubuntu systems.

```yaml
apt:
  name: nginx
  state: present
  update_cache: yes
```

---

## Outcome

Successfully automated the installation of Nginx on a remote Ubuntu server using Ansible and verified the deployment through a web browser.
