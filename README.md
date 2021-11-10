# qiot-manufacturing-factory-installer

steps to install components in Single Node Openshift:

## Prerequisites

Download the Openshift client from the [official download page](https://access.redhat.com/downloads/content/290/ver=4.8/rhel---8/4.8.13/x86_64/product-software)

Download helm CLI: https://helm.sh/docs/intro/install/


- Login to your SNO with oc CLI.

```
oc login -u kubeadmin -p [KUBEADMIN-PASSWORD] --server=https://api.[SNO.DOMAIN].com:6443
```

# Cleanup

```
helm uninstall sno-srv-install --namespace factory
helm uninstall sno-install --namespace factory
helm uninstall sno-core-install --namespace factory
helm uninstall sno-olm-install --namespace factory
oc delete subscription amq-broker -n factory
oc delete subscription amq-broker -n cert-manager
oc delete subscription amq-broker -n openshift-gitops-operator
helm uninstall sno-volumes-install --namespace factory
oc delete project factory
```

## Step 1:
- Install helm template sno-pre-install. It contains the Helm chart for the infrastructure piece.

This template will install the MachineConfig to mount HostPath for persistence.

```
helm install sno-volumes-install ./sno-volumes-install --create-namespace --namespace factory
```

Wait ~4 minutes until the SNO reboot caused by the MachineConfig.

## Step 2:

- Login to your SNO with oc CLI.

```
oc login -u kubeadmin -p [KUBEADMIN-PASSWORD] --server=https://api.[SNO.DOMAIN].com:6443
```

- Install helm template sno-olm-install. It contains the heml chart for the infrastructure piece.

This template will contain the operators of Openshift-GitOps and AMQ-Broker

```
helm install sno-olm-install ./sno-olm-install --namespace factory
```

## Step 3:

- Install helm template sno-core-install. It contains the heml chart for the factory core piece.

This template will contain the components required to deploy core-service and registration-service.

```
helm install --set coreservice.serial=<<YOUR_FACTORY_SERIAL>>,coreservice.name=<<YOUR_FACTORY_NAME>> sno-core-install ./sno-core-install --namespace factory
```

e.g.:

```
helm install --set coreservice.serial=abattaglfactorytestserial03,coreservice.name=abattaglfactorytestname03 sno-core-install ./sno-core-install --namespace factory
```

Run the next command to check if the issuer is valid

```
oc get issuer -n factory
```

## Step 4:

- Install helm template sno-install. It contains the helm chart for the software infrastructure piece.

This template will contain the components required to deploy MongoDB, PostgreSQL and AMQ Broker.

```
export WILDCARD=apps.$(oc get dns cluster -o jsonpath='{.spec.baseDomain}')
helm install sno-install ./sno-install --set amqbroker.certificate.wildcardDomain=$WILDCARD --namespace factory
```

Run the next command to check if the pods are running

```
oc get pods -n factory
```

If you can't see the broker pod "{{ .Values.amqbroker.brokerName }}" (see your values.yaml file!) run the next command as workaround to restart the AMQ-Broker operator pod:

```
oc delete pod -l name=amq-broker-operator -n factory
```

Check again if the broker is created:
```
oc get pods -n factory
```

## Step 5:

- Install helm template sno-after-install. It contains the heml chart for the workload piece.

This template will contain the components required to deploy MongoDB, PostgreSQL and AMQ Broker.

```
helm install sno-srv-install ./sno-srv-install --namespace factory
```


## Considerations

We need to provide persistent storage for databases, and for this context we add the easiest solution to integrate all-in-one providing hostPath storage which requires running pods as privileged, and SElinux chcon (done with MachineConfig CR), please note this is just for testing, in production environment you will need to add external storage to the NUC, like external disks, nfs, ODF, etc.