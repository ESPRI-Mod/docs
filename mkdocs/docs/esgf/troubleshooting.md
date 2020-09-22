
TROUBLESHOOTING
============================

* Version: 0.0.5
* Date: 03/09/2019
* Authors: Pierre Logerais
* Keywords: migration centos esgf index data node

## Description

This procedure describes common issues related to migrating or running an ESGF node, and how to fix them.

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

* esgf-test-suite no longer passes for an obscure reason and takes a very long time to execute

If the test-suite takes 20+ minutes for no reason, but there doesn’t appear to be any problem with the node itself, it might be due to a problem with many instances of the test-suite running. They slow each other down drastically.

Run `glances` and `nload` on both the data and index node to be sure that the problem isn’t an overloaded machine.

If this isn’t the case, you can see the test-suites running with the commands :

```ps aux | grep test-suite
ps aux | grep python3```

If there are multiple instances of the test-suite running, you could see an output like this one :

```(base) [esgf-watch-dog@esgf-monitoring ~]$ ps aux | grep test-suite
esgf-wa+  2419  0.0  0.0 112832   972 pts/0    S+   15:45   0:00 grep --color=auto test-suite
esgf-wa+ 21030  0.0  0.0 113284     0 ?        Ss   09:00   0:00 /bin/sh -c /home/esgf-watch-dog/scripts/script_bootstrap.sh '[ESGF-TEST-SUITE] prod'   'glipsl@ipsl.fr,plogerais@ipsl.fr' '/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user' 2>&1 | /usr/bin/logger -t '[ESGF-TEST-SUITE] prod'
esgf-wa+ 21032  0.0  0.0 113284    36 ?        S    09:00   0:00 /bin/bash /home/esgf-watch-dog/scripts/script_bootstrap.sh [ESGF-TEST-SUITE] prod glipsl@ipsl.fr,plogerais@ipsl.fr /home/esgf-watch-dog/scripts/check_esgf-test-suite.sh /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 21043  0.0  0.0 113284    12 ?        S    09:00   0:00 /bin/bash /home/esgf-watch-dog/scripts/script_bootstrap.sh [ESGF-TEST-SUITE] prod glipsl@ipsl.fr,plogerais@ipsl.fr /home/esgf-watch-dog/scripts/check_esgf-test-suite.sh /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 21044  0.0  0.0 113288    36 ?        S    09:00   0:00 /bin/bash /home/esgf-watch-dog/scripts/check_esgf-test-suite.sh /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 21045  0.0  6.6 1047036 258796 ?      Sl   09:00   0:16 python3 esgf-test.py -v --nocapture --nologcapture --tc-file /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 21320  0.0  6.9 1047036 269316 ?      Sl   09:01   0:01 python3 esgf-test.py -v --nocapture --nologcapture --tc-file /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 21321  0.0  6.9 973304 269252 ?       S    09:01   0:00 python3 esgf-test.py -v --nocapture --nologcapture --tc-file /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 24755  0.0  0.0 113284   180 ?        Ss   11:00   0:00 /bin/sh -c /home/esgf-watch-dog/scripts/script_bootstrap.sh '[ESGF-TEST-SUITE] prod'   'glipsl@ipsl.fr,plogerais@ipsl.fr' '/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user' 2>&1 | /usr/bin/logger -t '[ESGF-TEST-SUITE] prod'
esgf-wa+ 24756  0.0  0.0 113284   252 ?        S    11:00   0:00 /bin/bash /home/esgf-watch-dog/scripts/script_bootstrap.sh [ESGF-TEST-SUITE] prod glipsl@ipsl.fr,plogerais@ipsl.fr /home/esgf-watch-dog/scripts/check_esgf-test-suite.sh /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 24768  0.0  0.0 113284   256 ?        S    11:00   0:00 /bin/bash /home/esgf-watch-dog/scripts/script_bootstrap.sh [ESGF-TEST-SUITE] prod glipsl@ipsl.fr,plogerais@ipsl.fr /home/esgf-watch-dog/scripts/check_esgf-test-suite.sh /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 24769  0.0  0.0 113288   228 ?        S    11:00   0:00 /bin/bash /home/esgf-watch-dog/scripts/check_esgf-test-suite.sh /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 24770  0.0 20.9 1047700 814036 ?      Sl   11:00   0:15 python3 esgf-test.py -v --nocapture --nologcapture --tc-file /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 25426  0.0 21.0 973968 818340 ?       S    11:20   0:00 python3 esgf-test.py -v --nocapture --nologcapture --tc-file /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 25427  0.0 21.0 973968 818496 ?       S    11:20   0:00 python3 esgf-test.py -v --nocapture --nologcapture --tc-file /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 26241  0.0  0.0 113284   180 ?        Ss   12:00   0:00 /bin/sh -c /home/esgf-watch-dog/scripts/script_bootstrap.sh '[ESGF-TEST-SUITE] prod'   'glipsl@ipsl.fr,plogerais@ipsl.fr' '/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user' 2>&1 | /usr/bin/logger -t '[ESGF-TEST-SUITE] prod'
esgf-wa+ 26242  0.0  0.0 113284   240 ?        S    12:00   0:00 /bin/bash /home/esgf-watch-dog/scripts/script_bootstrap.sh [ESGF-TEST-SUITE] prod glipsl@ipsl.fr,plogerais@ipsl.fr /home/esgf-watch-dog/scripts/check_esgf-test-suite.sh /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 26253  0.0  0.0 113284   256 ?        S    12:00   0:00 /bin/bash /home/esgf-watch-dog/scripts/script_bootstrap.sh [ESGF-TEST-SUITE] prod glipsl@ipsl.fr,plogerais@ipsl.fr /home/esgf-watch-dog/scripts/check_esgf-test-suite.sh /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 26254  0.0  0.0 113288   232 ?        S    12:00   0:00 /bin/bash /home/esgf-watch-dog/scripts/check_esgf-test-suite.sh /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 26255  0.1 21.2 1045856 826072 ?      Sl   12:00   0:20 python3 esgf-test.py -v --nocapture --nologcapture --tc-file /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 26884  0.0 21.3 972124 828144 ?       S    12:20   0:00 python3 esgf-test.py -v --nocapture --nologcapture --tc-file /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
esgf-wa+ 26885  0.0 21.3 972124 828144 ?       S    12:20   0:00 python3 esgf-test.py -v --nocapture --nologcapture --tc-file /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
(base) [esgf-watch-dog@esgf-monitoring ~]$ ```

