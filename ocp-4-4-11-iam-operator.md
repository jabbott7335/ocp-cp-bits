## The OCP 4.4.11 Problem

Are you getting pods in "configerror" status in the ibm-common-services project after installing the IBM Common Services operator on 4.4.11?  You also likely are missing pods for IAM `iam-`.  The IBM IAM Operator is likely stuck in a pending state.  To fix this we will fix the cluster role associated to the `ibm-iam-operand-restricted` service account.

Fixing this problem.  First we will need a bit of information from the clusterrolebinding:

```oc edit clusterrolebinding```

Search for the clusterrole that is associated with the serviceaccount ibm-iam-operand-restricted.  You are looking for a section similar to the following:

```
- apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    creationTimestamp: "2020-07-14T22:50:06Z"
    labels:
      olm.owner: ibm-iam-operator.v3.6.4
      olm.owner.kind: ClusterServiceVersion
      olm.owner.namespace: ibm-common-services
    name: ibm-iam-operator.v3.6.4-6b55d84fd9-55d6bc8b5d
    resourceVersion: "34561"
    selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/ibm-iam-operator.v3.6.4-6b55d84fd9-55d6bc8b5d
    uid: 487e7254-40f4-4898-b131-e22bab211e5e
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: ibm-iam-operator.v3.6.4-6b55d84fd9
  subjects:
  - kind: ServiceAccount
    name: ibm-iam-operand-restricted
```

I know I am focused on the correction section of code because the `subject` of this `ClusterRoleBinding` of `kind: ServiceAccount` and the `name: ibm-iam-operand-restricted`.  The important bit of information is the `name: ibm-iam-operator.v3.6.4-6b55d84fd9`.  You will need the name of your clusterrole for the next step.

Next, edit the `ibm-iam-operator.v3.6.4-6b55d84fd9` clusterrole:

```oc edit clusterrole ibm-iam-operator.v3.6.4-6b55d84fd9```

**Add** the below `yaml` snippet to the end of the clusterrole definition and save the results to apply them.

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
