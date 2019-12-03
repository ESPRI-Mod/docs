How to reboot your machines and what services to restart
============================

* Version: 0.0.1
* Date: 03/12/2019
* Authors: Pierre Logerais
* Keywords: maintenance esgf

## Description

This procedure describes what to do when rebooting some of the machines used for ESGF.

## Preamble : when should you reboot ?

Most machines that you will see listed in this documentation are production machines. That means they are solicited all thorought the day, and as such you should try rebooting them as rarely as possible. When you do reboot them, make sure that they are out of service for as little time as possible. You should also make sure to reboot them at a time when few users use them ; for instance, at 18:00 CEST.

With great power comes great responsibility. If you need to plan ahead of a power cut, be sure to refer yourself to the page on what to do in case of power cuts and to send emails warning of the power cuts at least a few days in advance.

## GridFTP machine

At IPSL, there is one "bare-metal" machine called `gridftp.ipsl.upmc.fr`. There is no point in accessing it with anything else than root, so you can just ssh into it as root.

This machine has the sole purpose of hosting the GridFTP service. Since GridFTP tends to occupy all the processors and fill the ram easily, we found it best to implement it in a separate machine that runs on bare metal to optimize resource usage.

This means a few things :

* You can’t see this machine on vcenter, since it’s not a virtual machine. That means that if you type a `poweroff` by accident, you can’t start it again from a distance, and you will have to ask the infrastructure team to restart it. This also goes for cases of power cuts. In that case, however, the infrastructure team will most likely start the machine again when the cut is over.
* You don’t have access to snapshots or backups in the same way a VM has. Be extra careful.

Keeping this in mind, there might be cases where you want to reboot GridFTP. In this case, you will need to start the `globus-gridftp-server` service upon restarting it. Here is how you do it :

```bash
service globus-gridftp-server start
```

We also strongly recommend to run the esgf-test-suite in production right after that. Here is the command for production, but you can find it again in the `crontab` of esgf-watch-dog on the esgf-monitoring machine.

```bash
'/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user' 2>&1
```

If you see that all tests (and more specifically `dl_gridftp`) succeed, that means everything is runnig correctly.

## ESGF nodes

It’s important to stop the ESGF nodes before rebooting. This is done via the esgf-monitoring machine. Here are a few examples, that you can adapt to what you need :

```bash
esgf prod stop # stops BOTH production nodes
esgf x-data stop # stops the data node at polytechnique
esgf int-index stop # stops only the integration index node
```

After you rebooted your machines, you can launch the corresponding esgf command to restart your nodes. Here are a few examples :

```bash
esgf prod start # stops BOTH production nodes
esgf x-data start # stops the data node at polytechnique
esgf int-index start # stops only the integration index node
```

## Hermes and Synda machines

There is no particular precaution here, but before restarting the nodes you should ask the people responsible for the nodes (Mark Greenslade for Hermes and Atef Bennasser for Synda nodes, respectively). They work on these machines.

## copernicus-wps (/cmip-wps)

When restarting the machine, you should also rerun the `ansible-wps-playbook`.
