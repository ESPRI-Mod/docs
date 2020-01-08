Certificates Management
=======================

* Version: 0.0.2
* Date: 28/08/2019
* Authors: Sébastien Gardoll
* Keywords: certificate installation ssl ca myproxy globus apache esgf

## Description

This procedure describes how to request a SSL, Myproxy or GridFTP certificate and
gives the link of their installation procedure.

* ESGF version: 4.0.x
* License: as ESFG
* Repository: [github](https://github.com/ESGF/esgf-ansible)
* Documentation: [ESGF-Ansible certificate page wiki](https://esgf.github.io/esgf-ansible/usage/usage.html#local-certificate-installation)

## Hosts & access

### Hosts

Any host that runs an ESGF node.

### Access

- ssh access to esgf-watch-dog account on esgf-monitoring.ipsl.upmc.fr
- ssh access to the root account of the ESGF hosts

## Environment

### Requirements

* An already deployed ESGF node 4.0.x or higher
* An already deployed ESFG-Ansible repository (local or remote)

### Dependencies

For running ESGF-Ansible:

* Python = 3.6 and higher
* Ansible = 2.7.8 and higher

## Procedure

### Web/SSL certificate

* Certificate request

Just ask to the IPSL support for a SSL certificate renewing.

* Certificate installation

Follow the procedure describes in ESGF-Ansible wiki at this [page](https://esgf.github.io/esgf-ansible/usage/usage.html#web-certificate-installation).

### GridFTP/Myproxy certificate

* Certificate request

!!! note
    All the following instructions should be, or are, automated thanks to ESGF-Ansible. Hence, they may become obsolete in the future.

!!! warning
    The following instructions are not tested.

Issue these commands so as to generate the CSR file(s):


if the node is an IDP:
```bash
esgf_host="$(hostname)"
openssl req -new -nodes -config /root/certs/certs/openssl.cnf -keyout /root/certs/esgfcerts/cakey.pem -out /root/certs/esgfcerts/cacert_req.csr -subj "/O=ESGF/OU=ESGF.ORG/CN=$esgf_host-CA"
```

for any kind of node (including IDP node):

```bash
esgf_host="$(hostname)"
openssl req -new -nodes -config /etc/certs/openssl.cnf -keyout /etc/esgfcerts/hostkey.pem -out /etc/esgfcerts/hostcert_req.csr \
-subj "/O=ESGF/OU=ESGF.ORG/CN=$esgf_host"
```

Then, send the csr file(s) to the ESGF certificate signing authority.

!!! warning
    Never get a private key out of its host.

```bash
cd /etc/esgfcerts/
tar -czvf `hostname`_csr_files.tgz *.csr
scp `hostname`_csr_files.tgz YOUR_ACCOUNT@YOUR_MACHINE:.
```

* Certificate installation

When the certificates are expiring, after requesting a signature from the ESGF admins, you will receive an archive containing at least the following files :

* hostcert.pem
* cacert.pem
* cachain.pem
* globus_simple_ca[…].tgz

The name of the globus_simple_ca archive can vary. In our example it was `globus_simple_ca_85bc937c_setup-0.tar.gz`.

These files need to copied *on the node*. We will launch a playbook on esgf-monitoring, but the playbook itself doesn’t make any remote copy : it copies from the node to other places on the node, then launches a few programs.

The place where you want to copy them depends on where you specified the paths in your config file. On esgf-monitoring, the config files are placed in the directory `/home/esgf-watch-dog/scripts/esgf-ansible/host_vars`.

In our case, the relevant variables in our config are as follows, for esgf-node :

```yaml
ansible_user: root
[…]
globushostcert: /etc/grid-security/hostcert.pem
globushostkey: /etc/grid-security/hostkey.pem
[…]
myproxycacert: /etc/esgfcerts/cacert.pem
myproxycakey: /etc/esgfcerts/cakey.pem
myproxy_signing_policy: /etc/esgfcerts/globus_simple_ca_85bc937c_setup-0/85bc937c.signing_policy
[…]
hostkey_src: /etc/esgfcerts/hostkey.pem
hostcert_src: /etc/esgfcerts/hostcert.pem
cachain_src: /etc/esgfcerts/cachain.pem
[…]
```

And for vesg :

```yaml
[…]
globushostcert: /etc/grid-security/hostcert.pem
globushostkey: /etc/grid-security/hostkey.pem
[…]
hostkey_src: /etc/esgfcerts/hostkey.pem
hostcert_src: /etc/esgfcerts/hostcert.pem
cachain_src: /etc/esgfcerts/cachain.pem
[…]
```

Here are what each of these files represent :

* hostcert.pem and hostkey.pem are the public/private key pair for the globus certificate
* cacert.pem and cakey.pem are the public/private key pair for the myproxy certificate
* cachain is the certificate chain.

You need to install the files in the corresponding paths on the respective nodes. Be sure to have the private key copied as well ; you can copy it from somewhere on your node.

If need be, you can check that both of the pairs correspond to the same pairing using these commands :

```bash
openssl x509 -noout -modulus -in certificate.pem | openssl md5
openssl rsa -noout -modulus -in certificatekey.pem | openssl md5
```

These 2 commands should return the same output.

After that, you can run the playbook on esgf-monitoring, as per [the ESGF-Ansible wiki](https://esgf.github.io/esgf-ansible/usage/usage.html#local-certificate-installation).

Here is an example of the commands you can run on esgf-monitoring :

```bash
cd esgf-ansible
script local_certs_install.log
ansible-playbook -v -i /home/esgf-watch-dog/scripts/esgf-ansible/hosts.prod local_certs.yml
```

### Login

```bash
ssh esgf-watch-dog@esgf-monitoring.ipsl.upmc.fr
```

### Monitoring

Certificates’ validity and alerts are managed in nagios. Ask to the IPSL's support for
running SSL certificate validity check on the machines.
