
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

### On ESGF-monitoring

* Firefox not being killed properly and bloating the machine

There might be an issue where esgf-monitoring starts to report failures for no apparent reason, and the esgf node still runs correctly. The messages it will send may look like this :
```
something went wrong for /home/esgf-watch-dog/scripts/check_esgf-test-suite.sh:

'> checking with /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
[91mERROR  : No more available loop devices, try increasing 'max loop devices' in singularity.conf
[0m[31mABORT  : Retval = 255
[0m'
```

This message means that singularity can’t run its programs properly, as it has no memory. You can log into esgf-monito and check that the memory and the swap in this case are almost full (command : `free -m`).

In our case, it was caused by a lot of dormant `firefox` processes. The test-suite was using firefox for its tests (basically launching a page and showing if it was showing an error or not). However, it should always close.

We solved the problem by removing the contents of the `__pycache__` directory at these locations :

```
/home/esgf-watch-dog/esgf-test-suite/esgf-test-suite/__pycache__
/home/esgf-watch-dog/esgf-test-suite/esgf-test-suite/tests/__pycache__
```

This solved our problems.
