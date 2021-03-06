# Operator RBAC setup

If RBAC is in place, users need to setup RBAC rules for etcd operator. This doc serves a tutorial for it.

## Quick setup

If you just want to play with etcd operator, there is a quick setup.

It assumes that your cluster has an admin role. For example, on [Tectonic](https://coreos.com/tectonic/),
there is a `admin` ClusterRole. We are using that here.

Modify or export env `$TEST_NAMESPACE` to a new namespace, then create it:

```bash
$ kubectl create ns $TEST_NAMESPACE
```

Then create cluster role binding:

```bash
$ cat <<EOF | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: example-etcd-operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: default
  namespace: $TEST_NAMESPACE
EOF
```

Now you can start playing. One you are done, clean them up:

```bash
$ kubectl delete clusterrolebinding example-etcd-operator
$ kubectl delete ns $TEST_NAMESPACE
```

## Production setup

For production, we recommend users to limit access to only the resources operator needs, and create a specific role, for the operator.

The example below binds a role to the `default` service account in the namespace that the etcd-operator is running in. To bind to a different serviceaccount modify the `subjects.name` field in the [rolebinding templates](../../example/rbac) as needed.

### Role vs ClusterRole

The permission model required for the etcd-operator depends on the value of its `--create-crd` flag:
- `--create-crd=true` This the default behavior in which the operator will first try to create the CRD if it doesn't exist
  - In this mode the operator requires a ClusterRole with the permission to create a CRD.
- `--create-crd=false` The operator skips creating the CRD before creating the CR
  - In this mode the operator can be run with just a Role without the permission to create a CRD.


## Setup RBAC

Setup the RBAC rules using either a ClusterRole or Role depending on the use case as shown below.

Modify and export the following environment variables. These will be used to fill out the [RBAC templates](../../example/rbac):
```
export ROLE_NAME=<role-name>
export ROLE_BINDING_NAME=<role-binding-name>
export NAMESPACE=<namespace>
```

### RBAC with ClusterRole (create-crd=true)

1. Create a ClusterRole:

    ```sh
    sed -e "s/<ROLE_NAME>/${ROLE_NAME}/g" example/rbac/cluster-role-template.yaml \
      | kubectl create -f -
    ```

2. Create a ClusterRoleBinding which binds the default serviceaccount in the namespace to the ClusterRole:

    ```sh
    sed -e "s/<ROLE_NAME>/${ROLE_NAME}/g" \
      -e "s/<ROLE_BINDING_NAME>/${ROLE_BINDING_NAME}/g" \
      -e "s/<NAMESPACE>/${NAMESPACE}/g" \
      example/rbac/cluster-role-binding-template.yaml \
      | kubectl create -f -
    ```

### RBAC with Role (create-crd=false)

1. Create a Role:

    ```sh
    sed -e "s/<ROLE_NAME>/${ROLE_NAME}/g" \
      -e "s/<NAMESPACE>/${NAMESPACE}/g" \
      example/rbac/role-template.yaml \
      | kubectl create -f -
    ```

2. Create a RoleBinding which binds the default serviceaccount in the namespace to the Role:

    ```sh
    sed -e "s/<ROLE_NAME>/${ROLE_NAME}/g" \
      -e "s/<ROLE_BINDING_NAME>/${ROLE_BINDING_NAME}/g" \
      -e "s/<NAMESPACE>/${NAMESPACE}/g" \
      example/rbac/role-binding-template.yaml \
      | kubectl create -f -
    ```
