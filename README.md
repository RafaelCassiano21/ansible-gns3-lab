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

---

## 1. Install Required Packages

GNS3 requires several dependencies to function correctly. Install all necessary packages using the following command:

`
sudo dnf install -y git gcc cmake flex bison elfutils-libelf-devel libuuid-devel libpcap-devel python3-tornado python3-netifaces python3-devel python3-pip python3-setuptools python3-PyQt5 python3-zmq wireshark
`

---

## 2-install-gns3-server-and-gui

GNS3 is available in the official Fedora repositories. To install both the server and GUI, run:

`sudo dnf install -y gns3-server gns3-gui`

---


