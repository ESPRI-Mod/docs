Thredds machine setup
=========================

* Version: 0.0.1
* Date: 03/07/2020
* Authors: Pierre Logerais
* Keywords: thredds standalone installation

## Description

This procedure describes how to set up a standalone thredds machine (without an esgf node).

* ESGF-Ansible version: 4.0.4, should also work for any 4.x verison 
* License: as ESGF
* Repository: [github](https://github.com/ESGF/esgf-ansible)
* Documentation: [ESGF-Ansible wiki](https://esgf.github.io/esgf-ansible/)

## Hosts & access

### Hosts

esgf-monitoring.ipsl.upmc.fr

### Access

* Protocol: ssh
* Login: esgf-watch-dog
* Command: ssh esgf-watch-dog@esgf-monitoring.ipsl.upmc.fr

## Environment

### Requirements

A Linux machine.

### Dependencies

* Python = 3.6 or higher
* Ansible = 2.7.8 and higher

## Procedure

### Creation of Ansible inventory and configuration

Create a yaml file to configure the host that you will use. In our case, we used a test VM called `vesg-docker1.ipsl.upmc.fr`, which meant that we had the following file : `/home/esgf-watch-dog/esgf-ansible/host_vars/vesg-docker1.ipsl.upmc.fr`

Here is a sample of the variables that should be in there for a test installation :

```
ansible_user: root
admin_pass: aaaaaaa
generate_myproxyca: true # leave self-signed certificates for test installations, otherwise specify where the certificates are on the host machine and not on esgf-monito
generate_globus: true
generate_httpd: true
configure_centos6_iptables: false
configure_centos7_firewalld: false
globus_user: ipslesgf 
globus_pass: <actual password for globus>
register_gridftp: false
register_myproxy: false
```


