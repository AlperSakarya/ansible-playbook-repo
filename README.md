# Ansible Script Toolkit

This repository contains a collection of useful Ansible playbooks designed to help manage and automate tasks on a fleet of servers. Each script addresses a specific task, from managing SSH keys to deploying services or troubleshooting logs.

## Available Playbooks

### 1. `add-key.yml`
This playbook adds a public SSH key to a set of servers, allowing secure access for the specified user. It reads the key from a file on the management machine and adds it to the authorized keys of the target servers.

**Usage:**
- Adds the SSH key `myname.pub` from the local machine to all servers listed under `FLEET_NAME`.

```yaml
- name: adding myname.pub to FLEET_NAME
  hosts: FLEET_NAME
  remote_user: root
  tasks:
    - name: adding the key
      authorized_key:
        user: root
        key: "{{ lookup('file', '/home/yourUserName/.ssh/yourKeyName.pub') }}"
```

### 2. `deploy-new-service.yml`
This playbook automates the deployment of a new service to a group of servers. It creates the necessary service folder, clones a Git repository, and sets up the environment for the service.


**Usage:**
- Creates a folder `/srv/myservice/`, adds a temporary SSH key, clones a Bitbucket repository, and then deletes the SSH key.

```yaml
- name: Deploying new service to FLEET_NAME servers
  hosts: FLEET_NAME
  remote_user: root
  tasks:
    - name: Create service folder
      file: path=/srv/myservice/ state=directory
    - name: Adding the repo key temporarily
      copy: src=/etc/ansible/files/id_rsa dest=/root/.ssh/id_rsa owner=root group=root mode=0600
    - name: Accept bitbucket hostkey
      shell: ssh-keyscan -H bitbucket.org >> /etc/ssh/ssh_known_hosts
    - name: Clone REPO_NAME repository
      git: repo=git@bitbucket.org:ACCOUNT_NAME/REPO_NAME.git dest=/srv/myservice/ accept_hostkey=yes
    - name: Delete keys
      command: mv /root/.ssh/id_rsa /dev/null
```

### 3. `grep-errors-in-logs.yml`
This playbook helps troubleshoot logs by searching for errors in log files across multiple servers. It collects logs that contain "ERROR" and stores them in a temporary file on each server.

**Usage:**
- Greps for "ERROR" in the `/var/log/myLOG` file on the fleet of servers and saves the output to `/tmp/timeouts.txt`.

```yaml
- name: Grepping for errors in Logs
  hosts: FLEET_NAME
  remote_user: root
  tasks:
    - name: Grep for errors in logs
      command: grep ERROR /var/log/myLOG > /tmp/timeouts.txt
```

### How to Use
- Clone this repository.
- Customize the playbooks with your specific values for FLEET_NAME, paths, repository names, and other variables.
- Run the playbooks using the following command:
```bash
ansible-playbook <playbook-name>.yml
```
Ensure that you have Ansible installed and configured with access to the fleet of servers defined in your `hosts` file.
