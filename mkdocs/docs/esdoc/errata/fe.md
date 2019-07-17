# ES-DOC Errata Command line interface

* Version: 0.0.1
* Date: 17/07/2019
* Authors: Atef Bennasser
* Keywords: Errata, Datasets, CMIP6, ES-DOC, Command line, Client

## Description

The ESGF errata FE is the graphic user web interface for the ES-DOC errata 
service. 
It includes a listing of all issues discovered and created on the platform,
and allows for filtering via a set of key facets. 

The FE also presents users with a PID lookup feature. Which when 
queried with either a string (dataset id or dataset/file pid string) 
or an uploaded txt file, deliver a list of results regarding the errata
and the state of the data to date. 

A new addition for the ES-DOC errata FE comes in the shape of online forms, 
that facilitate users interaction with the service.  

* Software version: v0.6.4.0 
* License: CeCILL v2.1


## Hosts & access

PROD: web602.webfaction.com
TEST: web531.webfaction.com


## Environment

OS: x86_64 GNU/Linux

### Requirements

WebFaction credentials

### Dependencies

Errata ws


## Deployment

Log in as root
git pull 

## Configuration
N/A 
### Web-Services
N/A
## Monitoring
N/A
## Maintenance
N/A
## Certificates 
N/A
### Web Server Certificate

Domain:     https:*.es-doc.org
Issued by:    TERENA SSL CA 3
Expires:    29/10/2020

## Monitoring

AWStats collected automatically 

## Third party services

WebFaction: Internet Service Provider

### Billing

WebFaction
2 machine monthly plan: $ 10 per machine per month
Expires 
