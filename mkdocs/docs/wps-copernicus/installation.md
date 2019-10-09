WPS for Copernicus installation
=========================

* Version: 0.0.1
* Date: 2019-8-28
* Authors: Pierre Logerais
* Keywords: wps copernicus local installation

## Description

This procedure describes how to install the WPS stack from a local machine to a remote machine, using an ansible-playbook.

## Environment

### Requirements

A Linux machine, and root access to the remote machine where the stack is installed (either by key, or with a password).

### Dependencies

* ansible-playbook >= 2.7.
* Vagrant + VirtualBox (for testing)

## Procedure

### Fetching the playbook on the local machine

* Run these commands locally to fetch the playbook :

```bash
git clone https://github.com/bird-house/ansible-wps-playbook.git
```

* Install the ansible-playbook package (as root) :

```bash
yum install ansible-playbook ansible
```

* Configure the remote host on the local machine 

You need to make sure that you use a port that is opened on the remote machine. You can always use port 22, as it’s the default ssh port. Create a file on the playbook folder called myhost.cfg :

```bash
cd ansible-wps-playbook
vim myhosts.cfg
```

Here is an example of how the file could look like :

```bash
wpstest.demo ansible_ssh_user=root ansible_ssh_host=copernicus-wps.ipsl.upmc.fr ansible_ssh_port=22
```

ansible_ssh_user is always root, because you need to be root to perform the installation.
ansible_ssh_host is the name or the IP or your remote machine.
ansible_ssh_port is the port that ansible will use to connect over SSH. 22 is the default port ; if you use port 22, you don’t need any additional configuration.

### Adding a port for SSH on the remote machine

Edit the file /etc/ssh/sshd_config on the remote machine (as root or with sudo rights, obviously)

You should see the following line :

```bash
Port 22
```

You can add the port you want by adding another line like this :

```bash
Port 22
Port 5555
```

Then you can restart the SSH daemon with `service sshd restart`.

### Running the playbook

From the `ansible-wps-playbook` repo, run the following command :

```bash
script installation.log
export ANSIBLE_NOCOLOR=true # Make the log readable.

ansible-playbook -i myhost.cfg playbook.yml -K
```

When you will see the `BECOME password:` appear, either type in your password for root on the remote machine (if you log in via password), or do nothing and hit enter (if you are authentificated by key)

Here is a quick rundown of how the options work :
`-i` followed by the inventory file specifies that ansible should run on every host in the file. In this case, you run only on one machine, which is the remote machine you configured eralier.
`-K` is the equivalent of `--become`. It tells ansible that we will be logged in as root.

Everything else is in the playbook :)

### Troubleshooting

* If you have an error about a daemon that can’t start, check its error log or check its status via systemctl. If you see lines like « Adress already in use » it means that the custom port you chose is in conflict with another service. Change it. You can use netstat to check whether or not a port is used by an application or opened.

### Tests

#### Check if the WPS is running

On the machine where the WPS is installed, you can use birdy along with emu to check if your WPS is running. You can run `ipython` and run the following commands in it :

```py
from birdy import WPSClient
emu = WPSClient(url='http://localhost:5000/wps')
emu.hello(name='Birdy').get()[0]
```

If you don’t see any error and the hello is returning a value, that means your WPS is running.

#### Run the WPS

You will need docker on your machine, as the wps uses a docker container :

```bash
yum -y install docker
systemctl start docker 
```

#### Test with Vagrant

There is a premade Vagrantfile in the directory `ansible-wps-playbook`. You need to use a special configuration file for that, found in `etc/custom-vagrant.yml`. 

It’s recommended to test on a normal user instead of root.

Here is how you use the playbook :

```bash
cd ansible-wps-playbook
rm custom.yml
ln -s etc/custom-vagrant.yml custom.yml

vagrant up
vagrant provision```

You should see a playbook running on the provision step.

#### Check the configuration

**FIXME**

