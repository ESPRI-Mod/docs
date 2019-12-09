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
vim myhost.cfg
```

Here is an example of how the file could look like :

```bash
wpstest.demo ansible_ssh_user=root ansible_ssh_host=copernicus-wps.ipsl.upmc.fr ansible_ssh_port=22
```

ansible_ssh_user is always root, because you need to be root to perform the installation.
ansible_ssh_host is the name or the IP or your remote machine.
ansible_ssh_port is the port that ansible will use to connect over SSH. 22 is the default port ; if you use port 22, you don’t need any additional configuration.

Now you need to configure things like paths where the WPS will look up your ESGF data. This is done through a yaml file, that you can find examples of in `etc`. In our case at IPSL, since we use it on a load-balancer, the configuration is very close to what you can find in `etc/sample-cp4cds_load-balancer.yml`. Here is our config for cmip-wps :

```yaml
---
wps_external_hostname: cmip-wps.ipsl.upmc.fr
wps_internal_hostname: cmip-wps.ipsl.upmc.fr
wps_add_user: true
wps_user: wps
# wps_uid: 1000
# wps_user_home: /var/lib/pywps
wps_group: wps
# wps_gid: 1000
wps_port: 80
wps_services:
  - name: c4cds
    repo: https://github.com/cp4cds/c4cds-wps.git
    version: master
    hostname: "{{ wps_external_hostname }}"
    fs_hostname: "{{ wps_internal_hostname }}"
    port: "{{ wps_port }}"
    extra_config: |
      [data]
      c3s_cmip5_archive_root: /prodigfs/esgf/published/C3S-CMIP5/output
      cordex_archive_root: /prodigfs/esgf/published/C3S-CORDEX/output1
```

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

* Some possible errors

You might encounter an error like this while running the playbook :

```bash
TASK [ANXS.postgresql : PostgreSQL | Ensure the pid directory for PostgreSQL exists] *********************************************************************************************************************************************************
changed: [copernicus-wps]

TASK [ANXS.postgresql : PostgreSQL | Reload all conf files] **********************************************************************************************************************************************************************************
fatal: [copernicus-wps]: FAILED! => {"changed": false, "msg": "Unable to start service postgresql-11: Job for postgresql-11.service failed because the control process exited with error code. See \"systemctl status postgresql-11.service\" 
and \"journalctl -xe\" for details.\n"}

RUNNING HANDLER [ANXS.postgresql : restart postgresql] ***************************************************************************************************************************************************************************************

```

This means that the postmaster service is not working and can’t restart. If you go to the corresponding machine and type `systemctl status postgresql-11`, you will probably see something like that :

```bash
[root@copernicus-wps]~# systemctl status postgresql-11
● postgresql-11.service - PostgreSQL 11 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-11.service; disabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/postgresql-11.service.d
           └─custom.conf
   Active: failed (Result: exit-code) since lun. 2019-12-09 11:49:42 CET; 2min 18s ago
     Docs: https://www.postgresql.org/docs/11/static/
  Process: 26189 ExecStart=/usr/pgsql-11/bin/postmaster -D ${PGDATA} (code=exited, status=1/FAILURE)
  Process: 26183 ExecStartPre=/usr/pgsql-11/bin/postgresql-11-check-db-dir /var/lib/pgsql/11/data (code=exited, status=0/SUCCESS)
 Main PID: 26189 (code=exited, status=1/FAILURE)

déc. 09 11:49:42 copernicus-wps.ipsl.upmc.fr systemd[1]: Starting PostgreSQL 11 database server...
déc. 09 11:49:42 copernicus-wps.ipsl.upmc.fr postmaster[26189]: 2019-12-09 10:49:42 UTC LOG:  could not bind IPv6 address "::1": Address already in use
déc. 09 11:49:42 copernicus-wps.ipsl.upmc.fr postmaster[26189]: 2019-12-09 10:49:42 UTC HINT:  Is another postmaster already running on port 5432? If not, wait a few seconds and retry.
déc. 09 11:49:42 copernicus-wps.ipsl.upmc.fr systemd[1]: postgresql-11.service: main process exited, code=exited, status=1/FAILURE
déc. 09 11:49:42 copernicus-wps.ipsl.upmc.fr systemd[1]: Failed to start PostgreSQL 11 database server.
déc. 09 11:49:42 copernicus-wps.ipsl.upmc.fr systemd[1]: Unit postgresql-11.service entered failed state.
déc. 09 11:49:42 copernicus-wps.ipsl.upmc.fr systemd[1]: postgresql-11.service failed.
```

In this case, the solution is to kill postmaster manually. Run the following :

`ps aux | grep postgres`

You should see a process running for postmaster :

```bash
postgres 12833  0.0  0.1 362592 15668 ?        Ss   déc.03   0:17 /usr/pgsql-9.6/bin/postmaster -D /etc/postgresql/9.6/data
```

Using its PID, kill it manually. In our case, the command is `kill -9 12833`.

Then relaunch the playbook and it works.

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

