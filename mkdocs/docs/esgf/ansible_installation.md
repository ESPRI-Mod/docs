ESGF-Ansible Installation
=========================

* Version: 0.0.1
* Date: 15/07/2019
* Authors: Sébastien Gardoll
* Keywords: esgf ansible installation

## Description

This procedure describes how to install ESGF-Ansible and its dependencies.


* ESGF-Ansible version: 4.0.2 and higher
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

```bash
PARENT_DIR="${HOME}"
cd "${PARENT_DIR}"

TAG_NAME='4.0.4'
git clone https://github.com/ESGF/esgf-ansible.git && cd esgf-ansible && git checkout $TAG_NAME
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