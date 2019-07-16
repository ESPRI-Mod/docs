# ES-DOC Errata Command line interface

* Version: 1.1.2
* Date: 16/07/2019
* Authors: Atef Bennasser
* Keywords: Errata, Datasets, CMIP6, ES-DOC, Command line, Client

## Description

The ESGF errata client, called `esgissue` is a piece of software that
enables the interaction with the errata service. Like the web forms, it
can be used to easily create, update, close and retrieve issues. The
client is basically aimed to be used by publishing teams, data providers
and managers that are notified of the existence of an issue regarding
one or many datasets/files.

This lightweight CLI relies on `.json` and `.txt` file records. This
enables the data provider, in charge of ESGF issues, to efficiently
manage one or several issues remotely and locally.

* Software version: 1.1.2
* License: CeCILL v2.1

> **note**
>
> [Some screencasts are available on
> YouTube](https://www.youtube.com/channel/UCFVy0HC9cnGbIKc6UsDIHDA/playlists)
> to learn about the Errata CLI usage.


## Hosts & access

This is a command line client for the errata service. 
Meant to be installed in personal spaces. 
Therefore no hosts are required. 

## Environment

Tested on Linux/MacOS

### Requirements

Users can read errata information without credentials. 
In order to have write access on errata service, the client needs
GH credentials. Users should also make sure they are declared as 
errata officers for their respective institution. Contact errata
admins for more information.


### Dependencies

A remote Errata server. In case you suspect the service is down
feel free to contact the admins.
 
Other *soft* dependencies are available in the requirements.txt 
file for the module and should be automatically installed via 
pip when installing the client.

## Deployment

Or rather installation is through pip. 
run : > pip install esgissue-client


## Configuration

Only configuration required is user credentials in case write 
access is needed. Check user manual for step by step guide. 

### Web-Services
N/A
## Monitoring
N/A
## Maintenance
N/A
## Certificates 
N/A
### Web Server Certficate 
N/A
