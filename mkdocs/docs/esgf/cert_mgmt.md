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

Just ask to the IPSL' support for a SSL certificate renewing.

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

Follow the procedure describes in the ESGF-Ansible wiki at this [page](https://esgf.github.io/esgf-ansible/usage/usage.html#local-certificate-installation).

### Login

```bash
ssh esgf-watch-dog@esgf-monitoring.ipsl.upmc.fr
```

### Monitoring

Certificates’ validity and alerts are managed in nagios. Ask to the IPSL's support for
running SSL certificate validity check on the machines.
