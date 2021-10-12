# qiot-manufacturing-factory-installer

steps to install components in Single Node Openshift:

## Prerequisites

Download the Openshift client from the [official download page](https://access.redhat.com/downloads/content/290/ver=4.8/rhel---8/4.8.13/x86_64/product-software)

## Step 1:

- Install helm template sno-pre-install.

This template will contain the operators of Openshift-GitOps, AMQ-Broker and the MachineConfig to mount HostPath for persistence of MongoDB and PostgreSQL.

```
helm install sno-pre-install ./sno-pre-install --create-namespace --namespace factory
```

Wait ~4 minutes until the SNO reboot caused by the MachineConfig.

## Step 2:

- Install helm template sno-install.

This template will contain the components required to deploy MongoDB, PostgreSQL and AMQ Broker.

```
helm install sno-install ./sno-install --namespace factory
```
