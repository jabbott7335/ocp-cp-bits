## The OCP 4.4.11 Problem

https://www.ibm.com/support/knowledgecenter/SSHKN6/installer/3.x.x/troubleshoot/op_pending.html eventually points to a troubleshooting section that doesn't exactly work.

Are you getting pods in "configerror" status in the ibm-common-services project after installing the IBM Common Services operator on 4.4.11?  You also likely are missing pods for IAM `iam-`.  The IBM IAM Operator is likely stuck in a pending state.  To fix this we will fix the cluster role associated to the `ibm-iam-operand-restricted` service account.

Fixing this problem.  First we will need a bit of information from the clusterrolebinding:

- Log in to your OpenShift Container Platform cluster console.
- Click **User Management > Role Bindings**
- Select the **Cluster-wide Role Bindings** filter and enter the string `ibm-iam-operand-restricted` to filter by the subject name.
- For the role binding, copy the value for the Role Ref column. This value can resemble **ibm-iam-operator.v3.6.4-abc1d**.

Next, edit the `ibm-iam-operator.v3.6.4-abc1d` clusterrole you copied the actual name of in the previous step:

```oc edit clusterrole ibm-iam-operator.v3.6.4-abc1d```

Go to the very bottom and **append** the below `yaml` snippet **to the end** and save the results to apply them.  If you do this same step in the UI the `yaml` format differs:

```
- apiGroups:
  - monitoring.coreos.com
  resources:
  - servicemonitors
  verbs:
  - get
  - create
- apiGroups:
  - apps
  resourceNames:
  - ibm-iam-operator
  resources:
  - deployments/finalizers
  verbs:
  - update
- apiGroups:
  - operator.ibm.com
  resources:
  - '*'
  - policydecisions
  - oidcclientwatchers
  - authentications
  - policycontrollers
  - paps
  - securityonboardings
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - certmanager.k8s.io
  resources:
  - '*'
  - certificates
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - networking.k8s.io
  resources:
  - '*'
  - ingresses
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - batch
  resources:
  - jobs
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
```

The changes will take effect automatically in a few minutes and the operator deployment will continue.
