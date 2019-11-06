
MIGRATION TROUBLESHOOTING
============================

* Version: 0.0.5
* Date: 03/09/2019
* Authors: Pierre Logerais
* Keywords: migration centos esgf index data node

## Description

This procedure describes common issues related to migrating an ESGF node, and how to fix them.

## Migrating from ESGF 2.7 to ESGF 4 on CentOS 6

### On data nodes

* Errors related to httpd

For these errors, the ansible logs will look like this :

```bash
FAILED! => {
    "changed": false
}

MSG:

Unable to start service httpd: Job for httpd.service failed because the control process exited with error code. See "systemctl status httpd.service" and "journalctl -xe" for details.
```

These errors are caused by a bad configuration of httpd. In this case, you can look at your httpd configuration on older machines, or use the config present at IPSL. As an example, you can find out how we edit configuration at [Ansible upgrade](ansible_upgrade.md).

## While ESGF is running

### On data nodes

* Thredds stopping

You might encounter an error while querying your netcdf files, where the data node doesn’t respond. In this case, you should use an esgf command to restart the data node.
