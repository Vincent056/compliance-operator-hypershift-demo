# Compliance-Operator-Hypershift-Demo
This repository contains the files used to demonstrate the compliance-operator on HyperShift.

## Prerequisites
* a running HyperShift management cluster
* a hosted HyperShift cluster running on the HyperShift management cluster
## Overview
Demonstrate how to use compliance-operator on HyperShift managedment cluster and hosted cluster.

## Steps
### Running CIS scan on the HyperShift management cluster
1. clone the compliance-operator repository and install the operator:
```
git clone https://github.com/Vincent056/compliance-operator.git
cd compliance-operator
git checkout hypershift-demo
make deploy-local
```

2. find the name of the HostedCluster you want to scan on the management cluster:
```
[vincent@fedora hypershift]$ oc get --namespace clusters hostedclusters
NAME                 VERSION   KUBECONFIG                            PROGRESS   AVAILABLE   PROGRESSING   MESSAGE
wenshen-hypershift             wenshen-hypershift-admin-kubeconfig   Partial    True        False         The hosted control plane is available
```

3. clone this repository and use the name of the HyperShfit Cluster wenshen-hypershift, and apply the tailored profile and ScanSettingBinding file:
```
git clone https://github.com/Vincent056/compliance-operator-hypershift-demo.git
cd compliance-operator-hypershift-demo
oc apply -f management-cluster-file/tailored-profile-cis.yaml
oc apply -f management-cluster-file/tailored-profile-cis-ssb.yaml
```

4. check the scan result:
```
[vincent@fedora hypershift-demo]$ oc get ccr | head -10
NAME                                                                                STATUS   SEVERITY
cis-compliance-hypershift-accounts-restrict-service-account-tokens                  MANUAL   medium
cis-compliance-hypershift-accounts-unique-service-account                           MANUAL   medium
cis-compliance-hypershift-api-server-admission-control-plugin-alwaysadmit           PASS     medium
cis-compliance-hypershift-api-server-admission-control-plugin-alwayspullimages      PASS     high
cis-compliance-hypershift-api-server-admission-control-plugin-namespacelifecycle    PASS     medium
cis-compliance-hypershift-api-server-admission-control-plugin-noderestriction       PASS     medium
cis-compliance-hypershift-api-server-admission-control-plugin-scc                   PASS     medium
cis-compliance-hypershift-api-server-admission-control-plugin-securitycontextdeny   PASS     medium
cis-compliance-hypershift-api-server-admission-control-plugin-service-account       PASS     medium

[vincent@fedora hypershift-demo]$ oc get ccr | grep FAIL
cis-compliance-hypershift-audit-log-forwarding-webhook                              FAIL     medium
cis-compliance-hypershift-idp-is-configured                                         FAIL     medium
cis-compliance-hypershift-ocp-api-server-audit-log-maxbackup                        FAIL     low
```

### Running CIS scan on the HyperShift hosted cluster
1. clone this repository and apply the tailored profile and ScanSettingBinding file
```
git clone https://github.com/Vincent056/compliance-operator.git
cd compliance-operator
git checkout hypershift-demo
PLATFORM=hypershift make deploy-local
```

2. clone this repository:
```
git clone https://github.com/Vincent056/compliance-operator-hypershift-demo.git
cd compliance-operator-hypershift-demo
```
3. make sure the scansetting only has worker role:
```
[vincent@fedora hypershift-demo]$ oc get scansetting -o yaml
apiVersion: v1
items:
- apiVersion: compliance.openshift.io/v1alpha1
  kind: ScanSetting
  maxRetryOnTimeout: 3
...
  roles:
  - worker
```
4. apply the tailored profile and ScanSettingBinding file:
```
oc apply -f hosted-cluster-file/tailored-profile-cis.yaml
oc apply -f hosted-cluster-file/tailored-profile-cis-ssb.yaml
```
5. check the scan result:
```
[vincent@fedora hypershift-demo]$ oc get ccr | grep PASS
cis-compliance-hypershift-audit-profile-set                                   PASS     medium
cis-compliance-hypershift-configure-network-policies                          PASS     high
cis-compliance-hypershift-rbac-debug-role-protects-pprof                      PASS     medium
cis-compliance-hypershift-scc-limit-container-allowed-capabilities            PASS     medium
ocp4-cis-node-worker-file-groupowner-cni-conf                                 PASS     medium
ocp4-cis-node-worker-file-groupowner-kubelet-conf                             PASS     medium
ocp4-cis-node-worker-file-groupowner-multus-conf                              PASS     medium
ocp4-cis-node-worker-file-groupowner-ovn-cni-server-sock                      PASS     medium
ocp4-cis-node-worker-file-groupowner-ovn-db-files                             PASS     medium

[vincent@fedora hypershift-demo]$ oc get ccr | grep FAIL
cis-compliance-hypershift-configure-network-policies-namespaces               FAIL     high
cis-compliance-hypershift-idp-is-configured                                   FAIL     medium
cis-compliance-hypershift-kubeadmin-removed                                   FAIL     medium
```

## Files in this repository

files for HyperShift management cluster:
* `/hosted-cluster-file/tailored-profile-cis.yaml`
    - the tailored profile for CIS benchmark
* `/hosted-cluster-file/tailored-profile-cis-ssb.yaml`
    - the ScanSettingBingding file for CIS benchmark tailored profile

files for HyperShift hosted cluster:
* `/management-cluster-file/tailored-profile-cis.yaml`
    - the tailored profile for CIS benchmark
* `/management-cluster-file/tailored-profile-cis-ssb.yaml`
    - the ScanSettingBingding file for CIS benchmark tailored profile

## Reference
* [HyperShfit Concepts and Personas](https://hypershift-docs.netlify.app/reference/concepts-and-personas/)
* [HyperShift Installation Guide](https://hypershift-docs.netlify.app/getting-started/)
* [Complinace Operator Usage Guide for HyperShift](https://github.com/Vincent056/compliance-operator/blob/master/doc/usage.md)
* [Compliance Operator Installation Guide](https://github.com/Vincent056/compliance-operator/blob/master/doc/install.md)