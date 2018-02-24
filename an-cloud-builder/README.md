Ansible Networking Cloud Builder
=========

This is a set of playbooks used for provisioning ansible networking demos in clouds.  It takes a model and creates that architecture in the cloud provider specified by that blueprint.

Requirements
------------

These playbooks assume that you have your cloud environment setup correctly for authentication.  To pull down the repository:

```
git clone git@github.com:ismc/an-demo-kit.git
```

Usage
--------

To build the cloud given the model:

```
ansible-playbook -e 'cloud_model=csr-lab1.yml' build-cloud.yml
```

`buid-cloud.yml` creates  inventory file `{{ cloud_instance }}.yml` in directory `./inventory/{{ cloud_project }}`.  `buid-cloud.yml` can be run multiple times with different models and different cloud_instances, but all in the same project.  This allows for building complex cloud architectures using the models as building block.



The following options can be defined with `-e`:
- `cloud_model`: The cloud model that you want to deploy into the specified cloud provider (required)
- `cloud_project`: The name of the overall project. (defaults to username)
- `cloud_instance`: The specific instance of the overall project. (defaults to cloud_model)
- `cloud_provider`: The cloud provider in which to deploy the model. (defaults to aws)
- `cloud_region`: The region of the cloud provider in which to deploy the model (defaults to us-east-1)
- `inventory_root`: The root of the inventory file, host_vars, and group_vars.

`cloud_name` is derived from `cloud_project` + `cloud_instance` and used to identify that deployment.

To destroy the cloud given the model:

```
ansible-playbook -e 'cloud_model=csr-lab1.yml' destroy-cloud.yml
```

To configure the control node (install Ansible, setup Ansible Inventory, etc)

```
ansible-playbook -i <inventory> configure-control.yml
```

- If you want to skip the deployment of tower on the control node, add `--skip-tags=tower`

- If you want to deploy with the Ansible devel branch, add `-e 'use_ansible_devel=yes'`.  This will also disable installing tower.

You should see the IP address of the control node on successful completion of this playbook:

TASK [debug] ************************************************************************************
ok: [control] => {
    "msg": "Control node public IP':' 34.235.93.223"
}

At this point, you can ssh to `labuser@<control IP address>`

`configure-control.yml` performs the following operations:

- Installs Ansible Engine
- Installs Ansible Tower (unless skipped with `--skip-tags=tower`)
- Installs the dependencies for the modules
- creates the labuser account
- configures syslog to receive log messages
- creates a local git server
- copies the inventory of the cloud_project created by `build-cloud.yml` into the `~labuser/an-demos/inventory`.  Note: It removes the cloud_project directory layer.


## Scenarios
- csr-lab1: Two router setup: 2 x Cisco CSR (IOS)
- multi-lab1: Two router setup: Cisco CSR (IOS) and Juniper MX
- [multi-lab2](scenarios/multi-lab2): A Palo Alto firewall and a F5 Big-IP

## Inventory

In cloud environments, it is best to pull inventory dynamically.  For the purposes of this, however, we create a static inventory on the control node as follows:

```
[all]
rtr1 ansible_host=107.23.75.127 ansible_network_os=ios ansible_ssh_user=ec2-user
rtr2 ansible_host=34.193.22.128 ansible_network_os=ios ansible_ssh_user=ec2-user
control ansible_host=34.193.197.125 ansible_network_os=none ansible_ssh_user=ec2-user
host1 ansible_host=172.18.2.38 ansible_network_os=none ansible_ssh_user=ec2-user
[network]
rtr1 ansible_host=107.23.75.127 ansible_network_os=ios ansible_ssh_user=ec2-user
rtr2 ansible_host=34.193.22.128 ansible_network_os=ios ansible_ssh_user=ec2-user
[routers]
rtr1 ansible_host=107.23.75.127 ansible_network_os=ios ansible_ssh_user=ec2-user
rtr2 ansible_host=34.193.22.128 ansible_network_os=ios ansible_ssh_user=ec2-user
[ungrouped]
[hosts]
control ansible_host=34.193.197.125 ansible_network_os=none ansible_ssh_user=ec2-user
host1 ansible_host=172.18.2.38 ansible_network_os=none ansible_ssh_user=ec2-user
```

The inventory items are grouped function and are stored in the `hosts` file in the inventory directory:

```
.
├── inventory/                  # Inventory directory
│   │
│   ├── host_vars/              # device specific vars files
│   │   ├── rtr1
│   │   │   ├── interfaces.yml  # Device specific interface config
│   │   │   └── ospf.yml        # Device specific OSPF config
│   │   └── switch1
│   │       ├── interfaces.yml  # Device specific interface config
│   │       └── vlans.yml       # Device specific vlan config
│   │
│   ├── group_vars/             # group specific vars files
│   │   ├── all.yml             # Global config   
│   │   ├── network.yml         # Network specific
│   │   └── firewall.yml        # Function specific
│   │   
│   └── hosts         # Contains only the hosts in the dev environment
└── . . .
```

## Playbook Variables

Certain vars files are also created and pre-populated with key/value pairs that represent the Source of Truth for the network.  For example, an `interfaces.yml` file is create in `inventory\host_vars\{{ invetnory_hostname }}` with the interface information for that host:

```yaml
interfaces:
  - name: GigabitEthernet1
    ip: 172.17.0.254/24
    public_ip_address: 107.23.75.127
    public_dns_name: ec2-107-23-75-127.compute-1.amazonaws.com
  - name: GigabitEthernet2
    ip: 172.17.1.254/24
    public_ip_address: 34.192.63.70
    public_dns_name: ec2-34-192-63-70.compute-1.amazonaws.com
  - name: GigabitEthernet3
    ip: 172.17.2.254/24
    public_ip_address:
    public_dns_name:
```


## Running the example playbooks
In order to run the example playbooks, create an inventory directory using
`inventory/example` as a prototype.  The playbooks can then be run with the
command:

```
ansible-playbook -i inventory/<cloud_project>/<cloud_instance> network/network-facts.yml
```

`-i inventory/<cloud_project>/<cloud_instance>` tells ansible-playbook where to look for the inventory.


License
-------

GPL-3

---
![Ansible Red Hat Engine](ansible-engine-small.png)

In addition to open source Ansible, there is Red Hat® Ansible® Engine which includes support and an SLA for the nxos_facts module shown above.

Red Hat® Ansible® Engine is a fully supported product built on the simple, powerful and agentless foundation capabilities derived from the Ansible project.  Please visit [ansible.com](https://www.ansible.com/ansible-engine) for more information.
