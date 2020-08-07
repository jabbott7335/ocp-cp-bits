## Utilities Tips Tricks Issues 

Set of running notes for Cloud Pak on OCP (operators). If you stumble across this, it could be dated.


### Uninstall Operators

Uninstall the operators and then run this. Dev posted this script in troubleshooting:  bash <(curl -s https://raw.githubusercontent.com/IBM/ibm-common-service-operator/master/common/scripts/force-uninstall.sh)

### Wrong Version of Common Services

For OCP 4.4.8 - 4.4.12 you must install Common Services or risk getting a beta version.  https://github.ibm.com/IBMPrivateCloud/roadmap/issues/39058

### Seeing our Operator Logs

Run this command:  `oc -n openshift-operators logs -f $(oc -n openshift-operators get pod -l app.kubernetes.io/name=odlm -o jsonpath="{.items[].metadata.name}") >> logs.yaml`

### IAM Fix for IAM Operator SA Cluster Role Issue

https://www.ibm.com/support/knowledgecenter/SSHKN6/installer/3.x.x/troubleshoot/op_pending.html

### Podwatch

 `alias podwatch='watch "oc get pods -A|egrep -v \"1/1|2/2|3/3|4/4|5/5|6/6|7/7|Completed\""'`

## View Subscriptions

`oc get subscription.operators.coreos.com  -n openshift-operators`

### Catching My Fork Up

[GitHub doc](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/syncing-a-fork)
```
git fetch upstream
git checkout master
git merge upstream/master
git push
```

## Server Side Apply K-Saucer Workaround

Run `oc edit featuregate cluster`
and configured it with these settings

```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: config.openshift.io/v1
kind: FeatureGate
metadata:
  annotations:
    release.openshift.io/create-only: "true"
  creationTimestamp: "2020-07-21T14:06:06Z"
  generation: 3
  name: cluster
  resourceVersion: "426117"
  selfLink: /apis/config.openshift.io/v1/featuregates/cluster
  uid: e3e9d482-f15d-4145-9858-dd65cb3e6755
spec:
  customNoUpgrade:
    disabled:
    - ServerSideApply
    - LegacyNodeRoleBehavior
    enabled:
    - APIPriorityAndFairness
    - RotateKubeletServerCertificate
    - NodeDisruptionExclusion
    - ServiceNodeExclusion
    - SCTPSupport
  featureSet: CustomNoUpgrade
```
  
The other features `LegacyNodeRoleBehavior, APIPriorityAndFairness, RotateKubeletServerCertificate, NodeDisruptionExclusion, ServiceNodeExclusion, SCTPSupport` in this file are the OOTB feature configurations. Just adding ServerSideApply to this config removed the OOTB features which was not desired.

Once the API server is restarted you. can check the configs with

```oc --namespace=openshift-kube-apiserver rsh kube-apiserver-<xxxxx> cat /etc/kubernetes/static-pod-resources/configmaps/config/config.yaml```

## Azure, OCP, Common Services Cloud Pak for Integration

The worker nodes had camel case label for the region, and thus the PVC for mongodb could not find a node to run on.  Changed the failure-domain.beta.kubernetes.io/region=SouthAfricaNorth to sourthafricanorth.
