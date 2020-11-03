RabbitMQ installation
=======================

* Version: 0.0.1
* Date: 03/11/2020
* Authors: Pierre Logerais
* Keywords: install rabbitmq ssl esgf

## Description

This procedure describes how to install an ESGF RabbitMQ node, with the DKRZ load balancer.

## Hosts & access

### Hosts

Any host on CentOS 7 for the server.

### Access

- ssh access to the root account of host

## Procedure

Log in to the host as root.

### Update the host and install mandatory packages

Execute the following :

```
yum update
yum install epel-release socat wget vim git openssl zlib bzip2
```

### Install Erlang

We need a specific version of Erlang, so execute the following :

```
declare ERL_VER="22.2.4"
wget https://github.com/rabbitmq/erlang-rpm/releases/download/v${ERL_VER}/erlang-${ERL_VER}-1.el7.x86_64.rpm
yum localinstall erlang-${ERL_VER}-1.el7.x86_64.rpm
```

Check that erlang is installed with the command `erl`. Type `x` to test it out, and `Ctrl-c` twice to exit.

If it’s installed, you can remove the RPM :

```rm erlang-${ERL_VER}-1.el7.x86_64.rpm```

### Install RabbitMQ

Add the repo and installed :

```
rpm --import https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
rpm -Uvh https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.2/rabbitmq-server-3.8.2-1.el7.noarch.rpm
yum -q -y update
yum -q -y install rabbitmq-server-3.8.2-1.el7.noarch.rpm```

### Configure RabbitMQ

Enable the plugin we need :

```rabbitmq-plugins enable rabbitmq_management```

Enable the rabbitMQ service :

```
systemctl start rabbitmq-server.service
systemctl enable rabbitmq-server.service
```

Then configure and make RabbitMQ usable :

```
chown -R rabbitmq:rabbitmq /var/lib/rabbitmq
declare NODE_ADMIN_USER=node-admin
declare NODE_ADMIN_PWD=74p9ttcj7muEV4m6BVk69yqxe8PKCenH
rabbitmqctl add_user $NODE_ADMIN_USER $NODE_ADMIN_PWD
rabbitmqctl set_user_tags $NODE_ADMIN_USER administrator
rabbitmqctl set_permissions -p / $NODE_ADMIN_USER ".*" ".*" ".*"
```

Test that the RabbitMQ is running :
```
yum install elinks -y
elinks http://127.0.0.1:15672
```

If that is working, delete the user guest :
```rabbitmqctl delete_user guest```

Edit the file `/etc/rabbitmq/rabbitmq.conf` and add the following content to it :

```
listeners.ssl.default = 5671

ssl_options.cacertfile = /etc/pki/tls/certs/DigiCertCA.crt
ssl_options.certfile   = /etc/pki/tls/certs/esgf-pid-mq.ipsl.upmc.fr.crt
ssl_options.keyfile    = /etc/pki/tls/private/esgf-pid-mq.ipsl.upmc.fr.key
ssl_options.verify     = verify_peer
ssl_options.fail_if_no_peer_cert = false
```

Replace the certificate names with your certificate names. However, your certificates need to be placed in `/etc/pki/tls/certs/DigiCertCA.crt`.

Then add links to the certificates or move them :

```
# If you want to use symlinks
ln -s <path to your CA> /etc/pki/tls/certs/DigiCertCA.crt
ln -s <path to your crt> /etc/pki/tls/certs/esgf-pid-mq.ipsl.upmc.fr.crt
ln -s <path to your key> /etc/pki/tls/private/esgf-pid-mq.ipsl.upmc.fr.key
```

Enable the SSL plugin :

```rabbitmq-plugins enable rabbitmq_auth_mechanism_ssl```

Make sure the openssl service is running, and restart RabbitMQ :

```
systemctl restart openssl
systemctl restart rabbitmq-server
systemctl status rabbitmq-server
```

You also need to configure two additional users : `esgf-downstream` and `esgf-publisher`. Log into your node’s RabbitMQ node (in our example : `http://esgf-pid-mq.ipsl.upmc.fr:15672/#/`) and add these users. Add an `esgf-pid` virtual host as well, and give them access to the virtual host `esgf-pid`. All of this is done through the Admin tab. 

The passwords are as follows :

```
esgf-publisher : LDfj5784T4VeKTxwhpqk8UmSqbC9DkTW
esgf-downstream : qfwSeEjrPTZdfqbFLv7VqgSXzbnXD6Qr```


Then set the permissions :

```
rabbitmqctl set_permissions -p esgf-pid esgf-downstream ".*" ".*" ".*"
rabbitmqctl set_permissions -p esgf-pid esgf-publisher "" ".*" ""
rabbitmqctl set_parameter federation-upstream-set esgffed-upstream-set '[{"upstream":"esgf-pid"},{"upstream":"esgf-pid"}]'
```

You should also make the 5671 port accessible to the load-balancing instance. This can be done by the support team.

You should be able to add the RabbitMQ node to the load balancer, after testing it on your end.


