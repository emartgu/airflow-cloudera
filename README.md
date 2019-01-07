# Apache Airflow parcel and CSD for Cloudera

This git repository is used to build both CSDs and parcels for CDH.

Airflow will run as service in Cloudera Manager, and its configuration is maintained through the CM web UI.
All necessary python packeges (airflow, celery, flower, flask) are installed inside the parcel which uses Anaconda3 distribution.

Environmemnt:
  - Ubuntu 16.04.5 LTS/CDH 5.14.0
  - Anaconda3 v5.1.0
  - Airflow v1.10.1


Still WIP. 


## Prerequisites

```bash

sudo apt install maven
sudo apt-get install libmysqlclient-dev
sudo apt-get install libkrb5-dev
sudo apt-get install libsasl2-dev
sudo apt-get install openjdk-8-jdk-headless
sudo apt-get install python-kerberos
sudo apt-get install python-mysqldb

```

## Build

To build the CSDs and Parcels yourself, you can run the build script:

```
#Installation and upgrading requires setting SLUGIFY_USES_TEXT_UNIDECODE=yes
export SLUGIFY_USES_TEXT_UNIDECODE=yes
#Build the Parcel files, this make take some time
. build.sh parcel

#Build the CSDs
. build.sh csd
```

## Installation

### Installing the CSD and parcel files

Copy the Airflow parcel and checksum file to the Cloudera Manager Local Parcel Repository Path.
By default, the path is `/opt/cloudera/parcel-repo.`
To verify the path to use, click `Administration > Settings`. In the navigation panel, select the Parcels category. Place the Airflow parcel file in the path configured for Local Parcel Repository Path.

```bash

# Copy CSD
sudo cp AIRFLOW-1.10.1.jar /opt/cloudera/csd
sudo chown cloudera-scm:cloudera-scm /opt/cloudera/csd/AIRFLOW-1.10.1.jar

# Copy parcels
sha1sum AIRFLOW-1.10.1_build/AIRFLOW-1.10.1-xenial.parcel | cut -d ' ' -f 1 > AIRFLOW-1.10.1_build/AIRFLOW-1.10.1-xenial.parcel.sha
sudo cp -rf AIRFLOW-1.10.1_build/AIRFLOW-1.10.1-xenial.parcel /opt/cloudera/parcel-repo/AIRFLOW-1.10.1-xenial.parcel
sudo cp -rf AIRFLOW-1.10.1_build/AIRFLOW-1.10.1-xenial.parcel.sha /opt/cloudera/parcel-repo/AIRFLOW-1.10.1-xenial.parcel.sha
sudo chown cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo/AIRFLOW-1.10.1-xenial.parcel
sudo chown cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo/AIRFLOW-1.10.1-xenial.parcel.sha
sudo service cloudera-scm-server restart



```

### Distribute and Activate the Airflow Parcel

After you add the Airflow repository to Cloudera Manager, you can download, distribute, and activate the Airflow parcel across the cluster.

The Airflow parcel repository is added to Cloudera Manager during the installation of the CSD. 

To view the list of available parcels, in the menu bar, click the Parcels icon.
The Airflow parcel displays in the list of available parcels. If it doesn't display, click Check for New Parcels.

To distribute the Airflow parcel to the cluster, click Distribute.
After distribution, the Distribute button becomes the Activate button.
To activate the Airflow parcel, click Activate.


More Information about installing custom services can be found at [here](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_mc_addon_services.html#concept_kpt_spj_bn__section_upv_nqj_bn).


## Troubleshoot

### 1. Clear cloudera parcel cache

I wanted to update the Airflow parcel, so I completely removed the parcel from the cluster via the parcel page in Cloudera Manager. But with an updated content,  when I distributed the parcel I noticed that the content of the parcel still contains the content from the old version instead of the updated version. The solution is to delete all files that have been downloaded by the "flood" torrent mechanism

Let's say I have 16 hosts in my cluster

```bash
for i in {1..16};do ssh -o StrictHostKeyChecking=no ubuntu@10.10.30.$i sudo rm -rf /opt/cloudera/parcels/.flood/; done
for i in {1..16};do ssh -o StrictHostKeyChecking=no ubuntu@10.10.30.$i sudo rm -rf /opt/cloudera/parcel-cache; done
for i in {1..16};do ssh -o StrictHostKeyChecking=no ubuntu@10.10.30.$i sudo service cloudera-scm-agent restart; done
```

The goal here is to clear out files that appear to be causing trouble for the download.  When we start the agent, it should detect it needs to start the download again.
