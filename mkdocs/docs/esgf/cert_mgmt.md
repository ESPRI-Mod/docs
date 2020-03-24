Certificates Management
=======================

* Version: 0.0.3
* Date: 30/01/2020
* Authors: Sébastien Gardoll, Pierre Logerais
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

A machine that runs a GridFTP server, in case they are separated (it’s the case at IPSL).

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

For the GridFTP machine : 

```bash
esgf_host="$(hostname)"
rm -fr /root/certs_bak/*
cp /etc/grid-security/hostcert.pem /root/certs_bak/
cp /etc/grid-security/hostkey.pem /root/certs_bak/
mkdir -p /root/certs_bak/globus_connect_server/
cp /var/lib/globus-connect-server/grid-security/hostcert.pem /root/certs_bak/globus_connect_server/
cp /var/lib/globus-connect-server/grid-security/hostkey.pem /root/certs_bak/globus_connect_server/
mkdir -p /root/certs/
openssl req -new -nodes -config /etc/pki/tls/openssl.cnf -keyout /root/certs_bak/hostkey.pem -out /root/certs/hostcert_req.csr -subj "/O=ESGF/OU=ESGF.ORG/CN=$esgf_host"
```

Then, send the csr file(s) to the ESGF certificate signing authority.

!!! warning
    Never get a private key out of its host.

For ESGF nodes :

```bash
cd /etc/esgfcerts/
tar -czvf `hostname`_csr_files.tgz *.csr
scp `hostname`_csr_files.tgz YOUR_ACCOUNT@YOUR_MACHINE:.
```

For the GridFTP machine :
```bash
cd /root/certs/
tar -czvf `hostname`_csr_files.tgz *.csr
scp `hostname`_csr_files.tgz YOUR_ACCOUNT@YOUR_MACHINE:.
```

* Certificate installation

#### GridFTP machine

You should receive an archive containing the following files :

* cachain.pem
* hostcert.pem
* hostcert_req.csr

Extract these to `/root/certs` on the GridFTP machine :

```bash
scp YOUR_ACCOUNT@YOUR_MACHINE:/path/to/certificate/archive.tgz /root/certs/
cd /root/certs
tar xvzf archive.tgz
```

Replace `archive.tgz` with the name of your certificates archive.

Check that the key and certificate match correctly :

```bash
openssl x509 -noout -modulus -in /root/certs/hostcert.pem | openssl md5
openssl rsa -noout -modulus -in /root/certs_bak/hostkey.pem | openssl md5
```

If that’s the case, simply copy the certificates where needed :

```bash
cp /root/certs_bak/hostkey.pem /root/certs
cd /root/certs
cp hostcert.pem /var/lib/globus-connect-server/grid-security/
cp hostkey.pem /var/lib/globus-connect-server/grid-security/
cp hostcert.pem /etc/grid-security/
cp hostkey.pem /etc/grid-security/
```

Then restart GridFTP :

```bash
service globus-gridftp-server restart
```

!!! warning
    Run the esgf-test-suite from esgf-monitoring and be sure that dl_gridftp returns no error.


#### ESGF nodes

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

And for vesg and vesg-x :

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

By convention, the following folders are used for certificates :

On index nodes :

* `/etc/grid-security` contains myproxy certificates
* `/etc/esgfcerts` contains ESGF certificates
* `/etc/certs` contains httpd certificates
* anything in /root/certs is a backup

On data nodes :

* `/etc/grid-security` contains myproxy certificates
* `/etc/esgfcerts` contains ESGF certificates
* `/etc/certs` contains httpd certificates
* anything in /root/certs is a backup

After that, you can run the playbook on esgf-monitoring, as per [the ESGF-Ansible wiki](https://esgf.github.io/esgf-ansible/usage/usage.html#local-certificate-installation).

Here is an example of the commands you can run on esgf-monitoring :

```bash
cd esgf-ansible
script local_certs_install.log
source activate ansible
export ANSIBLE_NOCOLOR=true
ansible-playbook -v -i /home/esgf-watch-dog/scripts/esgf-ansible/hosts.prod local_certs.yml # for the IPSL data node
ansible-playbook -v -i /home/esgf-watch-dog/scripts/esgf-ansible/hosts.x local_certs.yml # for the data node of Polytechnique
```

### Login

```bash
ssh esgf-watch-dog@esgf-monitoring.ipsl.upmc.fr
```

### Monitoring

Certificates’ validity and alerts are managed in nagios. Ask to the IPSL's support for
running SSL certificate validity check on the machines.
