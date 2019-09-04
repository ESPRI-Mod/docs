MIGRATION FROM CENTOS 6 to 7
============================

* Version: 0.0.1
* Date: 03/09/2019
* Authors: Sébastien Gardoll
* Keywords: migration centos backup esgf

## Description

This procedure describes how to migrate a ESGF node 4.0.4 from CentOS 6 to CentOS 7.

Term definitions:

index-old: index node before migrating (under CentOS 6)
data-old: data node before migrating (under CentOS 6)

index-new: index node after migrating (under CentOS 7)
data-new: data node after migrating (under CentOS 7)

## Procedure

### 1. Create the vm new with temporary host name

### 2. Backup index-old (destination is the index-new)

* Tar balls

```bash
mkdir /root/migration_bakup
cd /root/migration_bakup
tar -pcJvf esgf_config.tar.xz -C /esg config
tar -pcJvf home.tar.xz -C / home
cp -p /root/.bashrc .
cp -p /root/.pgpass .
mkdir certs
cp -rp /etc/certs certs
cp -rp /etc/esgfcerts certs
cp -rp /etc/grid-security certs
cp -rp /var/lib/globus/simple_ca certs
tar -pcJvf certs.tar.xz certs
rm -fr certs
tar -pcJvf cog.tar.xz -C /usr/local cog
tar -pcJvf solr-index.tar.xz -C /esg solr-index
```

* PostgreSQL

```bash
cd /root/migration_bakup
pg_dump -U dbsuper --clean -Z 6 -v -F c esgcet > db_esgcet.bak 2> db_esgcet.log
pg_dump -U dbsuper --clean -Z 6 -v -F c cogdb > db_cogdb.bak 2> db_cogdb.log
pg_dump -U dbsuper --clean -Z 6 -v -F c slcsdb > db_slcsdb.bak 2>db_slcsdb.log
pg_dump -U dbsuper --clean -Z 6 -v -F c postgres > db_postgres.bak 2>db_postgres.log
```

### 3. Backup data-old (destination is the data-new)


* Tar balls

```bash
mkdir -p /root/migration_bakup
cd /root/migration_bakup
tar -pcJvf esgf_config.tar.xz -C /esg config
tar -pcJvf home.tar.xz -C / home
cp -p /root/.bashrc .
cp -p /root/.pgpass .
mkdir certs
cp -rp /etc/certs certs
cp -rp /etc/esgfcerts certs
cp -rp /etc/grid-security certs
tar -pcJvf certs.tar.xz certs
rm -fr certs
tar -pcJvf thredds.tar.xz -C /esg/content thredds
```

* PostgreSQL

```bash
mkdir -p /root/migration_bakup
cd /root/migration_bakup
pg_dump -U dbsuper --clean -Z 6 -v -F c esgcet > db_esgcet.bak 2> db_esgcet.log
pg_dump -U dbsuper --clean -Z 6 -v -F c postgres > db_postgres.bak 2>db_postgres.log
```

### 4. Shutdown the VM old

### 5. Rename the host name of the VM new

### 6. Install from scratch ESGF 4.0.4 on the VM new

### 7. Install the data for the index-new

### 8. Install the data for the data-new

### 9. Try install Solr indexes for index-new

### 10. Configure the iptables for data-new

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

### 11. Cron scripts
