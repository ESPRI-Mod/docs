ESGF-Ansible Installation
=========================

* Version: 0.0.2
* Date: 15/07/2019
* Authors: Pierre Logerais
* Keywords: esgf ansible installation data node

## Description

This procedure describes how to install ESGF-Ansible and its dependencies.


* ESGF-Ansible version: 4.0.2 and higher
* License: as ESGF
* Repository: [github](https://github.com/ESGF/esgf-ansible)
* Documentation: [ESGF-Ansible wiki](https://esgf.github.io/esgf-ansible/)

## Hosts & access

### Hosts

esgf-monitoring.ipsl.upmc.fr, or any machine that you use to monitor the ansible installations from. It can also be your personal machine.

### Access

Adapt this to your choice of a machine. It goes without saying that you need root access on the machine you want to install your data node on.

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

### Creation of the host file

You will need an inventory file, that specifies at least your data node and the index node at IPSL.

Create ar inventory file, and fill it :

```bash
PARENT_DIR="${HOME}"
cd "${PARENT_DIR}"
mkdir -p inventory
nano inventory/hosts
```

The file should look like this :

```bash
[data]
[FQN of your data node]

[index]
esgf-node.ipsl.upmc.fr

[idp]
esgf-node.ipsl.upmc.fr
```

### Installation of Miniconda

!!! note
    On CentOS 6 (Python 2.6.6), installation of Ansible crashes.


* Custom and run these commands so as to install Miniconda

```bash
PARENT_DIR="${HOME}"
mkdir -p "${PARENT_DIR}/tmp"
cd "${PARENT_DIR}/tmp"

wget 'https://repo.anaconda.com/miniconda/Miniconda2-latest-Linux-x86_64.sh'

chmod +x Miniconda2-latest-Linux-x86_64.sh
./Miniconda2-latest-Linux-x86_64.sh
```

* Then run these commands to create a conda environment

```bash
ENV_NAME='ansible'
source ${HOME}/.bashrc
conda create -y -n "${ENV_NAME}" python=3.6
```

* At last, activate the conda environment and install Ansible

```bash
conda activate ansible
pip install ansible
```

### Installation of ESGF-Ansible

Change the highlighted value.

```bash hl_lines="6"
script install.log
PARENT_DIR="${HOME}"
cd "${PARENT_DIR}"
HOST_NAME="<FQN of your node>"

TAG_NAME='XXX'
git clone https://github.com/ESGF/esgf-ansible.git && cd esgf-ansible && git checkout $TAG_NAME

export ANSIBLE_NOCOLOR=true # log lisible
INVENTORY="${PARENT_DIR}/inventory/hosts"

git status # Just to be sure
ansible-playbook -i "${INVENTORY}" -u root --tags data --limit "${HOST_NAME}" --skip-tags gridftp install.yml
```

### Configuration

The configuration of the ESGF-Ansible node is out of scope. See ESGF-Ansible documentation [here](https://esgf.github.io/esgf-ansible/usage/usage.html#quick-configuration).

### Tests

Running the status recipe of ESGF-Ansible is also a good starting point.

### Maintenance

ESGF-Ansible new releases are exported as Git tags. New release are easily installed as pulling and checkouting the related tag. That's it !

For example, this code install the release 4.0.4

```bash
PARENT_DIR="${HOME}"
TAG_NAME='4.0.4'
cd "${PARENT_DIR}/esgf-ansible"
git pull
git checkout ${TAG_NAME}

```

### Login

```bash
ssh esgf-watch-dog@esgf-monitoring.ipsl.upmc.fr
```
