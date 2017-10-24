# ansible-role-localdns
An Ansible role to install bind9 &amp; isc-dhcp-server on the target hosts.

[![Platform](http://img.shields.io/badge/platform-ubuntu-dd4814.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-debian-a80030.svg?style=flat)](#)

## Summary

This Ansible role installs bind9 and isc-dhcp-server on the target host. It allows to create a local network and assign IPs to hosts based on their mac address. This role does not cover all possible configurations and edge cases of a DNS and/or DHCP server but instead was created to cover my specific requirements in my test lab. This includes a fixed 10.0.0.0/16 network in the bind configuration files and some other things like network ranges, subnet masks and router IPs. Nevertheless I hope it is of any use to some people out there.

It has been tested with the following Linux distributions.

* Debian 9

_More Linux distributions will follow soon._

## Dependencies

* Ansible >= 2.x
* aptitude (Ansible < 2.4)

## Installation

To use this role it must first get downloaded from [Ansible Galaxy](https://galaxy.ansible.com). To install the role execute the command below on your Ansible controle machine. For more information please refer to the [official documentation](https://galaxy.ansible.com/intro#download).

`ansible-galaxy install frieder.localdns`

The command above will always pull the latest version off of Galaxy. Another possibility is to pull the role from Github and define a specific release version to download. This allows for better control over which version of this role to be used. Create a `requirements.yml` and put the following content in it.

```yaml
---
- name: frieder.localdns
  src: https://github.com/frieder/ansible-role-localdns
  version: 1.0.0
```

Next you can execute `ansible-galaxy install -r ./requirements.yml --ignore-errors` and it will download all dependencies defined in this list. `--ignore-errors` will make sure that the whole list is processed even when some dependencies are already downloaded. A nice overview of possible entries for `requirements.yml` can be found [here](https://zaiste.net/posts/automatically_install_ansible_galaxy_roles_with_requirements_yml/).


## Role Variables

| Variable | Default | Description |
|--:|:-:|:--|
| localdns_domain | | Defines the domain name of the local network - e.g. `domain.local` |
| localdns_subnet | 255.255.255.0 | The subnet of the network. |
| localdns_dns | | The DNS servers of the network. For multiple DNS servers provide a comma separated list of IPs - e. g. `10.0.1.10,10.0.1.20` |
| localdns_ntp | | The NTP servers of the network. For multiple NTP servers provide a comma separated list of IPs. |
| localdns_dhcp_lease_def | 600 | The default DHCP lease time in seconds. |
| localdns_dhcp_lease_max | 7200 | The maximum DHCP lease time in seconds. |
| localdns_dhcp_ipforwarding | 'on' | Define whether IP forwarding will be allowed or not. Useful if different subnets are in use. |
| localdns_zones | | A list of one or more zones. For an overview of the fields of a zone please check below. |

### Zone ###

Each zone defines a subnet and has the possibility to define the following fields to overwrite the default variables mentioned above.

| Field | Required | Description |
|--:|:-:|:--|
| router | 1 | Define the IP of the zone router. |
| subnet | ? | Define a different subnet mask than the mask in the network settings. |
| dns | ? | Define a different set of DNS servers for this zone. |
| ntp | ? | Define a different set of NTP servers for this zone. |
| ipforwarding | ? | Define whether IP forwarding is enabled for this zone or not. |
| entries/entry | + | For an overview of the possible fields of an entry please check below. |

### Entry ###

Each zone can have multiple entries. Each entry defines the hostname, it's assigned IP address and the mac address of the network interface.

| Field | Description |
|--:|:--|
| host | The hostname |
| ip | The IP that should get assigned to the host |
| mac | The mac address that belongs to this host and ip |

## Example Playbooks

**A simple network with three zones**
```yaml
- hosts: dns
  roles:
     - role: frieder.localdns
       localdns_domain: example.local
       localdns_subnet: 255.255.0.0
       localdns_dns: 10.0.0.3,10.0.1.2,10.0.2.2
       localdns_ntp: '{{ localdns_dns }}'
       localdns_dhcp_lease_def: 600
       localdns_dhcp_lease_max: 7200
       localdns_dhcp_ipforwarding: 'on'
       localdns_zones:
       - zone:
         router: 10.0.0.1
         entries:
         - entry:
           host: 'test1'
           ip: 10.0.0.10
           mac: 'B6:D5:87:4F:1C:32'
       - zone:
         router: 10.0.1.1
         entries:
         - entry:
           host: 'test2'
           ip: 10.0.1.10
           mac: 'A6:C9:79:F2:61:FE'
       - zone:
         router: 10.0.2.1
         subnet: 255.255.255.0
         dns: 10.0.2.2
         ntp: 10.0.2.2
         ipforwarding: 'off'
         entries:
         - entry:
           host: 'test3'
           ip: 10.0.2.10
           mac: '3E:DE:0E:12:F4:E2'
```

### Example Output

The files getting created by aboves playbook are available [here](https://github.com/frieder/ansible-role-localdns/example).