In that case, either reboot `esgf-monitoring` or run :

```killall check_esgf-test-suite.sh
killall python3```

The killed test-suite instances will still send an empty error e-mail, but it’s nothing to worry about.

Rerun the test-suite afterwards to be sure that everything is fine.


### PostgreSQL and semaphores

Sometimes, PostgreSQL will not be able to restart because there are too many semaphores. The command `systemctl status postgresql.service` will show the following :

```vesg.ipsl.upmc.fr pg_ctl[3027]: HINT: This error does *not* mean that you have run out of disk space. It occurs when either the system limit for the maximum number of semaphore sets (SEMMNI), or the system wide maximum number of semaphores (SEMMNS)```

In that case, rebooting is a way to fix this issue.

### ESGF-Test-suite’s cog login error

One error can happen, where the test-suite returns such error message :

```
> checking with /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
cog_root_login ... ok
cog_user_login ... ok
basic_ping (https://vesg.ipsl.upmc.fr/thredds/catalog/catalog.html) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esg-orp/home.htm) ... ok
ERROR
dl_http ... ok
basic_ping (https://esgf-node.ipsl.upmc.fr/esgf-idp) ... ok
basic_ping (https://esgf-node.ipsl.upmc.fr/) ... ok
basic_ping (https://esgf-node.ipsl.upmc.fr/solr/#) ... ok
basic_ping (https://esgf-node.ipsl.upmc.fr/esg-search/search) ... ok
basic_ping (https://esgf-node.ipsl.upmc.fr/esg-orp/home.htm) ... ok
myproxy_get_credentials ... SKIP: unable to check the credentials
myproxy_get_trustroots ... SKIP: unable to check the trustroots
slcs_django_admin_login ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/cmip5/stats-by-space/xml) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/cmip5/stats-by-dataset/xml) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/cmip5/stats-by-experiment/xml) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/cmip5/stats-by-model/xml) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/cmip5/stats-by-variable/xml) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/obs4mips/stats-by-space/xml) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/obs4mips/stats-by-dataset/xml) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/obs4mips/stats-by-realm/xml) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/obs4mips/stats-by-source/xml) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/obs4mips/stats-by-variable/xml) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esgf-stats-api/cross-project/stats-by-time/xml) ... ok

======================================================================
url downloaded: http://vesg.ipsl.upmc.fr/thredds/fileServer/cmip6/DCPP/IPSL/IPSL-CM6A-LR/dcppC-amv-Trop-neg/r9i1p1f1/Amon/tauu/gr/v20190110/tauu_Amon_IPSL-CM6A-LR_dcppC-amv-Trop-neg_r9i1p1f1_gr_185001-185912.nc
IndexError: list index out of range```


This is due to an error with the login in CoG. Check that you have the correct login by logging into https://esgf-node.ipsl.upmc.fr/. 
