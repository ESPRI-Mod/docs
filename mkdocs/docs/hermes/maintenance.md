MAINTENANCE OF HERMES MACHINES
=======================================

* Version : 0.0.1
* Contributors : Pierre Logerais
* Date : 31/01/2020
* Keywords : hermes maintenance restarts

## Description

This procedure describes how to maintain and restart hermes machines if need be.

## Procedure

The hermes machines are separated into 2 groups : test and prod.

In these 2 groups, there are 3 machines, one of which hosts the db, another the web server, and another the Message Queue services.

These are the machines that correspond :

* hermes.ipsl.upmc.fr : web-prod
* hermes-mq-prod.private.ipsl.fr : mq-prod
* hermes-db-prod.private.ipsl.fr : db-prod
* hermes-test.ipsl.upmc.fr : web-test
* hermes-mq-test.private.ipsl.fr : mq-test
* hermes-db-test.private.ipsl.fr : db-test

### Rebooting

Before rebooting, you should stop the hermes services. The command is as follows :

```bash
hermes-mq-daemons-stop-phase-1; sleep 10; hermes-mq-daemons-stop-phase-2
reboot
```

Always run it on the MQ server **BEFORE** the others.
