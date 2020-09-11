Machine Setup
=============

* Version: 0.0.1
* Date: 28/08/2019
* Authors: Sébastien Gardoll
* Keywords: setup centos yum security esgf

## Description

This procedure describes how to set up a fresh machine under CentOS 7.

* CentOS version : 7.6
* License: GNU GPL + others
* Documentation: https://centos.org/

### Access

ssh access to the root's account of the machine.

## Environment

### Requirements

- Admin privileges on the machine
- Prodiguer' Subversion repository access

## Procedure

* Make yum updates the system automatically

On CentOS 7, install package yum-cron, then set the upgrade command (varibale `update_cmd`) to `default` 
(CentOS 7 doesn't support another value) in /etc/yum/yum-cron.conf .

```bash
yum -y install yum-cron
systemctl start yum-cron
systemctl enable yum-cron
```

* Limit the number of kernel 

/boot has a very limited space, so keep up to two kernels by editing /etc/yum.conf
and set `installonly_limit=2`.

Also, comment the following instruction if it exists: `#exclude=kernel*`, so as
to update the kernel.

* Postfix configuration (optional)

Enable sending email from the host for any script.

Add the following instructions at the end of /etc/postfix/main.cf (replace the
highlighted word in capital letters):

``` hl_lines="1"
myhostname = FULL_QUALIFIED_NAME_OF_THE_HOST
myorigin = $mydomain
relayhost = [smtp4web.ipsl.upmc.fr]
inet_interfaces = loopback-only
mydestination =
```

Run these lines in order to test:

```bash
echo -e "You sucessfully received an email from ${HOSTNAME}" | mail -s "${HOSTNAME} email test" YOUR_EMAIL_ADDRESS
sleep 0.5
tail /var/log/maillog
```
You should see `status=sent` in the log file.

The following machines are able to send you email:

- hermes.ipsl.upmc.fr
- hermes-test.ipsl.upmc.fr
- hermes-mq-test.private.ipsl.fr
- hermes-mq-prod.private.ipsl.fr
- esgf-build.ipsl.umpc.fr
- esgf-monitoring.ips.upmc.fr
- esgf-node.ipsl.upmc.fr
- vesg.ipsl.upmc.fr

* Bashrc

Add the following instructions in the /root/.bashrc so as to ease the administration:

```bash
alias ll='ls -lh'
alias la='ls -a'
alias lla='ls -hla'
```

* Add the FQN of the machine in esgf_admin_tools/trunk/scripts/liste_vm.txt

* Ask to the IPSL' support to add the machine to the pool of machines that are 
monitored by the ESPRI-MOD's staff.