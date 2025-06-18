# Nomad & Consul Cluster Playbook
> Deploy your portable HomeLab cluster with a single command!

- Companion for video: [The Homelab that fits in your hand! Meet the Nanocluster](https://youtu.be/4ri2GnxCy6Y) 
- [Nomad 101 Video](https://www.youtube.com/watch?v=ckqWFSZ2So4)

This Ansible playbook bootstraps a single-server Nomad & Consul cluster (with client nodes) using Podman (or Docker) as the container runtime.

Features
  • Installs and configures Consul and Nomad agents
  • Sets up a single-server Nomad/Consul cluster (clients only join, no HA server replication yet)
  • Uses Podman by default; can switch to Docker via a variable

Prerequisites
  • Control machine with Ansible (≥2.9) installed
  • SSH access from the control machine to each target host
  • A valid inventory.yml defining your hosts and per-host variables

Getting Started
  1. Clone this repo
```bash
git clone https://github.com/caiodelgadonew/nomad-playbook.git
cd nomad-playbook
```

  2. Copy the inventory file
```bash
cp example.inventory.yml inventory.yml
```

Edit inventory.yml to override:
  • ansible_host (IP or hostname)
  • ansible_user / ansible_ssh_pass
  • datacenter (Nomad/Consul datacenter name)
  • domain (Nomad/Consul DNS domain)
  • server: true on exactly one host
  • client: true on all client nodes

  3. (Optional) Use Docker instead of Podman
in `inventory.yml` set:
```yaml
docker: true
```

  4. Run the playbook
```bash
ansible-playbook -i inventory.yml deploy-cluster.yml --ask-become-pass
```

  5. Bootstrap ACL 
> To access Nomad you need to bootstrap the cluster ACL, you can follow this documentation [Hashicorp Bootstrap the Nomad ACL System](https://developer.hashicorp.com/nomad/tutorials/access-control/access-control-bootstrap) or execute the following commands on the server node
```bash
nomad acl bootstrap
```
Save the generated `Secret ID` somewhere else, if you lose this it will require a cluster reset 
```bash
Accessor ID  = 5b7fd453-d3f7-6814-81dc-fcfe6daedea5
Secret ID    = 9184ec35-65d4-9258-61e3-0c066d0a45c5
Name         = Bootstrap Token
Type         = management
Global       = true
Policies     = n/a
Create Time  = 2017-09-11 17:38:10.999089612 +0000 UTC
Create Index = 7
Modify Index = 7
```

> You can set the environment variables `NOMAD_ADDR` with the server IP and `NOMAD_TOKEN` with the `Secret ID` generated on the boostrap test to interact with Nomad, or you can also authenticate in the UI in the address `http://<NOMAD_ADDR>:4646/ui/settings/tokens`

### Inventory Example
```yaml
prod:
  children:
    nanocluster:
  vars:
    ansible_user: sipeed
    ansible_ssh_pass: licheepi
    datacenter: prod
    domain: example.com
    docker: false

nanocluster:
  hosts:
    lpi3h-1:
      ansible_host: 10.11.11.21
      server: true
      client: true
    lpi3h-2:
      ansible_host: 10.11.11.22
    lpi3h-3:
      ansible_host: 10.11.11.23
    lpi3h-4:
      ansible_host: 10.11.11.24
    lpi3h-5:
      ansible_host: 10.11.11.25
    lpi3h-6:
      ansible_host: 10.11.11.26
    lpi3h-7:
      ansible_host: 10.11.11.27
```

## Variables

| Name                         | Default            | Description                                      |
|------------------------------|--------------------|--------------------------------------------------|
| `datacenter`                 | `prod`             | Nomad/Consul datacenter name                     |
| `domain`                     | `example.com`      | Nomad/Consul DNS domain                          |
| `server`                     | `false`            | Mark exactly one host as the server              |
| `client`                     | `false`            | Mark hosts that should join as Nomad clients     |
| `docker`                     | `false`            | If `true`, installs Docker alongside Podman      |
| `ansible_python_interpreter` | `/usr/bin/python3` | Python interpreter on target hosts               |
