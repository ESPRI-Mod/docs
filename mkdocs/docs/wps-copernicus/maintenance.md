WPS for Copernicus : Maintenance
=========================

* Version: 0.0.1
* Date: 2020-03-23
* Authors: Pierre Logerais
* Keywords: wps copernicus maintenance

## Description

Maintenance cases for the WPS VMs.

## Environment

### Requirements

A Linux machine.

### Dependencies

A sufficiently up-to-date OS (worked with CentOS 7, but encountered problems with CentOS 6 because of the kernel version).
* linux kernel version >= 3.10

## Maintenance cases

### Newer kernel version installed, reboot needed

The process that manages the WPS is automatically restarted upon reboot, so when a newer version of the kernel is installed through yum-cron (you will see the Nagios alert for check_nrpe_kernel), you can simply reboot.
