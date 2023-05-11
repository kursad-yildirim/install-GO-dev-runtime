# Install OpenShift Cluster

## Overview

A single node OpenShift cluster will be installed. Openshift version 4.12  with latest fixpack will be used. We will follow the official documentation which can be found [here](https://docs.openshift.com/container-platform/4.12/installing/installing_sno/install-sno-installing-sno.html) with some adjustments. The main steps of the installation process will be as below:

- Download RHCOS iso file
- Create install-config.yaml and ignition file
- Update RHCOS iso
    - Embed ignition data
    - Put network config file
- Boot the cluster node with the RHCOS iso
- Manually modify the cluster to set the hostname
- Monitor the installation process
- Wait until all cluster operators are ready
- Add a new cluster administrator

## 1. Preparation

Download openshift installer

```bash
# OCP_VERSION="latest-4.12"

# curl -k https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$OCP_VERSION/openshift-install-linux.tar.gz  > openshift-install-linux.tar.gz

# tar zxvf openshift-install-linux.tar.gz

# chmod +x openshift-install
```

Download RHCOS iso file

```bash
# ISO_URL=$(./openshift-install coreos print-stream-json | grep location | grep x86_64 | grep iso | cut -d\" -f4)

# curl -L $ISO_URL > rhcos-live.x86_64.iso
```

Prepare `install-config.yaml`, ignition, and network configuration files:

```bash
# CLUSTER_NAME="tripko"

# BASE_DOMAIN="tuff.local"

# DISK_NAME="/dev/nvme0n1"

# CLUSTER_NETWORK="172.24.0.0/14"

# CLUSTER_NETWORK_HOST_PREFIX="23"

# SERVICE_NETWORK="172.30.0.0/16"

# PULL_SECRET=*<removed>*

# SSH_KEY=$(cat ~/.ssh/id_rsa.pub)

# tee -a install-config.yaml <<EOF
apiVersion: v1
baseDomain: ${BASE_DOMAIN}
compute:
- name: worker
  hyperthreading: Enabled
  replicas: 0
controlPlane:
  name: master
  hyperthreading: Enabled
  replicas: 1
metadata:
  name: ${CLUSTER_NAME}
networking:
  networkType: OVNKubernetes
  clusterNetwork:
  - cidr: ${CLUSTER_NETWORK}
    hostPrefix: ${CLUSTER_NETWORK_HOST_PREFIX}
  serviceNetwork:
  - ${SERVICE_NETWORK}
platform:
  none: {}
bootstrapInPlace:
  installationDisk: ${DISK_NAME}
pullSecret: '${PULL_SECRET}'
sshKey: '${SSH_KEY}'
EOF

# mkdir ocp

# cp install-config.yaml ocp

# ./openshift-install --dir=ocp create single-node-ignition-config
INFO Consuming Install Config from target directory 
WARNING Making control-plane schedulable by setting MastersSchedulable to true for Scheduler cluster settings 
INFO Single-Node-Ignition-Config created in: ocp and ocp/auth

# mkdir -p fake-root-sno/etc/NetworkManager/system-connections
# tee -a fake-root-sno/etc/NetworkManager/system-connections/ens160.nmconnection <<EOF 
[connection]
id=ens160
type=ethernet
interface-name=ens160

[ethernet]
mtu=1500

[ipv4]
method=manual
addresses=10.10.221.5/24
gateway=10.10.221.254
dns=10.10.221.254
dns-search=tuff.local

[ipv6]
method=disabled
EOF

# tee -a fake-root-sno/etc/hostname <<EOF 
trip-ocp.tuff.local
EOF
```

After bootstrapping completed hostname file will need to be recreated.

Now we will update the default RHCOS iso, so that cluster and environment settings will be applied automatically during the first boot.

```bash
# /home/workspace/ocp-sno/filetranspile -f fake-root-sno/ -i ocp/bootstrap-in-place-for-live-iso.ign > iso.ign

# install_script_old=$(cat iso.ign | jq '.storage.files[] | select(.path=="/usr/local/bin/install-to-disk.sh") | .contents.source' | cut -f 2 -d "," | tr -d \")

# install_script_new=$(cat iso.ign | jq '.storage.files[] | select(.path=="/usr/local/bin/install-to-disk.sh") | .contents.source' | cut -f 2 -d "," | tr -d \" | base64 -d | sed 's/coreos-installer install -i/coreos-installer install -n -i/g' | base64 -w0)

# sed -i 's/'${install_script_old}'/'${install_script_new}'/g' iso.ign
```

At this stage the iso file which should be able to install a single node Openshift cluster with the specified settings is ready.

## 2. Installation

## 3. Post install tasks