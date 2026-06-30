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
```
<img width="1092" height="737" alt="Screenshot 2026-07-01 012911" src="https://github.com/user-attachments/assets/6b471dd4-9c05-420c-8950-9ba3012251d7" />
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
---
<img width="1391" height="467" alt="Screenshot 2026-07-01 013724" src="https://github.com/user-attachments/assets/83cea171-e8aa-41ba-bcd3-3361465d8b24" />

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
---
<img width="1323" height="455" alt="Screenshot 2026-07-01 013027" src="https://github.com/user-attachments/assets/b04deec6-a0af-455d-91f8-e6280286f34c" />


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






# Ansible Task 2 - Install, Start, and Enable Nginx Service

## Overview

This project demonstrates how to use Ansible to automate the installation and management of the Nginx web server on remote Linux hosts.

The playbook performs the following actions:

1. Installs Nginx on the target servers.
2. Starts the Nginx service.
3. Enables the Nginx service to start automatically during system boot.

---

## Prerequisites

Before running the playbook, ensure:

* Ansible is installed on the Control Node.
* SSH connectivity exists between the Control Node and Managed Nodes.
* Managed Nodes are running Ubuntu/Linux.
* Inventory file contains the target servers under the `webservers` group.

---

## Project Structure

```text
.
├── inventory
└── task2_service.yml
```

---

## Inventory File

Create an inventory file named `inventory`.

Example:

```ini
[webservers]
<MANAGED_NODE_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/key.pem
```

Replace:

* `<MANAGED_NODE_PUBLIC_IP>` with your server's public IP.
* `key.pem` with your SSH private key.

---

## Playbook

File: `task2_service.yml`

```yaml
---
- name: Install and Start Nginx Service
  hosts: webservers
  become: yes

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start and Enable Nginx Service
      service:
        name: nginx
        state: started
        enabled: yes
```

---

## Verify Connectivity

Before executing the playbook, verify connectivity with the managed node:

```bash
ansible webservers -i inventory -m ping
```

Expected Output:

```text
<IP_ADDRESS> | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## Execute the Playbook

Run the following command from the Control Node:

```bash
ansible-playbook -i inventory task2_service.yml
```

---

## Verify Nginx Service

SSH into the managed node and verify the service status:

```bash
sudo systemctl status nginx
```

Expected Output:

```text
Active: active (running)
```

Check whether the service is enabled:

```bash
sudo systemctl is-enabled nginx
```

Expected Output:

```text
enabled
```

---

## Validate Through Browser

Copy the Public IP of the Managed Node and open it in a browser:

```text
http://<MANAGED_NODE_PUBLIC_IP>
```

Expected Result:

```text
Welcome to nginx!
```

<img width="1353" height="681" alt="Screenshot 2026-07-01 015742" src="https://github.com/user-attachments/assets/29ed8aa1-a17e-4ca3-a864-da279c6b8e8b" />





# Ansible Task 3 - Verify Nginx Using Ad-hoc Commands

## Overview

This task demonstrates how to use Ansible ad-hoc commands to verify that Nginx has been installed correctly, the service is running, and the web server is accessible over HTTP.

Unlike previous tasks, no playbook is created. All operations are performed directly from the terminal using Ansible modules and commands.

---

## Prerequisites

Before performing this task, ensure that:

* Ansible is installed on the Control Node.
* SSH connectivity between the Control Node and Managed Node is working.
* Nginx has already been installed and started on the Managed Node.
* The Managed Node is listed under the `webservers` group in the inventory file.

---

## Project Structure

```text
.
└── inventory
```

---

## Inventory File

Example inventory configuration:

```ini
[webservers]
<MANAGED_NODE_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/key.pem
```

Replace:

* `<MANAGED_NODE_PUBLIC_IP>` with the public IP of your EC2 instance.
* `key.pem` with your SSH private key file.

---

## Step 1: Verify Nginx Installation

Run the following command:

```bash
ansible webservers -i inventory -m shell -a "nginx -v"
```

### Sample Output

```text
13.233.xx.xx | CHANGED | rc=0 >>
nginx version: nginx/1.24.0
```

### Verification

This confirms that:

* Nginx is installed on the managed node.
* The installed version is displayed successfully.

---

## Step 2: Verify Nginx Service Status

Run the following command:

```bash
ansible webservers -i inventory -m service -a "name=nginx"
```

### Sample Output

```text
13.233.xx.xx | SUCCESS => {
    "changed": false,
    "name": "nginx",
    "state": "started"
}
```

### Verification

This confirms that:

* The Nginx service exists.
* The service is currently running.

---

## Step 3: Verify HTTP Response Using URI Module

Replace `<PUBLIC-IP>` with the public IP of the managed node.

Run:

```bash
ansible localhost -m uri -a "url=http://<PUBLIC-IP>"
```

Example:

```bash
ansible localhost -m uri -a "url=http://54.123.45.67"
```

### Sample Output

```text
localhost | SUCCESS => {
    "changed": false,
    "status": 200,
    "url": "http://54.123.45.67"
}
```

