WPS for Copernicus : run docker containers
=========================

* Version: 0.0.1
* Date: 2019-8-28
* Authors: Pierre Logerais
* Keywords: wps copernicus emu installation

## Description

Installation of docker, and how to set up a user that can run it without needing root privileges.

## Environment

### Requirements

A Linux machine.

### Dependencies

A sufficiently up-to-date OS (worked with CentOS 7, but encountered problems with CentOS 6 because of the kernel version).
* linux kernel version >= 3.10

## Procedure

### Installation of docker

* Clean up any previous installation of docker, if any

On RedHat based OS, run these commands as root :

```bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

* Then run these commands to install docker

```bash
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

```

If you want to install a nightly or test environment for docker, you can also type this command :

```bash
yum-config-manager --enable docker-ce-nightly
```
for nightly releases, and

```bash
yum-config-manager --enable docker-ce-test
```
for test releases.

Then install docker itself :

```bash
yum install docker-ce docker-ce-cli containerd.io
```

* Running the daemon and testing

If it’s not already done, create a docker group and add it any user that you want to be able to run docker :

Run as root :

```bash
groupadd docker
usermod -aG docker <username>
```

You will have to log back into your shell for these to take effect. Restarting the terminal or typing « bash » or « zsh » into the command line work.

Then, you can run the daemon as root :

```bash
systemctl start docker.service
```

and test it from the user you added to the group docker :

```bash
docker run hello-world
```

