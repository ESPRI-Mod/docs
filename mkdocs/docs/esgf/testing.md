Testing the ESGF suite
=============

* Version: 0.0.1
* Date: 9/9/2019
* Authors: Pierre Logerais
* Keywords: tests esgf testing

## Description

This procedure describes how to test if ESGF is running correctly, both on ESGF 2.x and 4.0.x. 

### Access

At IPSL, the only requirement is to have an access to the account esgf-watch-dog@esgf-monitoring.

## Procedure

### From esgf-monitoring

* Check if your ESGF node is running

The command is `esgf <groupe>-<nœud> status`.
`groupe` can be set to `int`, `dev` or `prod`. 
`nœud` means the exact node you want to check. The 3 accepted values for this parameter are `index`, `idp` ou `data`. This parameter is optional, and if you omit it the test suite will check every node in that group.

For example, if you want to have the status of the data node in production :

```bash
esgf prod-data status
```

If you want to have the status of both integration nodes :

```bash
esgf int status
```

* Restart nodes

The command is the same as above, but with `restart` at the end.

For example, if you want to restart the data node in production :

```bash
esgf prod-data status
```

If you want to restart both integration nodes :

```bash
esgf int status
```

* More detailed tests

Normally, there are tests that run every hour and do multiple specific tests on nodes. They are disabled for now, but this should be reactivated after the migrations to CentOS 7 and ESGF 4.0.4.

These tests are bash scripts, that run a singularity container made specifically for tests. This container has python3 and launches nose tests with python3.

You can find them in the crontab : the command `crontab -l` on esgf-monitoring returns the following :

```bash
[esgf-watch-dog@esgf-monitoring ~]$ crontab -l
# m    h    dom  mon  dow   command
#  0   *     *    *    *    /home/esgf-watch-dog/scripts/script_bootstrap.sh '[ESGF-TEST-SUITE] prod'   'sgardoll@ipsl.fr,glipsl@ipsl.fr,sdipsl@ipsl.jussieu.fr,plogerais@ipsl.fr' '/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user' 2>&1 | /usr/bin/logger -t '[ESGF-TEST-SUITE] prod'
#  15   *     *    *    *    /home/esgf-watch-dog/scripts/script_bootstrap.sh '[ESGF-TEST-SUITE] int'    'sgardoll@ipsl.fr,glipsl@ipsl.fr,sdipsl@ipsl.jussieu.fr,plogerais@ipsl.fr' '/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_int.ini' 2>&1 | /usr/bin/logger -t '[ESGF-TEST-SUITE] int'
#  30   *     *    *    *    /home/esgf-watch-dog/scripts/script_bootstrap.sh '[ESGF-TEST-SUITE] dev'    'sgardoll@ipsl.fr,glipsl@ipsl.fr,sdipsl@ipsl.jussieu.fr,plogerais@ipsl.fr' '/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_dev.ini' 2>&1 | /usr/bin/logger -t '[ESGF-TEST-SUITE] dev'
#  45   *     *    *    *    /home/esgf-watch-dog/scripts/script_bootstrap.sh '[ESGF-TEST-SUITE] vesgx'  'sgardoll@ipsl.fr,glipsl@ipsl.fr,sdipsl@ipsl.jussieu.fr,plogerais@ipsl.fr' '/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_vesgx.ini -a data,!dl_gridftp' 2>&1 | /usr/bin/logger -t '[ESGF-TEST-SUITE] vesgx'
```

Let’s dissect these commands :

- `script_bootstrap.sh` exists solely to send status emails informing about potential errors.
- `esgf-test-suite.sh` is the actual test. It takes configuration written in `my_config_<group>.ini` and uses it to test all the nodes in that group.
- `2>&1 | /usr/bin/logger` sends all the logs (be it from stdout or stderr) to the system logs, just in case it’s needed.

If you want to run these tests manually, you can ignore the bootstrap script and the logger part. Hence, the command would look like this :

```bash
'/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user' 
```

There are flags to not try out `compute` and `cog_create_user`. This is because these tests always fail.

Here are results of a successful test in production, as an example :

