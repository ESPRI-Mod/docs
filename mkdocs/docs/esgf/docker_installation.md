
INSTALLATION OF ESGF VIA DOCKER AND ESGF-DOCKER
============================

* Version: 0.0.1
* Date: 27/11/2019
* Authors: Pierre Logerais
* Keywords: centos esgf index data node docker container

## Description

This procedures describes how to install the esgf stack via esgf-docker. This esgf stack differs in that it’s managed via docker containers.

## Pre requisites

A GNU/Linux machine, with docker and docker compose installed.

On Red Hat-based systems, you can install them with this command :

```bash
sudo yum install docker docker-compose
```

## Environment configuration

### Repo cloning

Type the following commands from your node :

```bash
git clone https://github.com/cedadev/esgf-docker.git
cd esgf-docker
```

Then set your environment variables :

```bash
export ESGF_HOSTNAME=any.esgf.org
export ESGF_CONFIG=/path/to/empty/config/directory
export ESGF_DATA=/path/to/data/directory
```

Replace these values with something that corresponds to your environment.

### Pulling images from Docker hub

You only need to do this if they have changed (rare case) or if you are deploying for the first time.

```bash
docker-compose pull
```

### Config generation

Run the following commands to generate the configuration :

```bash
docker-compose run esgf-setup generate-secrets
docker-compose run esgf-setup generate-test-certificates
docker-compose run esgf-setup create-trust-bundle
```

### Launch containers

Use docker-compose :

```bash
docker-compose up -d
```

## Tests

Navigate to the hostname you put in `ESGF_HOSTNAME`. You should see a CoG interface.

You can also view logs with commands like this :

```bash
docker-compose logs [-f] esgf-{cog,index-node,idp-node,orp,slcs,...}
```

Going to the page `https://$ESGF_HOSTNAME/esgf-idp/openid/rootAdmin` should prompt you to a login page. You can find the root password that was generated using the following command :

```bash
echo "$(cat "$ESGF_CONFIG/secrets/rootadmin-password")"
```

### Stopping containers

`docker-compose stop` stops the containers. `docker-compose down -v` removes every container and the associated data volumes.
