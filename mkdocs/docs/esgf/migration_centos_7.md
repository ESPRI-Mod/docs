MIGRATION FROM CENTOS 6 to 7
============================

* Version: 0.0.3
* Date: 03/09/2019
* Authors: Sébastien Gardoll, Pierre Logerais
* Keywords: migration centos backup esgf index data node

## Description

This procedure describes how to migrate a ESGF node 4.0.4 from CentOS 6 to CentOS 7.

Some definitions:

index-old: index node before migrating (under CentOS 6)

data-old: data node before migrating (under CentOS 6)

index-new: index node after migrating (under CentOS 7)

data-new: data node after migrating (under CentOS 7)

## Procedure

### 1. Create the *-new VMs with temporary host name

### 2. Shutdown the *-new VMs

### 3. Rename the host name of the *-old VMs

### 4. Boot the *-new VMs and rename their host name

### 5. Backup **index-old** (destination is the index-new)

* Tar balls

```bash
HOST_DEST='esgf-node.ipsl.upmc.fr'
mkdir /root/migration_backup
cd /root/migration_backup
tar -pcJf esgf_config.tar.xz -C /esg config
mkdir certs
cp -rp /etc/certs certs
cp -rp /etc/esgfcerts certs
cp -rp /etc/grid-security certs
cp -rp /var/lib/globus/simple_ca certs
tar -pcJf - certs | openssl enc -e -aes256 -out certs.tar.xz.enc # Provide a password.
rm -fr certs

ssh "root@${HOST_DEST}" 'mkdir /root/pre_migration_backup'
ssh "root@${HOST_DEST}" 'chmod go= /root/pre_migration_backup'
scp /root/migration_backup/* "root@{HOST_DEST}:/root/pre_migration_backup"

tar -pcJf home.tar.xz -C / home
cp -p /root/.bashrc .
cp -p /root/.pgpass .
```

* Cog

```bash
mkdir /root/migration_backup
cd /root/migration_backup
tar -pcJf cog.tar.xz -C /usr/local cog
```

* Solr indices

```bash
mkdir /root/migration_backup
cd /root/migration_backup
tar -pcJf solr-index.tar.xz -C /esg solr-index
```

* PostgreSQL

```bash
cd /root/migration_backup
pg_dump -U dbsuper --clean -Z 6 -v -F c esgcet > db_esgcet.bak 2> db_esgcet_backup.log || echo '***** ERROR *****'
pg_dump -U dbsuper --clean -Z 6 -v -F c cogdb > db_cogdb.bak 2> db_cogdb_backup.log || echo '***** ERROR *****'
pg_dump -U dbsuper --clean -Z 6 -v -F c slcsdb > db_slcsdb.bak 2>db_slcsdb_backup.log || echo '***** ERROR *****'
pg_dump -U dbsuper --clean -Z 6 -v -F c postgres > db_postgres.bak 2>db_postgres_backup.log || echo '***** ERROR *****'
```

* Transfert

```bash
HOST_DEST='esgf-node.ipsl.upmc.fr'
ssh "root@${HOST_DEST}" 'mkdir /root/migration_backup'
ssh "root@${HOST_DEST}" 'chmod go= /root/migration_backup'
scp /root/migration_backup/* "root@{HOST_DEST}:/root/migration_backup"
```

### 6. Backup **data-old** (destination is the data-new)

* Tar balls

```bash
HOST_DEST='vesg.ipsl.upmc.fr'
mkdir -p /root/migration_backup
cd /root/migration_backup
tar -pcJf esgf_config.tar.xz -C /esg config
mkdir certs
cp -rp /etc/certs certs
cp -rp /etc/esgfcerts certs
cp -rp /etc/grid-security certs
tar -pcJf - certs | openssl enc -e -aes256 -out certs.tar.xz.enc # Provide a password.
rm -fr certs

ssh "root@${HOST_DEST}" 'mkdir /root/pre_migration_backup'
ssh "root@${HOST_DEST}" 'chmod go= /root/pre_migration_backup'
scp /root/migration_backup/* "root@{HOST_DEST}:/root/pre_migration_backup"

tar -pcJf home.tar.xz -C / home
cp -p /root/.bashrc .
cp -p /root/.pgpass .
```