```bash
[esgf-watch-dog@esgf-monitoring ~]$ '/home/esgf-watch-dog/scripts/check_esgf-test-suite.sh' '/home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user'
> checking with /home/esgf-watch-dog/test-suite_config_files/my_config_prod.ini -a !compute,!cog_create_user
cog_root_login ... ok
cog_user_login ... ok
basic_ping (https://vesg.ipsl.upmc.fr/thredds) ... ok
basic_ping (https://vesg.ipsl.upmc.fr/esg-orp/home.htm) ... ok
dl_gridftp ... url downloaded: gsiftp://gridftp.ipsl.upmc.fr:2811//cmip6/DCPP/IPSL/IPSL-CM6A-LR/dcppC-pac-pacemaker/s1920-r10i1p1f1/Emon/sconcdust/gr/v20190110/sconcdust_Emon_IPSL-CM6A-LR_dcppC-pac-pacemaker_s1920-r10i1p1f1_gr_192001-201412.nc
ok
dl_http ... url downloaded: http://vesg.ipsl.upmc.fr/thredds/fileServer/cmip6/DCPP/IPSL/IPSL-CM6A-LR/dcppC-pac-pacemaker/s1920-r10i1p1f1/Emon/sconcdust/gr/v20190110/sconcdust_Emon_IPSL-CM6A-LR_dcppC-pac-pacemaker_s1920-r10i1p1f1_gr_192001-201412.nc
ok
basic_ping (https://esgf-node.ipsl.upmc.fr/esgf-idp) ... ok
basic_ping (https://esgf-node.ipsl.upmc.fr/) ... ok
basic_ping (https://esgf-node.ipsl.upmc.fr/solr/#) ... ok
basic_ping (https://esgf-node.ipsl.upmc.fr/esg-search/search) ... ok
basic_ping (https://esgf-node.ipsl.upmc.fr/esg-orp/home.htm) ... ok
myproxy_get_credentials ... ok
myproxy_get_trustroots ... ok
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

----------------------------------------------------------------------
Ran 25 tests in 32.551s

OK
```

### From the node itself

* ESGF-legacy (versions 2.x)

As root, run the following command :

```bash
esg-node status
```

Here is an example of a successful run :

```bash
[root@vesg ~]# esg-node status


  EEEEEEEEEEEEEEEEEEEEEE   SSSSSSSSSSSSSSS         GGGGGGGGGGGGGFFFFFFFFFFFFFFFFFFFFFF
  E::::::::::::::::::::E SS:::::::::::::::S     GGG::::::::::::GF::::::::::::::::::::F
  E::::::::::::::::::::ES:::::SSSSSS::::::S   GG:::::::::::::::GF::::::::::::::::::::F
  EE::::::EEEEEEEEE::::ES:::::S     SSSSSSS  G:::::GGGGGGGG::::GFF::::::FFFFFFFFF::::F
    E:::::E       EEEEEES:::::S             G:::::G       GGGGGG  F:::::F       FFFFFF
    E:::::E             S:::::S            G:::::G                F:::::F
    E::::::EEEEEEEEEE    S::::SSSS         G:::::G                F::::::FFFFFFFFFF
    E:::::::::::::::E     SS::::::SSSSS    G:::::G    GGGGGGGGGG  F:::::::::::::::F
    E:::::::::::::::E       SSS::::::::SS  G:::::G    G::::::::G  F:::::::::::::::F
    E::::::EEEEEEEEEE          SSSSSS::::S G:::::G    GGGGG::::G  F::::::FFFFFFFFFF
    E:::::E                         S:::::SG:::::G        G::::G  F:::::F
    E:::::E       EEEEEE            S:::::S G:::::G       G::::G  F:::::F
  EE::::::EEEEEEEE:::::ESSSSSSS     S:::::S  G:::::GGGGGGGG::::GFF:::::::FF
  E::::::::::::::::::::ES::::::SSSSSS:::::S   GG:::::::::::::::GF::::::::FF
  E::::::::::::::::::::ES:::::::::::::::SS      GGG::::::GGG:::GF::::::::FF
  EEEEEEEEEEEEEEEEEEEEEE SSSSSSSSSSSSSSS           GGGGGG   GGGGFFFFFFFFFFF.llnl.gov

Checking that you have root privs on vesg.ipsl.upmc.fr... [OK]
Checking requisites... 

postmaster (pid  9111) en cours d'exécution...
 WARNING: There is another process running on expected Tomcat (jsvc) ports!!!! [java] ?? 
java     9187   root   49u  IPv6 11830114      0t0  TCP *:8080 (LISTEN)
java     9187   root   54u  IPv6 11830118      0t0  TCP *:8443 (LISTEN)
java     9187   root   59u  IPv6 11830145      0t0  TCP *:8223 (LISTEN)

Stopped: At least one process not running

---------------------------
Running Node Services... 
node type: [ data ] (4) 
---------------------------
postmaste  9111 postgres    4u  IPv4 11829879      0t0  TCP *:5432 (LISTEN)
postmaste  9111 postgres    6u  IPv6 11829880      0t0  TCP *:5432 (LISTEN)
java       9187     root   49u  IPv6 11830114      0t0  TCP *:8080 (LISTEN)
java       9187     root   54u  IPv6 11830118      0t0  TCP *:8443 (LISTEN)
java       9187     root   59u  IPv6 11830145      0t0  TCP *:8223 (LISTEN)
java       9187     root   85u  IPv6 11831307      0t0  TCP 127.0.0.1:8005 (LISTEN)
httpd      8986     root    6u  IPv6 11829083      0t0  TCP *:80 (LISTEN)
httpd      8986     root    8u  IPv6 11829091      0t0  TCP *:443 (LISTEN)
esgf-dash 9276
---------------------------

```

This process can be run both on data and on index nodes.

* For ESGF-Ansible (ESGF 4.0.x)

DO NOT use the `esg-node` command.
**FIXME**

### From nagios

The [nagios dashboard](https://nagios-ng.ipsl.upmc.fr/nagios/) lets you review quickly if tests are successful or not.
