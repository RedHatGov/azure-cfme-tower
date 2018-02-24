# Network Automation for Operators

## Why Automate Networks

Running networks has not changed fundamentally in decades.  It requires domain specific knowledge and requires a complex set of commands that are done manually, exposing the operation to human error.

Infrastructure as Code (IaC) is describing the enterprise’s intended architecture, policy, connectivity, etc. in code form that describes what the end state of the network should be.

Potentially the biggest advantage of IaC is the collaboration that it enables across the organization.  Instead of the operational knowledge being locked into the minds of a few, code allows the infrastructure to be described in a complete, deterministic way.  The subject matter experts can communicate their knowledge into a central repository giving the entire organization the ability to see and collaborate across silos.

Ansible provides a way to describe your infrastructure as code in the form of a playbook.  These playbooks are a set of ordered tasks that leverage modules to specify the desired end state of a set of devices in an inventory.  These modules, in turn, communicate this intent over ssh or some other API to the devices.  This means that Ansible does not require an agent, giving it the flexibility to automate a wide range of devices.

### Source Code Management

When the infrastructure is described as code, that code can then take advantage of the tools that developers have leveraged and refined for decades.  The most powerful of these tools is source code management (SCM).  SCM provides a way to manage the bits of code that are created by storing them in a central repository.  The main benefits of SCM are:

* Revision Control -- new versions of the code and the changes that beget them are tracked.
* Configuration Management -- All code is housed in a central location to see, comment on, and approve if necessary.

### Data Models

When writing playbooks, it is best to do so in a way that separates the definition of the network from the implementation of that definition.  This decoupling allows the definition of the architecture, intent, policy, etc., once and its application across the infrastructure irrespective of location, device type, or vendor.  In order to do this, a data model must be created that structures the data in a way that is deterministic, agnostic, and flexible.  By doing this, the same configuration data can be fed to two different playbooks configuring, for example,
the underlying switch ports and the infrastructure to monitor those ports.

These data models can be tailored to the need of your organization or just a reflection of some convention or existing data model.  The net_* Declarative Intent modules define a data model by virtue of the attributes that they accept.  For example, this following is configuration data collected from a switch and stored in a way that can be fed directly to the `net_interface` module.

```yaml
interface_data:
- name: Ethernet1/1
  description: Uplink
  mode: trunk
  native_vlan: 10
  mtu: 9000
  state: present
- name: Ethernet1/2
  state: absent
- name: Ethernet1/3
  access_vlan: 1
  description: Host 1
  mode: access
  access_vlan: 100
  mtu: 1500
  state: present
```

### Managing Inventory

As the number of elements being managed with IaC grows, it is important to have a way to organize those element and their associated data.  In addition to the inventory file containing the elements, Ansible provides a hierarchical directory structure for housing vars files.  The host_vars directory contains either a file using the `inventory_hostname` of the device (e.g. switch1.yml) or a directory using the `inventory_hostname` containing multiple vars files.  For example, one vars file (e.g. interfaces.yml) can contain the interface configuration information and another vars file (e.g. vlans.yml) can contain the vlan configuration information for a switch.  Similarly, the group_vars directory contains vars files that apply to particular groups.  Finally, an `all` file or directory can be used to store vars file that apply to all devices.

To organize and separate inventories and configuration data among separate locations or tenants, the entire directory structure can be duplicate.  For an enterprise with two data centers, for example, two completely separate directory structures can be created for DC1 and DC2:  

```
.
├── ansible.cfg
├── inventory/            # Parent directory for our environment-specific directories
│   │
│   ├── DC1/              # Contains all files specific to Data Center 1
│   │   ├── host_vars/    # device specific vars files
│   │   │   ├── router1
│   │   │   │   ├── interfaces.yml  # Device specific interface config
│   │   │   │   └── ospf.yml        # Device specific OSPF config
│   │   │   └── switch1
│   │   │       ├── interfaces.yml  # Device specific interface config
│   │   │       └── vlans.yml       # Device specific vlan config
│   │   │
│   │   ├── group_vars/   # group specific vars files
│   │   │   ├── all
│   │   │   │   └── vlans.yml       # Global VLAN config
│   │   │   ├── routers
│   │   │   │   └── ospf.yml        # Global OSPF config
│   │   │   └── switches
│   │   │       └── stp.yml         # Global STP config
│   │   └── hosts         # Contains only the hosts in the dev environment
│   │
│   └── DC2/              # Contains all files specific to Data Center 1
│       ├── host_vars/    # device specific vars files
│       │   ├── router1
│       │   │   ├── interfaces.yml  # Device specific interface config
│       │   │   └── ospf.yml        # Device specific OSPF config
│       │   └── switch1
│       │       ├── interfaces.yml  # Device specific interface config
│       │       └── vlans.yml       # Device specific vlan config
│       │
│       ├── group_vars/   # group specific vars files
│       │   ├── all
│       │   │   └── vlans.yml       # Global VLAN config
│       │   ├── routers
│       │   │   └── ospf.yml        # Global OSPF config
│       │   └── switches
│       │       └── stp.yml         # Global STP config
│       └── hosts         # Contains only the hosts in the dev environment
│
├── playbook.yml
│
└── . . .
```

### Example Code Repository

Now that we have a potential structure to manage inventory and data, we need a way to combine that with playbooks and roles in a way that can take advantage of both public and privately created code.  In addition, this code needs to be properly revision controlled.

One way to do that is to fork the shared repository (e.g. ansible-netops), then link the inventory and data in as submodules either as the `inventory` director or within the `inventory` directory.  In this case, both the DC1 and DC2 inventories are linked in as a single repository.  Linking DC1 and DC2 in as separate repositories has the benefit of giving each of those inventories from different sources with different administrative controls.  Depending on the exact SCM that you use, you might not be able to fork public repositories into private repositories.  This method addresses that limitation keeping the fork public and making the repositories that contain the inventory and data private.


```
.
├── ansible.cfg
├── <- inventory/            # Parent directory for our environment-specific directories
│      ├── DC1/              # Contains all files specific to Data Center 1
│      └── DC2/              # Contains all files specific to Data Center 2
│
├── roles/            # Parent directory for our environment-specific directories
│   ├── <- net-backup/       # Contains all files specific to Data Center 1
│   └── <- net-version/      # Contains all files specific to Data Center 2
│
├── net-backup.yml
│
└── . . .
```

Here is an example of how we fork the public repository https://github.com/network-automation/ansible-netops and link in the private data.