### Verification

This confirms that:

* The web server is reachable.
* Nginx is serving content correctly.
* The HTTP response code returned is **200 OK**.

---

## Browser Verification

Open the Managed Node Public IP in a web browser:

```text
http://<MANAGED_NODE_PUBLIC_IP>
```

Expected Result:

```text
Welcome to nginx!
```

This confirms that the web server is accessible from the internet.

---

## Commands Summary

| Verification               | Command                                                      |
| -------------------------- | ------------------------------------------------------------ |
| Check Nginx Installation   | `ansible webservers -i inventory -m shell -a "nginx -v"`     |
| Check Nginx Service Status | `ansible webservers -i inventory -m service -a "name=nginx"` |
| Verify HTTP Status 200     | `ansible localhost -m uri -a "url=http://<PUBLIC-IP>"`       |

---

<img width="915" height="420" alt="Screenshot 2026-07-01 020604" src="https://github.com/user-attachments/assets/23bf7e8f-5cd2-4692-bc70-91b608458727" />



# Ansible Task 4 - Deploy a Static HTML Page Using the Copy Module

## Overview

This project demonstrates how to deploy a custom static HTML webpage to a remote Linux server using Ansible.

The playbook performs the following actions:

1. Installs Nginx.
2. Starts and enables the Nginx service.
3. Copies a local HTML file from the Control Node to the Managed Node.
4. Sets the correct ownership and permissions on the deployed file.

After successful execution, the webpage becomes accessible through the Managed Node's public IP address.

---


## Project Structure

```text
.
├── inventory
├── task4_deploy_site.yml
└── files
    └── index.html
```

---

## Step 1: Create the HTML File

Create a directory named `files` and add `index.html`.

File: `files/index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Ansible Site</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <p>This page was deployed using Ansible's copy module.</p>
</body>
</html>
```

---

## Inventory Configuration

Example inventory file:

```ini
[webservers]
<MANAGED_NODE_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/key.pem
```

Replace:

* `<MANAGED_NODE_PUBLIC_IP>` with your EC2 instance public IP.
* `key.pem` with your SSH private key.

---

## Playbook

File: `task4_deploy_site.yml`

```yaml
---
- name: Deploy Static Website Using Ansible
  hosts: webservers
  become: yes

  tasks:

    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start and Enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Copy HTML file to Nginx web root
      copy:
        src: files/index.html
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: '0644'
```

---

## Verify Connectivity

Before running the playbook, verify communication with the managed node:

```bash
ansible webservers -i inventory -m ping
```

Expected Output:

```text
<IP_ADDRESS> | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## Execute the Playbook

Run the following command from the Control Node:

```bash
ansible-playbook -i inventory task4_deploy_site.yml
```

---

## Verify File Deployment

Login to the managed node and verify the file:

```bash
ls -l /var/www/html/index.html
```

Expected Output:

```text
-rw-r--r-- 1 www-data www-data ... index.html
```

### Permission Explanation

| Permission | Description    |
| ---------- | -------------- |
| Owner      | Read and Write |
| Group      | Read           |
| Others     | Read           |

Mode `0644` corresponds to:

```text
-rw-r--r--
```

---

## Verify Nginx Service

Check Nginx status:

```bash
sudo systemctl status nginx
```

Expected Output:

```text
Active: active (running)
```

---

## Browser Verification

Open the Managed Node Public IP in a browser:

```text
http://<MANAGED_NODE_PUBLIC_IP>
```

Expected Result:

```text
Hello from Ansible!

This page was deployed using Ansible's copy module.
```

---
<img width="515" height="241" alt="Screenshot 2026-07-01 021141" src="https://github.com/user-attachments/assets/e950dc81-1dbc-4700-9a33-d2c04fd4cacc" />


## Understanding the Deployment

### Install Nginx

The `apt` module ensures Nginx is installed on the target server.

### Start and Enable Nginx

The `service` module starts the Nginx service and enables it to start automatically after a reboot.

### Copy the Web Page

The `copy` module transfers the HTML file from the Control Node to the Managed Node.

### Configure Ownership and Permissions

The deployed file is assigned:

* Owner: `www-data`
* Group: `www-data`
* Permissions: `0644`

This ensures Nginx can serve the webpage correctly.

---

## Sample Play Recap

First execution:

```text
PLAY RECAP ***********************************************************

13.233.xx.xx : ok=3 changed=3 unreachable=0 failed=0
```

Meaning:

* All tasks completed successfully.
* Changes were made on the server.

Subsequent execution:

```text
PLAY RECAP ***********************************************************

13.233.xx.xx : ok=3 changed=0 unreachable=0 failed=0
```
<img width="922" height="433" alt="Screenshot 2026-07-01 021154" src="https://github.com/user-attachments/assets/d7d1236a-0d40-4220-a7e3-c0cf1902e077" />

Meaning:

* The server is already in the desired state.
* No additional changes were required.
* Demonstrates Ansible's idempotent behavior.

---

















































## Author

Shreya Maidaragi

Automation | DevOps | AWS | Ansible

