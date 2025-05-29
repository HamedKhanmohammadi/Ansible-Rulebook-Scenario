# Scenario: Automatic Web Server Recovery Using Ansible EDA

##  Goal
Automatically monitor a web server and restart the NGINX service using **Ansible Event-Driven Automation** if the server becomes unreachable.

---

##  Use Case

You are managing a web server located at `http://192.168.56.1`. This server runs a business-critical application using the **NGINX** web server.  
To ensure high availability and minimize downtime, you want to **monitor the server health** and **automatically restart NGINX** when the server is detected as down.

---

##  Files Required

### 1. `check_url_rulebook.yml` (Ansible EDA Rulebook)

```yaml
# check_url_rulebook.yml
---
name: Check webserver
hosts: all

sources:
  - ansible.eda.url_check:
      urls:
        - http://192.168.56.1
      delay: 10

rules:
  - name: Restart Nginx
    condition: event.url_check.status == "down"
    #enabled: false
    action:
      run_playbook:
        name: restart_nginx.yml
```

 **What this does**:
- Monitors the web server URL every 10 seconds.
- If it is down, it triggers the next playbook to restart NGINX.

---

### 2. `restart_nginx.yml` (Ansible Playbook)

```yaml
# restart_nginx.yml
---
- name: Restart Nginx
  hosts: all
  become: true

  tasks:
    - name: Restart Nginx service
      ansible.builtin.service:
        name: nginx
        state: restarted
```

 **What this does**:
- Runs on all hosts defined in your inventory.
- Uses privilege escalation (`become: true`) to restart the NGINX service.

---

##  How to Implement and Run

###  Step 1: Set Up Your Inventory File (e.g., `inventory.ini`)

```ini
[webservers]
192.168.56.1 ansible_user=your_ssh_user ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Replace `your_ssh_user` and the key path with appropriate credentials.

---

###  Step 2: Install Required Tools

Ensure that you have Python 3.9+ and the following packages:

```bash
pip install ansible ansible-rulebook
```

---

###  Step 3: Directory Structure

Organize your files like this:

```
web-recovery/
├── inventory.ini
├── check_url_rulebook.yml
└── restart_nginx.yml
```

---

###  Step 4: Run the Rulebook

Use the following command to start the event-driven automation:

```bash
ansible-rulebook \
  -i inventory.ini \
  -r check_url_rulebook.yml \
  --verbose
```

You will see event logs printed in the terminal. If the server becomes unreachable, the playbook will run automatically and restart NGINX.

---

##  Benefits

- No manual monitoring required
- Lightweight and fast detection (10-second intervals)
- Scalable for more URLs or services
- Easily extendable (e.g., add Slack alerts or log collection)

---

##  Notes

- Make sure SSH access and sudo permissions are properly configured on your target server(s).
- You can test the automation by **stopping NGINX manually** and watching it recover automatically:

```bash
sudo systemctl stop nginx
```