* Thredds

```bash
mkdir -p /root/migration_backup
cd /root/migration_backup
tar -pcJf thredds.tar.xz -C /esg/content thredds
```

* PostgreSQL

```bash
mkdir -p /root/migration_backup
cd /root/migration_backup
pg_dump -U dbsuper --clean -Z 6 -v -F c esgcet > db_esgcet.bak 2> db_esgcet_backup.log || echo '***** ERROR *****'
pg_dump -U dbsuper --clean -Z 6 -v -F c postgres > db_postgres.bak 2>db_postgres_backup.log || echo '***** ERROR *****'
```

* Transfert

```bash
HOST_DEST='vesg.ipsl.upmc.fr'
ssh "root@${HOST_DEST}" 'mkdir /root/migration_backup'
ssh "root@${HOST_DEST}" 'chmod go= /root/migration_backup'
scp /root/migration_backup/* "root@{HOST_DEST}:/root/migration_backup"
```

### 7. Install from scratch ESGF 4.0.4 on the *-new VMs

#### Pre installation

* Set the same passwords

```bash
tar -C /root/pre_migration_backup -xapf /root/pre_migration_backup/esgf_config.tar.xz
mkdir -p /esg/config
cp -p /root/pre_migration_backup/config/.esgf_pass 
```

* Certs

On *-new VMs (both !)

```bash
openssl enc -d -aes256 -in /root/pre_migration_backup/certs.tar.xz.enc | tar -C /root -xap # Provide a password.
chown -R root:root /root/certs ; chmod -R go= /root/certs
```

#### Installation setup

- Set the host_var files so as to pick up the certificate files.

For the index-new (esgf-node.ipsl.upmc.fr):

```
globushostcert: /root/certs/grid-security/hostcert.pem
globushostkey: /root/certs/grid-security/hostkey.pem


myproxycacert: /root/certs/esgfcerts/cacert.pem
myproxycakey: /root/certs/esgfcerts/cakey.pem
myproxy_signing_policy: /root/certs/esgfcerts/globus_simple_ca_85bc937c_setup-0/85bc937c.signing_policy

hostkey_src: /root/certs/certs/hostkey.pem
hostcert_src: /root/certs/certs/hostcert.pem
cachain_src: /root/certs/certs/cachain.pem
```

For the data-new (vesg.ipsl.upmc.fr.yml):

```
globushostcert: /root/certs/grid-security/hostcert.pem
globushostkey: /root/certs/grid-security/hostkey.pem

hostkey_src: /root/certs/certs/hostkey.pem
hostcert_src: /root/certs/certs/hostcert.pem
cachain_src: /root/certs/certs/cachain.pem
```

**Don't forget to setup the globus variables for both files**

- On esgf-watch-dog@esgf-monitoring:

```bash
script install.log
cd /home/esgf-watch-dog/esgf-ansible
git checkout 4.0.4
git status
source activate ansible
export ANSIBLE_NOCOLOR=true
ansible-playbook -i hosts.prod -u root --skip-tags gridftp install.yml
```

#### Post installation

- Configuration of authorization and policies (Guillaume)

- Some commands:

```bash
chown root:tomcat /esg/config/.esgf_pass
rm -fr /root/pre_migration_backup
```

- follow the post installation section of the upgrade procedure.

### 8. Install the data for **index-new**

* Shutdown ESGF node on index-new

* Simple files

```bash
mv /root/migration_backup/.pgpass /root
chmod go= /root/.pgpass
```

* Cog

```bash
mkdir -p /root/fresh_install_backup
mv /usr/local/cog /root/fresh_install_backup && tar -C /usr/local -xapf /root/migration_backup/cog.tar.xz
```

