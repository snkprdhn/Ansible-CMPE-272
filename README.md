# SJSU Ansible Two-VM Webserver Assignment

## Student

- Name: Sonit Kumar Pradhan
- Email: sonitkumar.pradhan@sjsu.edu
- SJID: 020622849

## Objective

Use Ansible to deploy and undeploy Nginx webservers on two Ubuntu
virtual machines. Each server listens on port 8080 and displays a
VM-specific message.

## Architecture

- macOS host: Ansible control node
- VM1: Ubuntu 24.04, Nginx, port 8080
- VM2: Ubuntu 24.04, Nginx, port 8080
- Virtualization: Multipass

## Web Pages

- VM1: `Hello World from SJSU-1`
- VM2: `Hello World from SJSU-2`

## Commands

Test connectivity:

```bash
ansible webservers -m ansible.builtin.ping

Deploy:
ansible-playbook webserver.yml --tags deploy
Undeploy:
ansible-playbook webserver.yml --tags undeploy