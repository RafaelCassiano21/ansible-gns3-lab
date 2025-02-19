# Ansible x Cisco GNSE - Lab Setup Guide

This repository provides a step-by-step guide to setting up a lab environment using GNS3 with Ansible for managing Cisco devices. The environment includes the installation of required packages, GNS3 setup, KVM/QEMU support, and Docker configuration, along with automation examples using Ansible.

## Table of Contents

1. [Install Required Packages](#1-install-required-packages)
2. [Install GNS3 (Server and GUI)](#2-install-gns3-server-and-gui)
3. [Install Dynamips and VPCS](#3-install-dynamips-and-vpcs)
4. [Configure KVM/QEMU Support (Optional)](#4-configure-kvmqemu-support-optional)
5. [Add Docker Support (Optional)](#5-add-docker-support-optional)
6. [Configure SSH for Cisco Devices](#6-configure-ssh-for-cisco-devices)
7. [Run GNS3](#7-run-gns3)
8. [Ansible Playbooks Examples](#8-ansible-playbooks-examples)
   - [Backup Cisco Configuration](#backup-cisco-configuration)
   - [Configure Cisco Banner](#configure-cisco-banner)
   - [Configure ACLs Using Jinja2 Template](#configure-acls-using-jinja2-template)
9. [Inventory / Ansible.cfg](#Inventory/Ansible.cfg)

---

## 1. Install Required Packages

GNS3 requires several dependencies to function correctly. Install all necessary packages using the following command:

`
sudo dnf install -y git gcc cmake flex bison elfutils-libelf-devel libuuid-devel libpcap-devel python3-tornado python3-netifaces python3-devel python3-pip python3-setuptools python3-PyQt5 python3-zmq wireshark python3-paramiko
`


---


## 2. Install-gns3-server-and-gui

GNS3 is available in the official Fedora repositories. To install both the server and GUI, run:
`
sudo dnf install -y gns3-server gns3-gui`


---


## 3. Install-dynamips-and-vpcs

Dynamips is an emulator required to run Cisco router images, and VPCS allows virtual PC simulations. Install them using the following steps:

Install Dynamips:
`
git clone https://github.com/GNS3/dynamips
cd dynamips
mkdir build
cd build
cmake ..
sudo make install`

Install VPCS:
`
wget https://sourceforge.net/projects/vpcs/files/0.8/vpcs_0.8b_Linux64/download -O vpcs
chmod +x vpcs
sudo mv vpcs /usr/local/bin/`


---


## 4. Configure-kvmqemu-support-optional

To use KVM virtual machines within GNS3, install and configure KVM on Fedora:
`
sudo dnf install -y qemu-kvm libvirt libvirt-python libvirt-client virt-install virt-manager
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt $(whoami)`


---


## 5. Add-docker-support-optional

GNS3 also allows the use of Docker containers in its topologies. To configure Docker, run:
`
sudo dnf install -y docker
sudo systemctl enable --now docker
sudo usermod -aG docker $(whoami)
`
Again, after adding your user to the docker group, log out and log back in or restart your system.


---


## 6. Configure-ssh-for-cisco-devices

Edit the `~/.ssh/config` file to include SSH connection settings for Cisco devices. The example below configures a secure connection to the IP 192.168.122.51:

`
Host 192.168.122.51
    KexAlgorithms +diffie-hellman-group1-sha1
    HostKeyAlgorithms +ssh-rsa
    PubkeyAcceptedAlgorithms +ssh-rsa
    Ciphers +aes128-cbc,aes192-cbc,aes256-cbc,3des-cbc
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
`


---


### 7. Run-gns3

After installation, you can start GNS3 by searching for the "GNS3" application in your desktop environment menu or by running:

`gns3         `

On the first run, GNS3 will open a setup wizard. For local usage, select "Run appliances on my local computer" and follow the instructions.


---


### 8. Ansible-playbooks-examples

Below are some example Ansible playbooks for managing Cisco devices.

### Backup-Cisco-Configuration

```
- name: Backup Cisco Configuration
  hosts: all
  gather_facts: yes
  vars:
    ansible_user: admin
    ansible_ssh_pass: redhat
    ansible_network_os: ios
    ansible_connection: network_cli

  tasks:
    - name: Run show startup-config
      cisco.ios.ios_command:
        commands:
          - show startup-config
      register: startup_config

    - name: Save configuration to a temporary file
      ansible.builtin.copy:
        dest: "/tmp/router_startup_config.txt"
        content: "{{ startup_config.stdout[0] }}"
        mode: '0644'

    - name: Fetch the backup file to the local host
      ansible.builtin.fetch:
        src: "/tmp/router_startup_config.txt"
        dest: "./backup_cisco/"
        flat: yes

    - name: Remove temporary file
      ansible.builtin.file:
        path: "/tmp/router_startup_config.txt"
        state: absent
```


## Configure-Cisco-Banner

```
- name: Configure Cisco Banner
  hosts: all
  gather_facts: no
  vars:
    ansible_user: admin
    ansible_ssh_pass: redhat
    ansible_network_os: ios
    ansible_connection: network_cli

  tasks:
    - name: Set banner message
      cisco.ios.ios_config:
        lines:
          - banner motd ^C
          - "  *** Unauthorized access prohibited ***"
          - "  *** This system is monitored ***"
          - ^C

```

## Configure-Acls-using-jinja2-template

Configure ACLs Using Jinja2 Template


Jinja2 Template `acl_template.j2`:
```
ip access-list extended {{ acl_name }}
{% for rule in acl_rules %}
  {{ rule.action }} {{ rule.protocol }} {{ rule.source }} {{ rule.source_wildcard }} {{ rule.destination }} {{ rule.destination_wildcard }}
{% endfor %}
```

Playbook `configure_acl.yml`:
```
- name: Configure ACLs on Cisco
  hosts: all
  gather_facts: no
  vars:
    ansible_user: admin
    ansible_ssh_pass: redhat
    ansible_network_os: ios
    ansible_connection: network_cli
    acl_name: "SECURITY_RULES"
    acl_rules:
      - { action: "deny", protocol: "tcp", source: "any", source_wildcard: "0.0.0.0", destination: "any", destination_wildcard: "0.0.0.0 eq 22" }
      - { action: "permit", protocol: "ip", source: "192.168.1.0", source_wildcard: "0.0.0.255", destination: "any", destination_wildcard: "0.0.0.0" }
      - { action: "deny", protocol: "ip", source: "10.0.0.0", source_wildcard: "0.255.255.255", destination: "any", destination_wildcard: "0.0.0.0" }

  tasks:
    - name: Apply ACL configuration
      cisco.ios.ios_config:
        src: acl_template.j2
```



---


## 9. Inventory / Ansible.cfg

inventory example

```
[routers]
192.168.122.51 ansible_network_os=ios ansible_connection=network_cli ansible_user=admin ansible_ssh_pass=redhat ansible_ssh_common_args='-o KexAlgorithms=+di
ffie-hellman-group1-sha1 -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc,aes192-cbc,aes256-cbc,3des-cbc -o StrictHo
stKeyChecking=no -o UserKnownHostsFile=/dev/null'

[routers:vars]
some_router_specific_variable=value
```


ansible.cfg example

```
[defaults]
roles_path = YOUR ROLE PATH
inventory = YOUR RULE INVENTORY
host_key_checking = false
remote_user = YOUR USER CISCO
remote_password = YOUR CISCO PASSWORD 
host_key_checking = False

[network_cli]
ansible_network_cli_ssh_type: paramiko

[privilege_escalation]
ansible_become: true
ansible_become_method: sudo

[ssh_connection]
ssh_args = -o HostKeyAlgorithms=+ssh-rsa -o KexAlgorithms=+diffie-hellman-group14-sha -o KexAlgorithms=+diffie-hellman-group1-sha1 -o PubkeyAcceptedAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc,aes192-cbc,aes256-cbc,3des-cbc
```