* Solr indices

```bash
mkdir -p /root/fresh_install_backup
mv /esg/solr-index /root/fresh_install_backup && tar -C /esg -xapf /root/migration_backup/solr-index.tar.xz
```

* PostgreSQL

Backup the fresly installed db's
```bash
mkdir -p /root/fresh_install_backup
cd /root/fresh_install_backup
pg_dump -U dbsuper --clean -Z 6 -v -F c esgcet > db_esgcet.bak 2> db_esgcet_backup.log || echo '***** ERROR *****'
pg_dump -U dbsuper --clean -Z 6 -v -F c cogdb > db_cogdb.bak 2> db_cogdb_backup.log || echo '***** ERROR *****'
pg_dump -U dbsuper --clean -Z 6 -v -F c slcsdb > db_slcsdb.bak 2>db_slcsdb_backup.log || echo '***** ERROR *****'
pg_dump -U dbsuper --clean -Z 6 -v -F c postgres > db_postgres.bak 2>db_postgres_backup.log || echo '***** ERROR *****'
```

Inject the data from index-old
```bash
cd /root/migration_backup
pg_restore --clean -U dbsuper -d esgcet -v -F c db_esgcet.bak 2>db_esgcet_injec.log || echo '***** ERROR *****'
pg_restore --clean -U dbsuper -d cogdb  -v -F c db_cogdb.bak 2>db_cogdb_injec.log || echo '***** ERROR *****'
pg_restore --clean -U dbsuper -d slcsdb -v -F c db_slcsdb.bak 2>db_slcsdb_injec.log || echo '***** ERROR *****'
```

* Modify /root/.bashrc according to /root/migration_backup/.bashrc

### 9. Test Solr indices for **index-new**

### 10. Install the data for **data-new**

* Shutdown ESGF node on data-new

* Simple files

```bash
mv /root/migration_backup/.pgpass /root
chmod go= /root/.pgpass
```

* PostgreSQL

Backup the fresly installed db's
```bash
mkdir -p /root/fresh_install_backup
cd /root/fresh_install_backup
pg_dump -U dbsuper --clean -Z 6 -v -F c esgcet > db_esgcet.bak 2> db_esgcet_backup.log || echo '***** ERROR *****'
pg_dump -U dbsuper --clean -Z 6 -v -F c postgres > db_postgres.bak 2>db_postgres_backup.log || echo '***** ERROR *****'
```

Inject the data from data-old
```bash
cd /root/migration_backup
pg_restore --clean -U dbsuper -d esgcet -v -F c db_esgcet.bak 2>db_esgcet_injec.log || echo '***** ERROR *****'
```

* Thredds

```bash
mkdir -p /root/fresh_install_backup
mv /esg/content/thredds /root/fresh_install_backup && tar -C /esg/content -xapf /root/migration_backup/thredds.tar.xz
```

* Modify /root/.bashrc according to /root/migration_backup/.bashrc

### 11. Configure the iptables for **data-new**

```bash
systemctl stop firewalld
systemctl disable firewalld
yum -y install iptables-services
systemctl start iptables-services
systemctl enable iptables-services
iptables -A INPUT -i eth0 -m state --state ESTABLISHED -j ACCEPT
iptables -A INPUT -i eth0 -m state --state NEW -s 66.249.64.0/20 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 37.9.113.0/24 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 54.36.148.0/22 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 40.76.0.0/14 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 13.64.0.0/11 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 207.46.0.0/16 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 157.56.0.0/14 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 141.8.128.0/18 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 178.154.128.0/17 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 37.9.64.0/18 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 213.180.192.0/19 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 5.45.192.0/18 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 95.108.128.0/17 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 17.0.0.0/8 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 52.224.0.0/11 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 106.36.0.0/15 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 106.38.0.0/15 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 5.255.253.0/24 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 157.55.39.0/24 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 87.250.224.0/24 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 91.242.162.0/24 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 51.255.65.0/24 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 180.76.15.0/24 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i eth0 -m state --state NEW -s 207.46.13.0/24 -j REJECT --reject-with icmp-port-unreachable
service iptables save
```

