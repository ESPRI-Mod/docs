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

### Restarting Hermes

* Stop the MQ daemons

Before rebooting, you should stop the hermes services. The command is as follows :

```bash
hermes-mq-daemons-stop-phase-1; sleep 30; hermes-mq-daemons-stop-phase-2
reboot
```

Always run it on the MQ server **BEFORE** the others.

* Stop the web daemon

Connect to hermes (web-prod) and run :

```bash
hermes-web-daemon-stop
```

* Clean up the hermes postgres database

Connect to hermes-db-prod. Run the following commands :

```bash
# Truncate tables
hermes-db-pgres-truncate-mq-messages
hermes-db-pgres-truncate-mq-email
hermes-db-pgres-truncate-mq-email-stats 

# Run a vacuum on the database
hermes-db-pgres-vacuum-full

# Backup dB. This backup can be used on the test db, if necessary
hermes-db-pgres-backup
```

The database backup can be found at `/opt/hermes/ops/backups/db`

* Restart the web daemon

Connect to hermes (web-prod) and run :
```bash
hermes-web-daemon-start
```

* Clean up the mail box

Log on Zimbra (https://mail.ipsl.upmc.fr) with the account `superviseur`.

You then can vacuum the inboxes AMPQ-PROD-REJECTED and AMPQ-PROD-PROCESSED.

Then, move the emails in the inbox AMPQ-PROD to AMPQ-TEMP.

* Restart the MQ daemon

Connect to hermes-mq-prod, then run :

```bash
hermes-mq-daemons-start
```

Then, move back the emails in the inbox AMPQ-TEMP from AMPQ-PROD

Then, move the emails in the inbox AMPQ-PROD to AMPQ-TEMP.


Connect to hermes-mq-prod, then run :

```bash
hermes-mq-daemons-start
```

Then, move back the emails in the inbox AMPQ-TEMP from AMPQ-PROD.
