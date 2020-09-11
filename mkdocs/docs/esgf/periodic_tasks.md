PERIODIC TASKS
==============

* Version: 0.0.1
* Date: 20/09/2019
* Authors: Sébastien Gardoll
* Keywords: cron task automatic script

## Description

This document describes a set of tasks periodically executed on different systems.

!!! note
    All the scripts are versioned in the ESPRI-MOD' Subversion repository and located in esgf_admin_tools/trunk/scripts

### esgf-watch-dog@esgf-monitoring.ipsl.upmc.fr

this system runs ESGF-Test-Suite tasks:

```cron
# m    h    dom  mon  dow   command
#  0   *     *    *    *    /home/esgf-watch-dog/scripts/script_bootstrap.sh '[ESGF-TEST-SUITE] prod'   'glipsl@ipsl.fr,plogerais@ipsl.fr' '/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user' 2>&1 | /usr/bin/logger -t '[ESGF-TEST-SUITE] prod'
#  15   *     *    *    *    /home/esgf-watch-dog/scripts/script_bootstrap.sh '[ESGF-TEST-SUITE] int'    'glipsl@ipsl.fr,plogerais@ipsl.fr' '/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_int.ini' 2>&1 | /usr/bin/logger -t '[ESGF-TEST-SUITE] int'
```

### vesg.ipsl.upmc.fr

!!! warning
    Periodic tasks on this machine are currently disabled.

This system runs two cron tasks:

```cron
#Stats script
0 */2 * * * /root/scripts/check_stats.sh

#Dashboard
0 * * * * /root/scripts/check_injection.sh 2>&1 | tee -a /root/stats/check_injection.log | /usr/bin/logger -t '[CHECK_INJECTION]'

#Logger
*/30 * * * * top -b -n1 >> /root/stats/top.log
0,30 * * * *   /root/miniconda2/envs/sandbox/bin/python /root/scripts/esgf_stats.py 2>&1 |/usr/bin/logger -t '[STATS]'
```

- check_stats.sh

This script sends an email when we can inject dashboard queue entries because there are
less then 50 unprocessed entries (dashboard migration).

- check_injection.sh

This scripts logs the dashboard processing.

- top

Log the process activity on the system.

- esgf_stats.py

This script computes some basic statistics about ESGF data node.

### esgf-node.ipsl.upmc.fr

There is one script that runs every full hour to restart solr if it dies. It’s on the root crontab on esgf-node :

```cron
# m    h    dom  mon  dow   command
  0   *     *    *    *    python /root/scripts/restart-died-solr.py
```

This script doesn’t do anything if solr works correctly.

It was written by Alan Iwi, and the github is here : https://gist.github.com/alaniwi/a8f1f6943729b63377311c71947b0e9d

### Hermes machines

On Hermes machines, some of the hermes'services are started, at boot, with special care.

- On db machines: services are started without any help.

- On mq machines, services are started following these instructions:

```bash
source /opt/hermes/activate
hermes-mq-daemons-start
```

scripts/restart_hermes_mq_services.sh encapsulates these instructions and provides
error handling mechanism (send an email if the script is not able to start up the 
service). The script is executed from /etc/rc.local (specific to CentOS 6).

involved machines: hermes-mq-test.private.ipsl.fr and hermes-mq-prod.private.ipsl.fr

- On web machines, services are started following these instructions:

```bash
source /opt/hermes/activate
hermes-web-daemon-start
```

scripts/restart_hermes_web_services.sh encapsulates these instructions and provides
error handling mechanism (send an email if the script is not able to start up the 
service). The script is executed from /etc/rc.local (specific to CentOS 6).

involved machines: hermes.ipsl.upmc.fr and hermes-test.ipsl.upmc.fr