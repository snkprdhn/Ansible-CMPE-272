# SJSU Ansible Two-VM Webserver Assignment

## Student

- Name: Sonit Kumar Pradhan
- Email: sonitkumar.pradhan@sjsu.edu
- SJID: 020622849

## Demo

[Watch the recorded deployment demonstration](Screenshot/Screen%20Recording%202026-09-03%20at%206.27.20%E2%80%AFPM.mp4)

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

![Multipass showing both Ubuntu 24.04 LTS virtual machines running](Screenshot/Screenshot%202026-09-03%20at%203.14.47%E2%80%AFPM.png)

*Multipass confirms that `vm1` and `vm2` are running with their expected private addresses.*

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

![Ansible inventory graph and successful ping responses from vm1 and vm2](Screenshot/Screenshot%202026-09-03%20at%203.38.32%E2%80%AFPM.png)

*The inventory graph contains both hosts, and each host returns `pong`.*

Validate the playbook syntax:

```bash
ansible-playbook webserver.yml --syntax-check
```

![Successful syntax check for webserver.yml](Screenshot/Screenshot%202026-09-03%20at%203.38.58%E2%80%AFPM.png)

Deploy:

```bash
ansible-playbook webserver.yml --tags deploy
```

![Initial deployment of Nginx and the SJSU website on both virtual machines](Screenshot/Screenshot%202026-09-03%20at%203.39.54%E2%80%AFPM.png)

*The initial deployment completed on both hosts without unreachable or failed tasks.*

Open the deployed pages in a browser:

- <http://192.168.252.2:8080>
- <http://192.168.252.3:8080>

![VM1 displaying Hello World from SJSU-1 on port 8080](Screenshot/Screenshot%202026-09-03%20at%203.42.43%E2%80%AFPM.png)

![VM2 displaying Hello World from SJSU-2 on port 8080](Screenshot/Screenshot%202026-09-03%20at%203.43.01%E2%80%AFPM.png)

*The browser checks show that the shared template renders a different message for each VM.*

Run deployment again to verify idempotence. An unchanged system should
report `changed=0` for both hosts:

```bash
ansible-playbook webserver.yml --tags deploy
```

![Second deployment showing an idempotent result with changed=0 on both hosts](Screenshot/Screenshot%202026-09-03%20at%203.44.02%E2%80%AFPM.png)

*The second deployment reports `changed=0`, confirming that the desired state is already applied.*

Undeploy:

```bash
ansible-playbook webserver.yml --tags undeploy
```

![Undeployment removing Nginx and the generated website resources](Screenshot/Screenshot%202026-09-03%20at%203.44.39%E2%80%AFPM.png)

*The undeploy play completes with `changed=2` per host and no failures.*

Undeployment removes Nginx, the generated website, its server configuration,
and the enabled-site link. The Multipass virtual machines remain available
for a later deployment.

![Post-undeploy curl checks showing the expected unavailable webserver result](Screenshot/Screenshot%202026-09-03%20at%203.46.04%E2%80%AFPM.png)

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