### 12. Cron scripts

Root's crontab on index-old:
```
# ESGF cronjob BEGIN ###
35 0,12 * * * ESG_USAGE_PARSER_CONF=/esg/config/gridftp/esg-bdm-usage-gridftp.conf /esg/tools/esg_usage_parser 
5 0,12 * * * ESG_USAGE_PARSER_CONF=/esg/config/gridftp/esg-server-usage-gridftp.conf /esg/tools/esg_usage_parser 
# ESGF cronjob END ###
```

Root's crontab on data-old:

```
0 */2 * * * /root/dashboard/scripts/check_stats.sh
#Ansible: Run esg_usage_parser
5 0,12 * * * ESG_USAGE_PARSER_CONF=/usr/local/esg-usage-parser/esg_usage_parser.conf /usr/local/esg-usage-parser/src/esg_usage_parser
0 * * * * /root/dashboard/scripts/check_injection.sh 2>&1 | tee -a /root/dashboard/check.log | /usr/bin/logger -t '[CHECK_INJECTION]'
*/30 * * * * top -b -n1 >> /root/dashboard/top.log
```

#### Reinstallation of the monitoring scripts on **data-new**

* Scripts repository installation

```bash
yum -y install subversion.x86_64
cd /root
svn co svn+ssh://sgardoll@forge.ipsl.jussieu.fr/ipsl/forge/projets/prodiguer/svn/esgf_admin_tools/trunk/scripts
mkdir -p /root/stats
```

* Stats injection reminder

This script sends an email when the number of unprocessed entries in the dashboard queue is below 50. The purpose of this reminder is to warm the node admin so as to inject stats.

Just add the following line in the root's crontab:

```
0 */2 * * * /root/scripts/check_stats.sh
```

Then configure the postfix so as to enable sending email (see machine setup procedure).

* Dashboard processing report

This script reports some stats about the activity of the ESGF dashboard, like the
number of dashboard queue entries that are processed or not.

Just add the following line to the root's crontab.

```
0 * * * * /root/scripts/check_injection.sh 2>&1 | tee -a /root/stats/check_injection.log | /usr/bin/logger -t '[CHECK_INJECTION]'
```

* System activity logger

This cron instruction periodically logs the output of the command top so as to
monitor the activity of the processes of the dashboard. Add this instruction to the root's crontab.

```
*/30 * * * * top -b -n1 >> /root/stats/top.log
```

* Stats script (dashboard substitute) 

Install miniconda and specify that ** you don't want to modify the bashrc file** (that interfers with ESGF-Ansible) ! 

```bash
curl -o Miniconda2-latest-Linux-x86_64.sh https://repo.anaconda.com/miniconda/Miniconda2-latest-Linux-x86_64.sh
chmod +x Miniconda2-latest-Linux-x86_64.sh
./Miniconda2-latest-Linux-x86_64.sh
# Wait until the end of the installation.
rm Miniconda2-latest-Linux-x86_64.sh
/root/miniconda2/bin/conda create -n sandbox python=3.6
# Wait until the end of the creation of the environment.
/root/miniconda2/bin/conda install -n sandbox psycopg2 sqlalchemy pyyaml
```

Copy /root/stats/esgf_stats.log from vesgdev-data to /root/stats

Then add the following line in the root's crontab:

```
0,30 * * * *   /root/miniconda2/envs/sandbox/bin/python /root/scripts/esgf_stats.py 2>&1 |/usr/bin/logger -t '[STATS]'
```
### 13. Delete migration archives

!!! warning
    Do not run the following commands until you are very sure that the nodes are alright

On *-new:

```bash
rm -fr /root/migration_backup
```