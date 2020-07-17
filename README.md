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
