WPS for Copernicus installation
=========================

* Version: 0.0.1
* Date: 2019-8-28
* Authors: Pierre Logerais
* Keywords: wps copernicus usage birdy

## Description

This procedure describes how to use the WPS that is installed at IPSL as part of the Copernicus project.

## Environment

### Requirements

A linux machine, with admin rights to it.

### Dependencies

* git
* anaconda

## Procedure

### Birdy installation

* Run these commands to download and install birdy :

```bash
conda install -c conda-forge birdy
```

* Alternative method : via github

```bash
git clone https://github.com/bird-house/birdy.git
cd birdy
conda env create -f environment.yml
python setup.py install
```

### Usage

Here are a few examples of how to use birdy that you can reproduce on your own computer.

* Through python

```py
from birdy import WPSClient
emu = WPSClient(url='http://copernicus-wps.ipsl.upmc.fr/wps') # to access the WPS at IPSL specifically
emu = WPSClient(url='http://compute.mips.copernicus-climate.eu/wps') # to access the load balanced WPS
emu.hello(name='Birdy').get()[0]
```

If you don’t see any error and the hello is returning a value, that means your WPS is running.

* Through your shell

First, activate birdy and define the WPS you want to reach :

```bash
conda activate birdy
export WPS_SERVICE=http://copernicus-wps.ipsl.upmc.fr/wps
export WPS_SERVICE=http://compute.mips.copernicus-climate.eu/wps # to access the load balanced service
```

Then you can make sure that it works :

```bash
birdy -h
```

This command should put out the following text :
```bash
Usage: birdy [OPTIONS] COMMAND [ARGS]...

  Birdy is a command line client for Web Processing Services.

  Documentation is available on readthedocs:
  http://birdy.readthedocs.org/en/latest/

Options:
  --version         Show the version and exit.
  --cert TEXT       Client side certificate containing both certificate and
                    private key.
  -S, --send        Send client side certificate to WPS. Default: false
  -s, --sync        Execute process in sync mode. Default: async mode.
  -t, --token TEXT  Token to access the WPS service.
  -h, --help        Show this message and exit.

Commands:
  cmip5_regridder   CMIP5 Regridder: CMIP5 Regridder working on the...
  cordex_subsetter  CORDEX Subsetter: CORDEX Subsetter working on the...
```

As you can see, birdy sees the two methods `cordex_subsetter` and `cmip5_regridder`. This shows that the WPS is reachable and it has the correct functionnalities.

You can download a dataset with a command like this :

```bash
birdy cmip5_regridder --model IPSL-CM5A-LR --experiment historical --variable tas
```

