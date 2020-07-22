## The OCP 4.4.x IAM Operator Cluster Role Issue

https://www.ibm.com/support/knowledgecenter/SSHKN6/installer/3.x.x/troubleshoot/op_pending.html eventually points to a troubleshooting section that doesn't exactly work.

Are you getting pods in "configerror" status in the ibm-common-services project after installing the IBM Common Services operator on 4.4.x?  You also likely are missing pods for IAM `oc get po -n ibm-common-services | grep iam-`.  The IBM IAM Operator is likely stuck in a pending state and its pod may be missing.  To fix this we will fix the ClusterRoles associated to the `ibm-iam-operand-restricted` and `ibm-iam-operator` service accounts.

To fix this behavior we will alter a pair of ClusterRoles that are bound to a pair of ServiceAccounts.

** Before running this note the -o wide on the clusterrolebinding get will not work unless you have a recent oc CLI.  If you find this you will want to upgrade your local oc CLI or use the UI to lookup the Cluster Roles under User Management **

First, find the cluster role associated with the `ibm-iam-operand-restricted` ClusterRoleBinding subject:

```oc get clusterrolebinding -o wide -n ibm-common-services  | grep ibm-iam-operand-restricted | awk '{print $3}'```

Now use use `oc edit` with the returned string, such as:

```oc edit ClusterRole/ibm-iam-operator.v3.6.4-xyz123```

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
Next, find the cluster role associated with the `ibm-iam-operator` ClusterRoleBinding subject:

```oc get clusterrolebinding -o wide -n ibm-common-services  | grep ibm-common-services/ibm-iam-operator | awk '{print $3}'```

Now use use `oc edit` with the returned string, such as:

```oc edit ClusterRole/ibm-iam-operator.v3.6.4-6b55d84fd9```

Go to the very bottom and **append** the below `yaml` snippet **to the end** and save the results to apply them.  If you do this same step in the UI the `yaml` format differs:

```
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - services/finalizers
  - endpoints
  - persistentvolumeclaims
  - events
  - configmaps
  - secrets
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - apps
  resources:
  - deployments
  - daemonsets
  - replicasets
  - statefulsets
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
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
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - apps
  resources:
  - replicasets
  - deployments
  verbs:
  - get
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

> To understand further what you have done, you can explore the ClusterRoleBindings with the **Role Bindings** tool from the UI.  The changes above could be made from the **Roles** tool, but the above `uyaml` would require reformat.
