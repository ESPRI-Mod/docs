Maintenance cases
=========================

* Version: 0.0.1
* Date: 12/11/2019
* Authors: Pierre Logerais
* Keywords: esgf maintenance

## Description

This procedure describes what to do in case of a maintenance (eg. power being cut off, forcing a shutdown of ciclad…), in ESGF nodes most notably.


* ESGF-Ansible version: any
* License: as ESGF
* Repository: [github](https://github.com/ESGF/esgf-ansible)
* Documentation: [ESGF-Ansible wiki](https://esgf.github.io/esgf-ansible/)

## Hosts & access

### Hosts

esgf-monitoring.ipsl.upmc.fr

### Access

* Protocol: ssh
* Login: esgf-watch-dog
* Command: ssh esgf-watch-dog@esgf-monitoring.ipsl.upmc.fr

## Environment

### Requirements

A Linux machine.

## Procedure

### In case of a power cut

You should be warned a few days in advance, and the cut should not last more than a full day.

The first step is to warn the users of ESGF about the power cut. Issue your warnings as many days in advance as the power cut will take at least (example : if power is cut for a day, send information a day in advance).

#### Email

Write to the following mailing lists : `esci@listes.ipsl.fr, ipslcm6_dev@listes.ipsl.fr, ipsl-cmip6@listes.ipsl.fr`.

Here is an example of an email you could write :

```
Bonjour à tous,

En anticipation de la coupure électrique qui intervient sur Jussieu jusqu'à demain après-midi, le noeud vesg.ipsl.upmc.fr vient d'être éteint.

Les accès thredds sont donc temporairement interrompus.
```

You should also write an email to the users of ESGF (esgf-users@listes.ipsl.fr) that would look like this :


```
Dear ESGF users,

Please apologies for this late notification.
The IPSL ESGF node is down for electrical shutdown until tomorrow afternoon.

The IPSL CMIP and CORDEX data will be so temporarily unavailable.
Please use the "Show Replica" on the other ESGF front-ends.
```

It’s important to note that this message should at least have an english version, because the esgf users might include postdocs who are not francophones.

#### Shutting down the ESGF nodes

A few hours before the power cut, the ESPRI-INFRA team will shut down ciclad, along with every VM. Before that, you need to :

* Shut down ESGF
* Turn off nagios notifications for all machines
* Turn off esgf-test-suite

Log in on the esgf-monitoring machine, and edit the crontab to comment every line that runs esgf test suite tests.

Then launch the following commands :

```bash
esgf prod stop
esgf cds stop
esgf int stop
esgf dev stop
```

On the nagios page, disable notifications for every test on every VM. This makes it so that Nagios doesn’t spam anyone with emails.

#### When the power comes back up

Don’t forget to :

* Restart the esgf nodes :

```bash
esgf prod start 
esgf cds start
esgf int start
esgf dev start
```

* Turn on the cron jobs again for esgf-test-suite
* Turn on notifications from nagios again

### In case of an overcharged production node

If RAM is almost fully used (like 90% or more) on the index node, it means that there are a lot of requests. It’s not necessarily bad, but you should always check that the `/prodigfs` filesystem is mounted and responding, by accessing subdirectories in `/prodigfs`. If it’s available, then you should wait and see, and if the problem doesn’t subside by itself, restart the node during the evening or the night.

If the data node is overcharged, it means that there are lots of downloads. If the load average is very high, it means that there are lots of pending downloads and it could be an issue with the mounted filesystems on `/prodigfs`. Again, you should check it by accessing subdirectories in `/prodigfs`. If the mounted filesystems are responding just fine, keep an eye out and restart only during the evening or the night.
