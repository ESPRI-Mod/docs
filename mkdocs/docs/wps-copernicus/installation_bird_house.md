WPS for Copernicus installation
=========================

* Version: 0.0.1
* Date: 2019-8-28
* Authors: Pierre Logerais
* Keywords: wps copernicus emu installation

## Description

This procedure describes how to install Birdhouse components from scratch on your system, and run a basic Emu WPS for testing.

## Environment

### Requirements

A Linux machine.

### Dependencies

* Python = 3.6 or higher

## Procedure

### Installation of Miniconda

* Custom and run these commands so as to install Miniconda

```bash
PARENT_DIR="${HOME}"
mkdir -p "${PARENT_DIR}/tmp"
cd "${PARENT_DIR}/tmp"

wget 'https://repo.anaconda.com/miniconda/Miniconda2-latest-Linux-x86_64.sh'

chmod +x Miniconda2-latest-Linux-x86_64.sh
./Miniconda2-latest-Linux-x86_64.sh
```

* Then run these commands to create a conda environment

```bash
ENV_NAME='copernicus-wps-test'
source ${HOME}/.bashrc
conda create -y -n "${ENV_NAME}" python=3.6
```

* At last, activate the conda environment 

```bash
conda activate copernicus-wps-test 
```

### Installation of emu

```bash
PARENT_DIR="${HOME}"
cd "${PARENT_DIR}"

git clone https://github.com/bird-house/emu.git
cd emu
make clean install
```

### Configuration

For emu, there is no need to configure anything as it’s made for testing purposes.

### Tests

Run the following command from the emu directory you created earlier :
```bash
make start &
```

You should see a line akin to this :
```bash
starting WPS service on http://localhost:5000/wps
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

In this case, that means that you can run a browser on ```http://localhost:5000/wps```

You can test it with Firefox if you use a GUI on your computer, or via a CLI web browser like this :
```bash
elinks http://localhost:5000/wps
```

If you can see a few lines of XML text, that means the WPS works.

