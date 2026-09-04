# SJSU Ansible Two-VM Webserver Assignment

## Student

- Name: Sonit Kumar Pradhan
- Email: sonitkumar.pradhan@sjsu.edu
- SJID: 020622849

## Objective

Use Ansible to deploy, verify, redeploy, and undeploy Nginx webservers on
two Ubuntu virtual machines. Each server listens on port 8080 and displays
a VM-specific message generated from one reusable Jinja2 template.

## Architecture

- macOS host: Ansible control node
- VM1: Ubuntu 24.04, Nginx, port 8080
- VM2: Ubuntu 24.04, Nginx, port 8080
- Virtualization: Multipass
- Source repository: <https://github.com/snkprdhn/Ansible-CMPE-272>

## Prerequisites

- macOS control node
- Ansible installed on the control node
- Multipass installed with two running Ubuntu 24.04 LTS instances
- SSH access to both instances as `ubuntu`
- The private key `~/.ssh/sjsu_ansible`

The included inventory expects the following private addresses:

| Host | Address | Page value |
| --- | --- | --- |
| `vm1` | `192.168.252.2` | `Hello World from SJSU-1` |
| `vm2` | `192.168.252.3` | `Hello World from SJSU-2` |

Update `inventory.ini` if Multipass assigns different addresses.

## Web Pages

- VM1: `Hello World from SJSU-1`
- VM2: `Hello World from SJSU-2`

The shared `templates/index.html.j2` template uses the inventory variable
`sjsu_instance` to render the correct message for each host. Nginx serves
the generated page from `/var/www/sjsu` through the custom server block in
`templates/nginx-sjsu.conf.j2`.

## Commands

Show the VM status and addresses:

```bash
multipass list
```

Test connectivity:

```bash
ansible webservers -m ansible.builtin.ping
```

Validate the playbook syntax:

```bash
ansible-playbook webserver.yml --syntax-check
```

Deploy:

```bash
ansible-playbook webserver.yml --tags deploy
```

Open the deployed pages in a browser:

- <http://192.168.252.2:8080>
- <http://192.168.252.3:8080>

Run deployment again to verify idempotence. An unchanged system should
report `changed=0` for both hosts:

```bash
ansible-playbook webserver.yml --tags deploy
```

Undeploy:

```bash
ansible-playbook webserver.yml --tags undeploy
```

Undeployment removes Nginx, the generated website, its server configuration,
and the enabled-site link. The Multipass virtual machines remain available
for a later deployment.

## Repository Contents

- `webserver.yml`: tagged deploy and undeploy plays
- `inventory.ini`: managed hosts and VM-specific values
- `ansible.cfg`: inventory and SSH defaults
- `templates/index.html.j2`: VM-specific HTML page
- `templates/nginx-sjsu.conf.j2`: Nginx port-8080 configuration
- `Screenshot/`: deployment and verification evidence

## Verification Results

- Both hosts returned `pong` from the Ansible ping module.
- Syntax validation completed successfully.
- Deployment completed with `failed=0` and `unreachable=0`.
- Both browser checks displayed the expected VM-specific message.
- A second deployment produced `changed=0` on both hosts.
- Undeployment removed the webserver resources without removing the VMs.